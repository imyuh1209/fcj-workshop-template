---
title: "Phase 4: EC2 CPU Configuration for AI"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4 </b> "
---

# EC2 CPU Configuration for AI (YOLOv8 & WebSockets)

To process the WebSockets video stream from the candidate's camera, we will use Amazon EC2 to run the AI. In particular, we will hide this EC2 instance within the Private Subnet for absolute safety according to Enterprise standards.

---

### Step 1: Launch Instance

1. Access the **EC2** service, click **Launch instance**.
2. Name: `aura-academic-ai-server`.
3. Application and OS Images: Select **Ubuntu Server 24.04 LTS**.
4. Instance type: To optimize cost while ensuring performance, select **t3.medium** (2 vCPU, 4GB RAM) or higher. Don't worry about cost as we will use a Spot Instance.
5. Key pair: Click *Create new key pair* (if you don't have one), name it **`aura-academic-key`**, and download the `.pem` file to your computer as a backup (in practice we prefer connecting via Session Manager).
6. Configure storage: The default is 8 GiB, you should increase it to **15 GiB** or **20 GiB** (gp3 type) because downloading Python libraries and AI Models consumes significant space.

![Launch Instance 1](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step1-1.png)

![Launch Instance 2](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step1-2.png)

---

### Step 2: Network Configuration & Hiding in Private Subnet

1. Scroll down to **Network settings**, click **Edit**.
2. VPC: Select `aura-academic`.
3. Subnet: **MANDATORY** select a **Private Subnet** (Example: `Private-1a`).
4. Auto-assign public IP: Must select **Disable**.
5. Firewall (security groups): Select *Select existing security group*, tick `aura-academic-ai-sg`.

![Network Settings](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step2.png)

---

### Step 3: Login to server without Public IP

Since the server is in a Private Subnet and has no Public IP, we cannot use the default `EC2 Instance Connect` feature (as AWS yellow warning indicates).

**Mandatory requirement (Attach IAM Role):** To use the higher security feature SSM, your EC2 needs permissions.
- Go outside to the Instances list, right-click the virtual machine -> **Security** -> **Modify IAM role**.
- If you use an AWS Academy account: Select the `LabRole` -> Click **Update IAM role**.
- **If the IAM Role list is empty (like a personal account):**
  1. Click the blue text **Create new IAM role** nearby.
  2. A new page opens, click **Create role**. Select **AWS service** -> Choose Use case as **EC2** -> Next.
  3. In the permissions search box, type `AmazonSSMManagedInstanceCore`, tick the square box next to that name -> Next.
  4. Name the Role name as `aura-academic-ssm-role` -> Scroll to the bottom and click **Create role**.
  5. Return to the previous EC2 tab, click the small **Refresh** circular button next to the dropdown, select the newly created `aura-academic-ssm-role` -> Click **Update IAM role**.

After attaching permissions, proceed to connect:
1. Tick the `aura-academic-ai-server` machine, click **Connect**.
2. On the **In web browser** tab, look down at *Choose how to connect*, select the **SSM Session Manager** box (Maximum security).
3. Click the orange **Connect** button. The browser will open a black Terminal window. You have successfully entered the Private Subnet!

![Connect EC2](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step3.png)

---

### Step 4: Install AI & FastAPI

1. In the browser Terminal window, run the system update command. The computer will automatically go online to download update packages (via NAT Gateway):
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install python3-pip git libgl1 libglib2.0-0 -y
   ```
2. Install the necessary AI and WebSockets libraries (Since Ubuntu 24.04 high security blocks direct installation, we need to add the `--break-system-packages` flag to force install):
   ```bash
   pip3 install fastapi uvicorn websockets ultralytics opencv-python-headless --break-system-packages
   ```
3. Download the AI source code from Github to the server (remember to switch to the `home` directory before cloning to avoid permission denied errors):
   ```bash
   cd ~
   git clone https://github.com/lengthanhdat/AuraAcademic_AI.git
   ```
4. Move into the newly downloaded code directory and launch the background AI Engine (to run 24/7):
   ```bash
   cd AuraAcademic_AI
   nohup python3 -m uvicorn main:app --host 0.0.0.0 --port 8001 &
   ```

### Step 5: Configure Target Group and Load Balancer for AI

For the external system (Internet) to communicate with the background AI Engine, you need to "bridge" from the Application Load Balancer (ALB) into the EC2:

1. Go to AWS Console **EC2 -> Target Groups**, create a new Target Group named `aura-academic-ai-tg`.
2. Select Target type as **Instances**, Protocol as **HTTP**, Port as **8001**, and select VPC `aura-academic`.
3. At the Register targets step, tick the `aura-academic-ai-server` virtual machine and click **Include as pending below**. Then create Target Group.
4. Switch to the **Load Balancers** menu, tick your current ALB. Look down at the lower half of the screen, click the **Listeners and rules** tab (right next to the Details tab).
5. You will see a Listener line with Port **80** (HTTP protocol). Click the link in the **Rules** column (or click directly on that Listener line) -> Select **Add rule**.
6. Configure the rule to "steer" the AI stream as follows:
   - **Name**: Name it `ai-websocket-rule`.
   - **Condition**: Select type **Path**, type `/ws/*` (to catch all AI image analysis streams from the Receptionist).
   - **Action**: Select **Forward** to the `aura-academic-ai-tg` Target Group you just created above.
   - Click Create/Save to save. (At this point, you can also conveniently delete the old EC2 port 8000 rule if you want).

![Configure ALB for AI](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step5.png)
