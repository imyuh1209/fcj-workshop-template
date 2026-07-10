---
title: "Phase 3: Backend Deployment (ECS Fargate)"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3 </b> "
---

# Enterprise Backend Deployment (ECS Fargate)

In this phase, we will set up the core infrastructure for the Spring Boot Backend application, placing it securely inside Private Subnets.

---

### Step 1: Initialize Docker Image Repository (Amazon ECR)

1. Access the **Elastic Container Registry (ECR)** service.
2. Click **Create repository**.
3. Visibility settings: Choose **Private**.
4. Repository name: **`aura-academic-be`**.
5. Click **Create repository**.

![Create ECR](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step1.png)

---

### Step 2: Set up Application Load Balancer (ALB)

The ALB will act as a "Receptionist", standing in the Public Subnet to receive requests and route them to the Private Subnet.

1. Access the **EC2** service, from the left menu select **Load Balancers** -> **Create load balancer**.
2. Select **Application Load Balancer**.
3. Name: `aura-academic-alb`.
4. Scheme: Ensure **Internet-facing** is checked (Allows the ALB to communicate over the internet).
5. IP address type: Ensure **IPv4** is checked.
6. Network mapping:
   - VPC: Select `aura-academic`.
   - Mappings: Select **BOTH PUBLIC SUBNETS** (`Public-1a` and `Public-1b`). They must be in different availability zones.
7. Security groups: Remove the default, select `aura-academic-alb-sg`.
8. Listeners and routing:
   - Click the small blue text **`create target group`** (Located under Forward to target group). A new tab will open.
   - In the new tab, configure as follows:
     - Target type: Must be **IP addresses** (Since Fargate uses IPs, not Instances).
     - Target group name: `aura-academic-ecs-tg`
     - Port: `8080` (The port the Spring Boot Backend is running on).
     - VPC: Select `aura-academic`.
     - Health checks: Change the path to **`/api/health`** (This is the standard health check endpoint of our current Backend).
     - Scroll down, click **Next** -> Click **Create target group**.
   - After creating, **return to the previous tab** (Load Balancer creation tab).
   - Click the **Refresh arrow** next to the search box.
   - Click the search box and select the **`aura-academic-ecs-tg`** you just created.
9. Click **Create load balancer**.

![Create ALB](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step2.png)

---

### Step 3: Create Compute Cluster (ECS Cluster)

1. Access the **Elastic Container Service (ECS)** -> **Clusters** -> **Create cluster**.
2. Cluster name: `aura-academic-cluster`.
3. Infrastructure: Select **AWS Fargate** (Serverless).
4. Click **Create**.

![Create Cluster](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step3.png)

---

### Step 4: Create Task Definition

1. In the ECS console, from the left menu select **Task definitions** -> **Create new task definition**.
2. Name: `aura-academic-task`.
3. Launch type: AWS Fargate.
4. OS, Architecture, Network: Linux/X86_64, AWSVPC.
5. CPU: `1 vCPU`, Memory: `2 GB`.
6. Container details:
   - Name: `backend-container`.
   - Image URI: Paste the URI of the ECR created in Step 1.
   - Container port: `8080`.
7. Click **Create**.

![Task Definition](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step4.png)

---

### Step 5: Deploy Service (ECS Service) into Private Subnet

This is the most critical step to achieve Enterprise standards.

1. Click the blue name **`aura-academic-cluster`** (As seen in the Clusters list screen).
2. Once the Cluster details interface opens, look down to the **Services** tab. Click the **Create** button in that tab.
3. Compute options: Launch type (Fargate).
4. Task definition: Select `aura-academic-task`.
5. Service name: **`aura-academic-backend-enterprise`** (Different from old name to run in parallel - Zero Downtime).
6. Desired tasks: `1`.
7. **Networking (IMPORTANT):**
   - VPC: `aura-academic`.
   - Subnets: Remove the default public subnets. **ONLY SELECT 1 PRIVATE SUBNET (`Private-1a`)** (You must choose the Subnet in the same Availability Zone as the NAT Gateway to have internet access).
   - Security group: Select `aura-academic-ecs-sg`.
   - **Public IP: Turn off (DISABLED)**. The system will route to the internet via the NAT Gateway.
8. Load balancing:
   - Type: Application Load Balancer.
   - Container: `backend-container: 8080`.
   - Application Load Balancer: Select **Use an existing load balancer** -> Select `aura-academic-alb`.
   - Listener: Select **Use an existing listener** -> Select `80 HTTP`.
   - Target group: Select **Use an existing target group** -> Select `aura-academic-ecs-tg`.
9. Click **Create**.

![Create Service](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step5.png)

Your Backend is now maximally protected and only interacts with the outside world through the ALB!
