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

**AuraAcademic** is an advanced online examination platform that integrates **AI-powered proctoring** to automate exam monitoring and ensure fairness and integrity in remote assessments. By utilizing the **YOLOv8** (Nano version) deep learning model for real-time video analysis together with a cost-optimized **Amazon Web Services (AWS)** cloud architecture, the platform delivers low latency, high scalability, and an incredibly cost-effective infrastructure. Through the use of AWS Spot Instances and a Public Subnet-only architecture, operational costs can be reduced by up to **70-80%** compared to traditional deployments. AuraAcademic aims to provide educational institutions with an affordable, scalable, and intelligent online examination solution.

---

## 2. Problem Statement

### Current Challenges

The rapid growth of online education has significantly increased the demand for reliable remote examination systems. However, traditional Learning Management Systems (LMS) often lack automated monitoring capabilities, making it difficult to prevent academic dishonesty.

Existing AI-based online proctoring solutions have several limitations:

- High licensing and subscription costs.
- High hardware and network bandwidth requirements.
- Limited flexibility for customization and integration.

### Proposed Solution

AuraAcademic addresses these challenges through a cloud-native architecture built on AWS.

- **Frontend (Next.js):** Hosted on **Amazon S3 & CloudFront** for global CDN distribution, HTTPS security, and fast content delivery.
- **Backend API (Spring Boot):** Handles authentication, examinations, submissions, and business logic on **Amazon ECS Fargate Spot**.
- **AI Proctoring Engine (FastAPI + YOLOv8 Nano):** Runs on **Amazon EC2 (t3.medium Spot)** leveraging efficient CPU computing to analyze webcam streams in real time through WebSockets.
- **Evidence Management:** Stores suspicious screenshots and video clips securely for later review.

### Business Value

By automating the online proctoring process, educational institutions can significantly reduce manual supervision costs while maintaining examination integrity.

For approximately **100 concurrent students**, the estimated monthly infrastructure cost is only **USD 26–32**, completely eliminating expensive elements like NAT Gateways and GPUs, making the solution much more affordable than existing commercial platforms.

---

## 3. Solution Architecture

AuraAcademic follows a **Microservices** and **Cloud-Native Architecture**, separating web services from AI processing services.

### Core AWS Services

- **Amazon S3 & CloudFront**
  - Static website hosting and CI/CD
  - Global Content Delivery Network (CDN)

- **Application Load Balancer (ALB)**
  - Routes REST API requests
  - Routes WebSocket traffic to AI servers

- **Amazon ECS Fargate Spot**
  - Runs Spring Boot backend services
  - Automatically scales based on workload

- **Amazon EC2 t3.medium Spot**
  - CPU-optimized instances for YOLOv8 Nano inference

- **AWS WAF & AWS Shield Standard**
  - Protection against DDoS attacks
  - Web application security


- **Amazon VPC**
  - Public Subnets (Cost-optimized)
  - Highly secured using strict Security Groups (No NAT Gateway required)

### External Services

- MongoDB Atlas
- Google Gemini API
- Groq API

---

## 4. Technical Implementation

The project is planned to be completed within **three months** following the Agile development methodology.

### Phase 1 – Planning & Infrastructure

- Requirement analysis
- AWS network architecture
- MongoDB schema design
- YOLOv8 model preparation

### Phase 2 – Core Development

- Spring Boot REST API
- Authentication system
- AI Proctoring Engine
- Question generation using Generative AI

### Phase 3 – Deployment

- Next.js frontend
- Docker containerization
- GitHub Actions CI/CD
- AWS deployment
- Performance and stress testing

### Technology Stack

**Frontend**

- Next.js
- React
- WebSocket
- MediaDevices API

**Backend**

- Java Spring Boot
- Spring Security
- MongoDB

**AI**

- Python
- FastAPI
- OpenCV
- YOLOv8 Nano

**DevOps**

- Docker
- GitHub Actions
- AWS CloudFormation / Terraform

---

## 5. Roadmap and Milestones

### Weeks 1–2

- Requirement analysis
- Architecture design
- UI/UX design
- Repository setup

### Weeks 3–5

- Authentication
- Examination management
- Question bank
- AI-assisted question generation

### Weeks 6–9

- AI proctoring development
- Real-time WebSocket processing
- Performance optimization

### Weeks 10–12

- AWS deployment
- CI/CD pipeline
- Load testing
- Documentation

---

## 6. Budget Estimation

The following estimation is based on approximately **100 concurrent users**, strictly following the Standard Enterprise Architecture diagram which includes Private Subnets and a NAT Gateway.

| Infrastructure   | AWS Service               | Monthly Cost      | Notes                       |
| ---------------- | ------------------------- | ----------------- | --------------------------- |
| Frontend Hosting | Amazon S3 & CloudFront    | ~$0.00            | AWS Free Tier               |
| Backend Compute  | ECS Fargate Spot          | ~$4–6             | Spot pricing                |
| AI Processing    | EC2 t3.medium Spot        | ~$4–8             | YOLOv8 Nano (CPU Inference) |
| Load Balancer    | Application Load Balancer | ~$16              | HTTP & WebSocket routing    |
| Networking       | NAT Gateway & Elastic IP  | ~$32.50           | Private to Public routing   |
| Database         | MongoDB Atlas             | ~$0.00            | Free Tier                   |
| Other Services   | Amazon CloudWatch         | ~$2               | System logging & monitoring |
| **Total**        |                           | **~$58.50–64.50** | Standard Enterprise setup   |

---

## 7. Risk Assessment

| Risk                       | Impact | Probability | Mitigation                                                |
| -------------------------- | ------ | ----------- | --------------------------------------------------------- |
| Spot Instance interruption | High   | Medium      | Automatic replacement and fallback to On-Demand instances |
| High network latency       | Medium | High        | Compress webcam frames before transmission                |
| Unexpected AWS costs       | Medium | Low         | AWS Budgets and Billing Alarms                            |

---

## 8. Expected Outcomes

### Technical Outcomes

- Develop a real-time AI-powered online proctoring platform.
- Demonstrate the application of Cost-Optimized Cloud-Native Architecture on AWS.
- Build a scalable microservices system using AWS services without incurring massive infrastructure costs.

### Practical Outcomes

- Provide educational institutions with an affordable online examination platform.
- Reduce infrastructure costs dramatically through AWS Spot Instances and smart networking.
- Improve examination fairness using automated AI monitoring.
- Establish a solid foundation for future SaaS commercialization.
