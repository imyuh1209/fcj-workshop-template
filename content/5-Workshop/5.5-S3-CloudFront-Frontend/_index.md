---
title: "Phase 5: Frontend Hosting with S3 & CloudFront"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5 </b> "
---

# Deploy Frontend (Next.js) with S3 & CloudFront

The Frontend component of the application (Next.js) will be built into static files (HTML, CSS, JS) and hosted with unlimited storage on Amazon S3. It will then be distributed globally at ultra-fast speeds using a Content Delivery Network (CDN) - Amazon CloudFront.

---

### Step 1: Create S3 Storage Bucket

1. Access the **S3** service, click **Create bucket**.
2. Enter a Bucket name (Example: `aura-academic-frontend-2024` - this name must be globally unique).
3. Under **Object Ownership**, leave the default `ACLs disabled`.
4. Under **Block Public Access**, keep the **Block all public access** checkbox ticked. The system will lock this bucket down, only allowing CloudFront to access the files (Enterprise Security).
5. Scroll to the bottom and click **Create bucket**.

![Create S3](/images/5-Workshop/5.5-S3-CloudFront-Frontend/5.5-s3-step1.png)

---

### Step 2: Configure ALB & Push Static Code to S3

Since the old backend system has been removed, you need to inform the Frontend of the new "doorway" to the system (the Load Balancer).

1. Open the `.env` (or `.env.local`, `.env.production`) file in your Next.js source code.
2. Find the `NEXT_PUBLIC_API_URL` variable (or the variable containing the API domain) and change its value to the **DNS name of your ALB** (Example: `http://aura-academic-alb-12345.ap-southeast-1.elb.amazonaws.com`).
3. Open Terminal and Build the project:

```bash
npm install
npm run build
```

The above commands will automatically compile the entire UI into static files (HTML, CSS, JS) and place them in the `out/` directory. 4. Open the AWS S3 Console, and upload **all the contents inside the `out/` directory** to the S3 Bucket you just created.

---

### Step 3: Create CDN Distribution (CloudFront)

_(New AWS CloudFront Interface 2024)_

1. Access the **CloudFront** service, click **Create distribution**.
2. **Step 1 - Choose a plan**: Select the **Free ($0/month)** plan. Click _Next_.
3. **Step 2 - Get started**: Under **Distribution name**, enter `aura-academic-fe-cdn`. Click _Next_.
4. **Step 3 - Specify origin**:
   - **Origin type**: Select **Amazon S3**.
   - **S3 origin**: Click the **Browse S3** button and select your bucket. _(Note: Ignore the yellow "Use website endpoint" warning if it appears; we must use the bucket endpoint to maintain private security for S3)_.
   - Scroll down to **Origin access**: Very important. Select **Origin access control settings (recommended)** -> Click **Create control setting** -> **Create**.
   - Finally, click _Next_.
5. **Step 4 - Enable security**: Leave the default WAF security configuration. Click _Next_.
6. **Step 5 - Review and create**:
   - Thanks to a new AWS feature, CloudFront will automatically update the Policy for the S3 Bucket, so we don't need to do it manually anymore.
   - You just need to click **Create distribution** at the bottom.

![Create CloudFront](/images/5-Workshop/5.5-S3-CloudFront-Frontend/5.5-s3-step3.png)

---

### Step 4: Additional Configuration (Mandatory)

Because the AWS quick-create interface hides some settings, we need to add them manually after creation:

**1. Declare the default home page (Default Root Object):**

- On the details page of the newly created CloudFront distribution, in the **General** tab, click the **Edit** button under Settings.
- Scroll down to **Default root object**, type `index.html`. Click **Save changes**.

**2. Force browsers to use HTTPS (Required for Camera access):**

- Switch to the **Behaviors** tab.
- Tick the only existing behavior (Default (\*)), then click **Edit**.
- Scroll down to **Viewer protocol policy**, change it from _HTTP and HTTPS_ to **Redirect HTTP to HTTPS**. Click **Save changes**.

---

### Step 5: Verify the Final Website

Return to the CloudFront interface, copy the URL under **Distribution domain name** (Example: `d1234abcd.cloudfront.net`).
Paste it into your browser, and you will see your Web application load smoothly with a secure HTTPS padlock, ready for the Camera stream.
