---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AuraAcademic

## AI-Powered Proctoring & Exam Platform on AWS

### 1. Executive Summary

AuraAcademic is an online examination platform integrated with AI proctoring designed to ensure fairness and transparency for remote exams. The system utilizes the YOLOv8 model for real-time cheating detection via camera streams. By combining an AWS cloud-native architecture (CloudFront, ECS Fargate, EC2 GPU Spot) with managed services, AuraAcademic provides an automated, low-latency monitoring solution while optimizing operational costs (reducing infrastructure costs by up to 70% using Spot Instances) for educational institutions.

### 2. Problem Statement

**What’s the Problem?**
Current online examination platforms lack effective automated proctoring mechanisms, making cheating easy. Existing AI proctoring solutions in the market (e.g., ProctorU, Respondus) often come with expensive licensing fees, complex integration processes, and demand massive bandwidth or server resources.

**The Solution**
AuraAcademic solves this by building a comprehensive system:

- **Frontend (Next.js):** Statically hosted on Amazon S3 and distributed globally via CloudFront for a smooth experience.
- **Backend Core API (Spring Boot):** Runs on ECS Fargate Spot to handle exam logic, submissions, and authentication.
- **AI Proctoring (FastAPI + YOLOv8):** Hosted on an EC2 GPU instance (G4dn) to process real-time WebSockets streams from students, detecting anomalies like looking away, multiple people, or leaving the frame.
- **Video Evidence:** Directly uploaded to S3 via VPC Endpoints for secure storage of violation evidence.
- **Security & Authentication:** Managed by Amazon Cognito, AWS WAF, IAM, and Secrets Manager.

**Benefits and Return on Investment (ROI)**
The system automates the proctoring process, reducing the need for manual invigilators. By leveraging Spot Instances for both ECS Fargate and EC2 GPU, the system cuts infrastructure costs by up to 70% compared to On-Demand pricing. The total cost to maintain the system at a testing/small scale is only around $26 - $30 USD/month. This solution enables schools and educational organizations to adopt advanced proctoring technology affordably, enhancing the credibility of their online exams.

### 3. Solution Architecture

AuraAcademic employs a microservices architecture on AWS, strictly separating the Core API from AI Processing.

**AWS Services Used**

- **Amazon S3 & CloudFront**: Hosts and distributes the Next.js frontend globally, and stores video evidence.
- **Amazon Cognito**: Manages user authentication via JWT.
- **ALB (Application Load Balancer)**: Routes REST API requests to ECS and WebSockets traffic to the EC2 GPU instance.
- **Amazon ECS (Fargate Spot)**: Runs the Spring Boot Backend container.
- **Amazon EC2 (G4dn Spot)**: Runs FastAPI and the YOLOv8 model for real-time video processing.
- **AWS SQS & Lambda**: Handles asynchronous (Event-Driven) exam extraction to reduce load on the Backend.
- **AWS WAF & Secrets Manager**: Web application firewall and secure secret storage.
- **Amazon SES**: Sends email notifications and OTPs.
- **VPC, NAT Instance & Endpoints**: Secures the internal network and optimizes S3 uploads and CloudWatch logs with minimal data transfer costs.
- **External Services**: MongoDB Atlas (Database) and Google Gemini API (Automated Question Generation).

**Component Design**

- **Web Interface**: Students take exams and stream their cameras; Teachers manage exams and review violation reports.
- **Core API**: Handles exam logic, pushes messages to SQS for AI question extraction, and stores results.
- **AI Engine**: Receives camera streams via WebSockets, detects cheating using YOLOv8, and automatically uploads violation videos to S3.

### 4. Technical Implementation

**Implementation Phases (3-Month Internship Project)**

- **Month 1 (Initiation & Design):** Analyze VPC architecture, design the MongoDB database schema, prepare the YOLOv8 model, and define basic Event-Driven flows.
- **Month 2 (Backend & AI Development):** Program the Spring Boot API, configure SQS + Lambda for exam extraction, and build FastAPI for WebSockets processing.
- **Month 3 (Completion & Deployment):** Build the Next.js interface, deploy the system to AWS (S3, CloudFront, ECS, EC2 Spot), conduct testing, and finalize the internship report.

**Technical Requirements**

- **Frontend**: Next.js, WebSockets client, browser-based recording.
- **Backend**: Spring Boot, Spring Security, MongoDB Atlas integration.
- **AI**: FastAPI, OpenCV, YOLOv8 object detection, video streaming processing.
- **DevOps/AWS**: Docker, VPC (Public/Private Subnets, NAT Instance, VPC Endpoints), SQS, Lambda.

### 5. Timeline & Milestones

- **Weeks 1-2**: Finalize system design, UI/UX, and set up code repositories.
- **Weeks 3-5**: Develop core features (Exam Management) and SQS + Lambda integration for Gemini extraction.
- **Weeks 6-9**: Integrate real-time YOLOv8 AI for camera stream analysis via WebSockets.
- **Weeks 10-12**: Deploy to AWS, test Spot Instance fault-tolerance, and complete the final internship report.

### 6. Budget Estimation

Utilizing a cost-optimized model with Spot Instances, estimated for a Testing/Small Scale environment (~100 concurrent students):

- **Frontend (S3 + CloudFront):** ~$0.00 USD/month (Free Tier).
- **ECS Fargate Spot (Backend):** ~$4.00 - $6.00 USD/month.
- **EC2 GPU Spot (AI Engine - g4dn.xlarge):** ~$3.00 USD/month (running only during exams, ~$0.15/hour).
- **NAT Instance (t4g.nano):** ~$2.50 USD/month.
- **Application Load Balancer (ALB):** ~$16.00 USD/month.
- **MongoDB Atlas:** ~$0.00 USD/month (M0 Free Tier).
- **VPC Endpoints, SES & Others:** ~$2.00 USD/month.

**Total Estimation:** Approximately $28.00 - $30.00 USD/month.

### 7. Risk Assessment

**Risk Matrix**

- **Sudden Spot Instance Termination**: High impact, medium probability. (AWS can reclaim Spot instances at any time).
- **High Camera Network Latency**: Medium impact, high probability (Depends on the student's internet connection).
- **Billing Overage**: Medium impact, low probability (If EC2 GPU is left running).

**Mitigation Strategies**

- **Spot Instances**: Configure Auto Scaling groups or use scripts to automatically fallback to On-Demand instances if Spot capacity is unavailable.
- **Latency**: Optimize video resolution sent to the server (e.g., 480p instead of 1080p) and compress video before transmission.
- **Costs**: Set up AWS Budgets Alarms to trigger email notifications when costs exceed $10 and $20.

### 8. Expected Outcomes

- **Technical Improvements:** Delivers an automated, real-time AI proctoring solution with a highly available Cloud-Native architecture, optimizing bandwidth via WebSockets.
- **Long-term Value:** The platform can be packaged as a SaaS solution for universities and training centers, solving the online cheating problem with significantly lower operational costs than existing alternatives.
