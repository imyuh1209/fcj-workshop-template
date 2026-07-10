---
title: "Phase 6: CI/CD Deployment (GitHub Actions)"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6 </b> "
---

# CI/CD Automation with GitHub Actions

CI/CD is the "heart" of DevOps. With GitHub Actions, whenever a Developer pushes new code to the `main` branch, the system automatically does everything: Builds the Frontend and pushes to S3, Builds the Backend into a Container, pushes to ECR, and tells ECS to auto-update to the new version. No human intervention needed!

---

### Step 1: Declare Secrets (GitHub Secrets)

For GitHub to access AWS and to secure sensitive information, you need to go to the **Settings -> Secrets and variables -> Actions** tab of each Repository and create the corresponding secrets (**New repository secret**):

**1. For Frontend (Next.js):**

- `AWS_ACCESS_KEY_ID`: The IAM User's Access Key.
- `AWS_SECRET_ACCESS_KEY`: The IAM User's Secret Key.

**2. For Backend (ECS Fargate):**

- `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`: To build and push Docker Images.
- Spring Boot environment variables: `MONGODB_URI`, `JWT_SECRET`, `SES_SMTP_HOST`, `SES_SMTP_USER`, `SES_SMTP_PASS`, `GOOGLE_CLIENT_ID`, `CORS_ORIGINS`.
- Link to AI: `AI_EC2_PRIVATE_IP` (Private IP of the EC2 machine) and `LITELLM_MASTER_KEY`.

**3. For AI Engine (EC2):**

- `AI_EC2_HOST`: Public IP of the EC2 machine (for SSH).
- `EC2_USER`: Default is `ubuntu`.
- `EC2_SSH_KEY`: Copy the entire content of the `.pem` file downloaded when creating EC2 and paste it here.
- AI Keys: `GEMINI_API_KEY`, `GROQ_API_KEY`.
- `BACKEND_API_URL`: URL for AI to call back to the Backend for data.

---

### Step 2: Declare Deploy File (`deploy.yml`)

Instead of manual operations, you just need to create a `.github/workflows` folder in the source code of each repo, and add a `deploy.yml` file.

#### 1. Frontend Script (S3 + CloudFront):

This file builds Next.js statically, pushes to S3, and invalidates the CDN cache.

<details>
<summary><b>👉 Click to view deploy.yml Code (Frontend)</b></summary>

```yaml
name: Deploy Frontend (S3 + CloudFront)

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    name: Build & Deploy to AWS S3
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Set up Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Create .env.local
        run: |
          cat > .env.local <<EOF
          NEXT_PUBLIC_API_URL=${{ secrets.NEXT_PUBLIC_API_URL }}
          NEXT_PUBLIC_AI_URL=${{ secrets.NEXT_PUBLIC_AI_URL }}
          NEXT_PUBLIC_GOOGLE_CLIENT_ID=${{ secrets.NEXT_PUBLIC_GOOGLE_CLIENT_ID }}
          EOF

      - name: Build Next.js (static export)
        run: |
          sed -i 's/\/\/ output: '"'"'export'"'"'/output: '"'"'export'"'"'/' next.config.mjs
          sed -i 's/\/\/ trailingSlash: true/trailingSlash: true/' next.config.mjs
          npm run build
          mkdir -p out
          echo '<meta http-equiv="refresh" content="0; url=/vi" />' > out/index.html

          find out -name "index.txt" | while read -r file; do
            dir=$(dirname "$file")
            if [ "$dir" != "out" ]; then
              cp "$file" "$dir.txt"
            fi
          done

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-southeast-1

      - name: Deploy to S3
        run: |
          aws s3 sync out/ s3://${{ secrets.S3_BUCKET_NAME }} \
            --delete \
            --cache-control "public, max-age=31536000, immutable" \
            --exclude "*.html"

          aws s3 sync out/ s3://${{ secrets.S3_BUCKET_NAME }} \
            --delete \
            --cache-control "public, max-age=0, must-revalidate" \
            --include "*.html"

      - name: Invalidate CloudFront cache
        continue-on-error: true
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"

      - name: ✅ Deploy complete
        run: echo "Frontend deployed to https://${{ secrets.CLOUDFRONT_DOMAIN }}"
```

</details>

#### 2. Backend Script (ECS Fargate):

This file automatically logs into ECR, Builds the Docker Image, and restarts the Task on ECS.

<details>
<summary><b>👉 Click to view deploy.yml Code (Backend)</b></summary>

```yaml
name: Deploy Backend → ECR → ECS Fargate

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: aura-academic-be
  ECS_CLUSTER: aura-academic-cluster
  ECS_SERVICE: aura-academic-be-service
  CONTAINER_NAME: backend-container

jobs:
  build-and-deploy:
    name: Build Docker → Push ECR → Deploy ECS
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push image to ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker tag   $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
          docker push  $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push  $ECR_REGISTRY/$ECR_REPOSITORY:latest
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      - name: Download current task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition aura-academic-be-task \
            --query taskDefinition > task-definition.json

      - name: Update task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}
          environment-variables: |
            SPRING_DATA_MONGODB_URI=${{ secrets.MONGODB_URI }}
            APP_JWT_SECRET=${{ secrets.JWT_SECRET }}
            SPRING_MAIL_HOST=${{ secrets.SES_SMTP_HOST }}
            SPRING_MAIL_USERNAME=${{ secrets.SES_SMTP_USER }}
            SPRING_MAIL_PASSWORD=${{ secrets.SES_SMTP_PASS }}
            SPRING_MAIL_PROPERTIES_MAIL_SMTP_AUTH=true
            SPRING_MAIL_PROPERTIES_MAIL_SMTP_STARTTLS_ENABLE=true
            APP_GOOGLE_CLIENT_ID=${{ secrets.GOOGLE_CLIENT_ID }}
            APP_CORS_ALLOWED_ORIGINS=${{ secrets.CORS_ORIGINS }}
            LITELLM_PROXY_URL=http://${{ secrets.AI_EC2_PRIVATE_IP }}:4000
            LITELLM_PROXY_KEY=${{ secrets.LITELLM_MASTER_KEY }}
            LITELLM_MODEL_MAIN=main-ai
            LITELLM_MODEL_VISION=vision-ai

      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

</details>

#### 3. AI Engine Script (EC2 GPU):

Uses SSH to enter the EC2, pull new code, and call PM2 commands to restart the server.

<details>
<summary><b>👉 Click to view deploy.yml Code (AI Engine)</b></summary>

```yaml
name: Deploy AI Service → EC2 GPU

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    name: Deploy to EC2 GPU
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy to EC2 GPU via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.AI_EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          port: 22
          command_timeout: 5m
          script: |
            set -e
            if [ ! -d "/opt/aura-ai" ]; then
              sudo mkdir -p /opt/aura-ai
              sudo chown -R $USER:$USER /opt/aura-ai
              git clone https://github.com/lengthanhdat/AuraAcademic_AI.git /opt/aura-ai
              cd /opt/aura-ai
              sudo apt-get update && sudo apt-get install -y python3-venv
              python3 -m venv venv
              if ! command -v pm2 &> /dev/null; then
                curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
                sudo apt-get install -y nodejs
                sudo npm install -g pm2
              fi
            fi

            cd /opt/aura-ai
            git fetch origin main
            git reset --hard origin/main

            source venv/bin/activate
            pip install --no-cache-dir -r requirements.txt -q

            cat <<ENV_EOF > .env
            GEMINI_API_KEY_1=${{ secrets.GEMINI_API_KEY }}
            GEMINI_API_KEY_2=${{ secrets.GEMINI_API_KEY }}
            GEMINI_API_KEY_3=${{ secrets.GEMINI_API_KEY }}
            GEMINI_API_KEY_4=${{ secrets.GEMINI_API_KEY }}
            GEMINI_API_KEY_5=${{ secrets.GEMINI_API_KEY }}
            GROQ_API_KEY_1=${{ secrets.GROQ_API_KEY }}
            GROQ_API_KEY_2=${{ secrets.GROQ_API_KEY }}
            GROQ_API_KEY_3=${{ secrets.GROQ_API_KEY }}
            GROQ_API_KEY_4=${{ secrets.GROQ_API_KEY }}
            GROQ_API_KEY_5=${{ secrets.GROQ_API_KEY }}
ENV_EOF

            pm2 delete aura-ai || true
            pm2 start bash --name "aura-ai" -- -c 'cd /opt/aura-ai && source venv/bin/activate && export BACKEND_API_URL="${{ secrets.BACKEND_API_URL }}" && python main.py'

            pm2 delete aura-litellm || true
            pm2 start bash --name "aura-litellm" -- -c 'cd /opt/aura-ai && source venv/bin/activate && set -a && source .env && set +a && litellm --config litellm_config.yaml --port 4000'
            pm2 save
```

</details>

---
