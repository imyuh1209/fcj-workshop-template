---
title: "Resource Cleanup"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7 </b> "
---

# Resource Cleanup

After completing the project and successfully defending it, you **MUST** delete the resources on AWS to avoid your credit card being charged in the following months.

Below is the safe resource deletion order (delete from the outside in):

### 1. Delete Application Load Balancer (ALB) and Target Groups
1. Access EC2 -> **Load Balancers**.
2. Select `aura-academic-alb`, click **Actions** -> **Delete**.
3. Switch to **Target Groups**, select both `aura-academic-tg-ecs` and `aura-academic-tg-ec2` groups, click **Actions** -> **Delete**.

### 2. Delete ECS Cluster and ECR
1. Access **ECS**, go to the `aura-academic-cluster`.
2. In the Services tab, select `aura-academic-be-service`, click **Update**, change the **Desired tasks** to `0` and save (To turn off running containers).
3. After tasks report 0, select the service and click **Delete**.
4. At the top right corner of the Cluster screen, click **Delete cluster**.
5. Access **ECR**, tick the `aura-academic-be` repository and click **Delete** (type 'delete' to confirm).

### 3. Delete AI Server (EC2)
1. Access **EC2** -> **Instances**.
2. Select the `aura-academic-ai-server` machine.
3. Click **Instance state** -> **Terminate instance**. Wait a moment for the machine to be completely destroyed.

### 4. Delete Frontend (S3 & CloudFront)
1. Access **CloudFront**, select the Distribution `aura-academic-fe-cdn`, click **Disable**. Wait about 2-3 minutes for the disable process to complete.
2. Then select this Distribution again and click **Delete**.
3. Access **S3**, go to the bucket `aura-academic-fe-2024`.
4. Click the **Empty** button to delete all files inside (you will need to type `permanently delete`).
5. After it's empty, return to the Bucket list, select it, and click **Delete**.

### 5. Delete NAT Gateway & Virtual Network (VPC)

🔥 **Red Warning:** NAT Gateway and Elastic IP (EIP) if forgotten will deduct a quite "painful" amount from your wallet (over $1/day). You **must manually delete** them before deleting the VPC, because the VPC deletion command will be blocked if the NAT is still alive.

**A. Delete NAT Gateway & Release IP:**
1. On the left menu of the VPC service, scroll down to the Virtual private cloud section, select **NAT gateways**.
2. Select the NAT Gateway of `aura-academic`, click **Actions** -> **Delete NAT gateway** (type `delete` to confirm).
3. **Make some tea, sit and wait about 2-3 minutes** until the NAT state completely changes to *Deleted*.
4. Immediately after, select **Elastic IPs** in the left menu. Select the IP address that was just released, click **Actions** -> **Release Elastic IP addresses**. *(Absolutely do not forget this step, floating IPs on AWS will incur junk charges)*.

**B. Delete VPC (Final Step):**
1. Access **VPC** -> **Your VPCs** again.
2. Select `aura-academic`, click **Actions** -> **Delete VPC**. This powerful command will automatically clean up all Subnets, Route Tables, Internet Gateways, and remaining Security Groups.

---

🎉 **Congratulations!** You have cleaned up the entire Cloud environment flawlessly. AuraAcademic's Enterprise architecture practice train ends perfectly here! Sleep well tonight without worrying about Visa deduction text messages!
