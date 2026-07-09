---
title: "Stage 2: VPC Network"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2 </b> "
---

# Initializing Virtual Private Cloud (VPC)

Amazon Virtual Private Cloud (VPC) is the heart of our AWS infrastructure network.

**Note (Cost-Optimization):** Although the Enterprise design diagram requires splitting Public/Private Subnets and using a NAT Gateway, to maximize the practice budget savings (about $86/month), we will deploy the **Optimized Architecture (Using only Public Subnets)**. Security will be strictly tightened through Security Groups.

---

### Step 1: Create VPC with "VPC and more"

1. Access the **VPC** service on the AWS Console interface.
2. Click the orange **Create VPC** button in the upper right corner.
3. Select the **VPC and more** option (this option helps auto-create related components).
4. Configure the parameters as follows:
   - **Name tag auto-generation:** `aura-academic`
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - **Number of Availability Zones (AZs):** Select `2` (Mandatory to have 2 AZs to install a Load Balancer).
   - **Number of public subnets:** Select `2`
   - **Number of private subnets:** Select `0` (Cost optimization for Students)
   - **NAT gateways:** Select **None** (Very Important ⚠️ - Helps save $86/month).
   - **VPC endpoints:** Select **None**.
5. Click the **Create VPC** button and wait for AWS to automatically create the Subnets, Internet Gateway, and Route Tables.

---

### Step 2: Enable "Auto-assign public IP" for Public Subnets

For virtual machines (EC2) or Containers (ECS) located in the Public Subnet to automatically receive a Public IP address and access the Internet, we need to enable this feature:

1. On the left menu of the VPC service, select **Subnets**.
2. Find and check the first subnet containing the word `public` in its name: **`aura-academic-subnet-public1-ap-southeast-1a`**.
3. Click the **Actions** button (at top right) -> Select **Edit subnet settings**.
4. Check the box **Enable auto-assign public IPv4 address** -> **Save**.
5. Repeat exact steps 2-4 for the second public subnet: **`aura-academic-subnet-public2-ap-southeast-1b`**.
*(Note: Never enable this feature for the 2 subnets with `private` in their names).*

---

### Step 3: Set up Security Groups (Firewall)

Since all resources are located in the Public Subnet, the Security Group (SG) acts as a "vital checkpoint" to protect the system from Hackers:

1. On the left menu, scroll down to the Security section, select **Security groups** -> **Create security group**.
2. **ALB Security Group (For Load Balancer):**
   - Name: `aura-academic-alb-sg`
   - VPC: Select the newly created `aura-academic` VPC.
   - Inbound rules: Allow `HTTP` (Port 80) and `HTTPS` (Port 443) from `Anywhere-IPv4`.
3. **ECS Backend Security Group:**
   - Name: `aura-academic-ecs-sg`
   - VPC: Select `aura-academic` VPC.
   - Inbound rules: 
     - Custom TCP, Port `8080`, Source: Click the magnifying glass icon and type `alb`, then select **`aura-academic-alb-sg`** (Only allow traffic from ALB into Backend, strictly block direct internet access).
4. **EC2 GPU Security Group (For AI):**
   - Name: `aura-academic-ai-sg`
   - VPC: Select `aura-academic` VPC.
   - Inbound rules: 
     - Custom TCP, Port `8001`, Source: Click the magnifying glass icon and type `alb`, then select **`aura-academic-alb-sg`** (For WebSockets from ALB).
     - Custom TCP, Port `4000`, Source: Click the magnifying glass icon and type `ecs`, then select **`aura-academic-ecs-sg`** (For internal API from Backend).
     - (There is NO NEED to open Port 22 for SSH, because we will use AWS Systems Manager (Session Manager) to securely connect to the Private Subnet).
---

At this point, the super cost-effective and secure network backbone of the system is ready! We can move on to the [Next Stage](../5.3-ecs-backend) to configure the Backend.
