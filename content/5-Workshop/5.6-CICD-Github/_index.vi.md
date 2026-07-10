---
title: "Giai đoạn 6: Triển khai CI/CD (GitHub Actions)"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6 </b> "
---

# Triển khai Tự động hóa CI/CD với GitHub Actions

CI/CD là "trái tim" của DevOps. Với GitHub Actions, mỗi khi Developer đẩy code mới lên nhánh `main`, hệ thống sẽ tự động làm hết mọi thứ: Build Frontend đẩy lên S3, Build Backend thành Container đẩy lên ECR và báo cho ECS tự động cập nhật bản mới. Không cần ai đụng tay vào!

---

### Bước 1: Khai báo Biến bí mật (GitHub Secrets)

Để GitHub có quyền truy cập vào AWS cũng như bảo mật các thông tin nhạy cảm, bạn cần vào tab **Settings -> Secrets and variables -> Actions** của từng Repository và tạo các biến (**New repository secret**) tương ứng:

**1. Đối với Frontend (Next.js):**

- `AWS_ACCESS_KEY_ID`: Mã Access Key của IAM User.
- `AWS_SECRET_ACCESS_KEY`: Mã Secret Key của IAM User.

**2. Đối với Backend (ECS Fargate):**

- `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY`: Để build và đẩy Docker Image.
- Các biến môi trường của Spring Boot: `MONGODB_URI`, `JWT_SECRET`, `SES_SMTP_HOST`, `SES_SMTP_USER`, `SES_SMTP_PASS`, `GOOGLE_CLIENT_ID`, `CORS_ORIGINS`.
- Liên kết với AI: `AI_EC2_PRIVATE_IP` (IP Private của máy EC2) và `LITELLM_MASTER_KEY`.

**3. Đối với AI Engine (EC2):**

- `AWS_ACCESS_KEY_ID`: Mã Access Key (Để dùng quyền SSM gọi vào AWS).
- `AWS_SECRET_ACCESS_KEY`: Mã Secret Key.
- `AI_EC2_INSTANCE_ID`: Lấy cái ID của máy ảo EC2 AI (VD: `i-0101a388d12922f46`).
- Các Key AI: `GEMINI_API_KEY`, `GROQ_API_KEY`.
- `BACKEND_API_URL`: URL để AI gọi ngược lại lấy dữ liệu từ Backend.

---

### Bước 2: Khai báo File Deploy (`deploy.yml`)

Thay vì thao tác thủ công, bạn chỉ cần tạo thư mục `.github/workflows` trong source code của mỗi repo, và thêm file `deploy.yml`.

#### 1. Kịch bản của Frontend (S3 + CloudFront):

File này thực hiện việc build Next.js ra tĩnh, đẩy lên S3 và xóa cache CDN.

<details>
<summary><b>👉 Click để xem Code deploy.yml (Frontend)</b></summary>

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

#### 2. Kịch bản của Backend (ECS Fargate):

File này tự động đăng nhập ECR, Build Docker Image, và khởi động lại Task trên ECS.

<details>
<summary><b>👉 Click để xem Code deploy.yml (Backend)</b></summary>

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

#### 3. Kịch bản của AI Engine (EC2 GPU):

Dùng SSH để chui vào EC2, pull code mới về và gọi lệnh PM2 khởi động lại server.

<details>
<summary><b>👉 Click để xem Code deploy.yml (AI Engine)</b></summary>

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
