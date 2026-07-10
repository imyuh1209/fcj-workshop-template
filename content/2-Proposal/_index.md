---
title: "Project Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AuraAcademic

## AI-Powered Online Examination and Proctoring Platform on AWS

### 1. Executive Summary

**AuraAcademic** is an advanced online examination platform that integrates **AI-Powered Proctoring** to automate monitoring and ensure absolute fairness and transparency for remote exams. By applying the **YOLOv8** (Nano version) deep learning model for real-time video stream analysis combined with the cost-optimized cloud architecture of **Amazon Web Services (AWS)**, AuraAcademic delivers a comprehensive solution: ultra-low latency, flexible scalability, and especially optimized operational costs (reduced by up to 70-80% through the use of Spot Instances and a Cost-Optimized architecture). The project aims to provide a powerful, accessible tool for modern educational institutions.

### 2. Problem Statement

**Current Challenges:**
The shift towards online education brings an urgent need for remote examination organization. However, traditional Learning Management Systems (LMS) lack effective automated monitoring mechanisms, leading to an increase in cheating rates. Meanwhile, existing AI-integrated proctoring solutions on the market (such as ProctorU, Respondus) face many barriers:

- Licensing costs are too high for small and medium-sized educational institutions.
- Strict hardware configuration requirements or excessive end-user bandwidth consumption.
- Complex integration capabilities and a lack of flexibility.

**Proposed Solution:**
AuraAcademic thoroughly solves this problem through a synchronized and optimized ecosystem on AWS:

- **Frontend (Next.js):** Statically distributed globally via **Amazon S3 & CloudFront**, ensuring a smooth user experience, fast page load times in all geographic locations, and HTTPS security.
- **Backend Core API (Spring Boot):** Handles core business logic (exams, submissions, authentication) running on **Amazon ECS Fargate Spot**, automatically scaling according to actual traffic.
- **AI Proctoring Engine (FastAPI + YOLOv8 Nano):** Exploits the efficient CPU computing power of **Amazon EC2 (t3.medium Spot)** to process WebSockets streams directly from candidates' devices. The system automatically detects abnormal behaviors (strangers appearing, cheating, leaving the frame, etc.) in real time.
- **Evidence Management:** Automatically records and securely stores video clips of violations via databases and cloud storage.

Automating the monitoring process helps training institutions significantly save on traditional proctoring resources. AuraAcademic's breakthrough lies in the perfect balance between Security and Cost Optimization. The system fully utilizes AWS Spot Instances and deploys core processing flows entirely in **Private Subnets** (following Enterprise standards). At a pilot scale (around 100 concurrent students), the total infrastructure maintenance cost (including NAT Gateway and Load Balancer) ranges from only **58 - 64 USD/month**, which is still extremely economical compared to expensive commercial solutions.

### 3. Solution Architecture

AuraAcademic strictly adheres to the **Microservices** and **Cloud-Native** architecture, clearly separating the Web processing flow (REST API) and the Video analysis flow (WebSockets).

**Core AWS Services:**

- **Amazon S3 & CloudFront:** Static Hosting for the Frontend, automated CI/CD, and global Content Delivery Network (CDN) setup.
- **Application Load Balancer (ALB):** The single entry point for the application, intelligently routing the REST API to ECS and WebSockets to EC2.
- **Amazon ECS (Fargate Spot):** A Serverless Container environment running Spring Boot, requiring no physical server management.
- **Amazon EC2 (t3.medium Spot):** Provides optimal CPU processing power for the YOLOv8 Nano model analyzing Computer Vision.
- **AWS WAF & Shield Standard:** A multi-layered defense against DDoS attacks and web vulnerability exploits (OWASP Top 10).

- **VPC & Private Subnets:** Building a standard Enterprise network architecture. Core compute resources (Backend ECS, AI EC2) are completely hidden in Private Subnets to avoid security risks from the Internet. These resources access the Internet through a NAT Gateway and are strictly protected by Security Groups.

**Auxiliary External Services:**

- **MongoDB Atlas:** A flexible NoSQL database, managed as a managed service (Serverless/Free Tier).
- **Google Gemini API / Groq API:** Integrates Generative AI to support teachers in automatically extracting and generating questions from raw documents.

### 4. Technical Implementation

The project is designed to be completed within a 3-month development cycle, applying the Agile methodology.

**Core Phases:**

- **Phase 1 (Design & Infrastructure Initialization):** Sketching the system architecture, setting up AWS VPC, modeling the MongoDB database, and training/fine-tuning the YOLOv8 model configuration.
- **Phase 2 (Business Development & AI Integration):** Building the Core API system using Spring Boot, programming the AI Engine with FastAPI to process WebSockets. Integrating Generative AI to automate the exam creation process.
- **Phase 3 (Completion & Cloud Deployment):** Developing the Next.js interface, packaging Containers, and pushing the entire system to the actual AWS environment via a CI/CD Pipeline (GitHub Actions). Conducting Stress Tests and handing over.

**Technology Stack:**

- **Frontend:** Next.js (React), WebSockets Client, MediaDevices API (Browser recording).
- **Backend:** Java Spring Boot 3, Spring Security, MongoDB Data.
- **AI/Computer Vision:** Python, FastAPI, OpenCV, Ultralytics YOLOv8 Nano.
- **DevOps & Cloud:** Docker, GitHub Actions, AWS CloudFormation / Terraform (optional Infrastructure as Code management).

### 5. Roadmap & Milestones

- **Weeks 1-2 (Foundation):** Unifying requirements, designing the system architecture (Architecture Diagram), sketching UI/UX, and setting up source code repositories.
- **Weeks 3-5 (Core Features):** Completing core features (Exam creation, question bank management, exam flow) and integrating question extraction using Gemini.
- **Weeks 6-9 (AI Proctoring):** Developing the AI module, receiving and analyzing WebSockets camera streams from multiple clients simultaneously, optimizing FPS.
- **Weeks 10-12 (Deployment & Validation):** Deploying the entire system to AWS. Building test scenarios simulating Spot Instances loss. Packaging acceptance report documentation.

### 6. Budget Estimation

The budget is calculated for a small/pilot environment (~100 concurrent connections), closely following the Standard Enterprise Architecture which includes Private Subnets and a NAT Gateway:

| Infrastructure Category | AWS Service              | Estimated Cost (Monthly) | Notes                               |
| :---------------------- | :----------------------- | :----------------------- | :---------------------------------- |
| **Frontend Hosting**    | S3 + CloudFront          | ~$0.00                   | Using Free Tier                     |
| **Core API Compute**    | ECS Fargate (Spot)       | ~$4.00 - $6.00           | 70% optimized over On-demand price  |
| **AI Processing**       | EC2 CPU (t3.medium Spot) | ~$4.00 - $8.00           | Running YOLOv8 Nano (CPU)           |
| **Load Balancing**      | ALB                      | ~$16.00                  | Routing HTTP/WebSockets             |
| **Networking**          | NAT Gateway & Elastic IP | ~$32.50                  | Private -> Internet transit station |
| **Database**            | MongoDB Atlas (M0)       | ~$0.00                   | Free Tier                           |
| **Other**               | Amazon CloudWatch        | ~$2.00                   | System logging & monitoring         |
| **Total**               |                          | **~$58.50 - $64.50**     |                                     |

### 7. Risk Assessment

| Risk                                             | Impact | Probability | Mitigation Strategy                                                                                                                                    |
| :----------------------------------------------- | :----- | :---------- | :----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Sudden AWS Spot Instance recall**              | High   | Medium      | Configure ECS Service to automatically replace Tasks. For EC2, set up an automatic fallback script to On-Demand Instances when the Spot pool is empty. |
| **Video transmission latency (Network Latency)** | Medium | High        | Optimize client: Only send low-resolution (480p) frames, apply JPEG/WebP compression before pushing via WebSockets.                                    |
| **Unexpected costs (Billing Shock)**             | Medium | Low         | Set up AWS Budgets and Billing Alarms. Automatically send alerts via Email/Slack when costs exceed $10, $20. Strictly manage EC2 lifecycle.            |

### 8. Expected Outcomes

- **Technical Breakthrough:** Successfully building a distributed, real-time AI online proctoring solution, proving the ability to apply a Cloud-Native architecture and wide-area WebSockets processing techniques at an extremely low cost.
- **Practical & Commercial Value:** The platform has the full potential to be packaged into a Software as a Service (SaaS) solution. Thereby, providing schools and training centers with a highly reliable online anti-cheating tool with operational costs that are only a fraction of traditional commercial platforms.
