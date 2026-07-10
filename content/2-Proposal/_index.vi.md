---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AuraAcademic

## Nền tảng Thi trực tuyến và Giám sát bằng AI trên AWS

### 1. Tóm tắt điều hành

**AuraAcademic** là nền tảng tổ chức thi trắc nghiệm trực tuyến tiên tiến, tích hợp công nghệ Trí tuệ Nhân tạo (AI-Powered Proctoring) nhằm tự động hóa việc giám sát và đảm bảo tính công bằng, minh bạch tuyệt đối cho các kỳ thi từ xa. Bằng việc ứng dụng mô hình học sâu **YOLOv8** (phiên bản Nano) phân tích luồng video theo thời gian thực kết hợp với kiến trúc điện toán đám mây tối ưu của **Amazon Web Services (AWS)**, AuraAcademic mang đến một giải pháp toàn diện: độ trễ cực thấp, khả năng mở rộng linh hoạt và đặc biệt là tối ưu hóa chi phí vận hành (giảm tới 70-80% nhờ chiến lược sử dụng Spot Instances và kiến trúc Cost-Optimized). Dự án hướng tới việc cung cấp một công cụ mạnh mẽ, dễ tiếp cận cho các cơ sở giáo dục hiện đại.

### 2. Tuyên bố vấn đề

**Thách thức hiện tại:**
Sự chuyển dịch sang giáo dục trực tuyến kéo theo nhu cầu cấp thiết về việc tổ chức thi từ xa. Tuy nhiên, các hệ thống LMS (Learning Management System) truyền thống lại thiếu đi cơ chế giám sát tự động hiệu quả, khiến tỷ lệ gian lận gia tăng. Trong khi đó, các giải pháp giám sát có tích hợp AI hiện có trên thị trường (như ProctorU, Respondus) vấp phải nhiều rào cản:

- Chi phí cấp phép bản quyền quá cao đối với các tổ chức giáo dục quy mô vừa và nhỏ.
- Yêu cầu cấu hình phần cứng khắt khe hoặc tiêu tốn quá nhiều băng thông của người dùng cuối.
- Khả năng tích hợp phức tạp, thiếu tính linh hoạt.

**Giải pháp đề xuất:**
AuraAcademic giải quyết triệt để bài toán này thông qua một hệ sinh thái đồng bộ và tối ưu trên AWS:

- **Frontend (Next.js):** Được phân phối tĩnh toàn cầu qua **Amazon S3 & CloudFront**, đảm bảo trải nghiệm người dùng mượt mà, tải trang nhanh chóng ở mọi vị trí địa lý và bảo mật HTTPS.
- **Backend Core API (Spring Boot):** Xử lý nghiệp vụ lõi (đề thi, nộp bài, xác thực) vận hành trên **Amazon ECS Fargate Spot**, tự động mở rộng theo lưu lượng thực tế.
- **AI Proctoring Engine (FastAPI + YOLOv8 Nano):** Khai thác sức mạnh tính toán CPU hiệu quả của **Amazon EC2 (t3.medium Spot)** xử lý luồng WebSockets trực tiếp từ thiết bị thí sinh. Hệ thống tự động nhận diện các hành vi bất thường (người lạ xuất hiện, quay cóp, rời khỏi khung hình,...) theo thời gian thực.
- **Quản lý bằng chứng (Evidence Management):** Tự động ghi nhận và lưu trữ an toàn các đoạn video vi phạm thông qua cơ sở dữ liệu và lưu trữ đám mây.

Việc tự động hóa khâu giám sát giúp các cơ sở đào tạo tiết kiệm đáng kể nguồn lực giám thị truyền thống. Điểm đột phá của AuraAcademic nằm ở sự cân bằng hoàn hảo giữa Bảo mật và Tối ưu Chi phí. Hệ thống tận dụng triệt để AWS Spot Instances và triển khai các luồng xử lý lõi hoàn toàn trong **Private Subnet** (theo chuẩn Enterprise). Ở quy mô thử nghiệm (khoảng 100 sinh viên đồng thời), tổng chi phí duy trì cơ sở hạ tầng (đã bao gồm NAT Gateway và Load Balancer) chỉ dao động từ **58 - 64 USD/tháng**, vẫn vô cùng tiết kiệm so với các giải pháp thương mại đắt đỏ.

### 3. Kiến trúc giải pháp

AuraAcademic tuân thủ chặt chẽ kiến trúc **Microservices** và **Cloud-Native**, phân tách rõ ràng luồng xử lý Web (REST API) và luồng phân tích Video (WebSockets).

**Các dịch vụ AWS chủ đạo:**

- **Amazon S3 & CloudFront:** Lưu trữ tĩnh (Static Hosting) cho Frontend, CI/CD tự động và thiết lập Mạng phân phối nội dung (CDN) toàn cầu.
- **Application Load Balancer (ALB):** Điểm vào duy nhất cho ứng dụng, định tuyến thông minh REST API tới ECS và WebSockets tới EC2.
- **Amazon ECS (Fargate Spot):** Môi trường Container Serverless chạy Spring Boot, không cần quản lý máy chủ vật lý.
- **Amazon EC2 (t3.medium Spot):** Cung cấp năng lực xử lý CPU tối ưu cho mô hình YOLOv8 Nano phân tích Computer Vision.
- **AWS WAF & Shield Standard:** Lớp phòng thủ đa tầng chống lại các cuộc tấn công DDoS và khai thác lỗ hổng web (OWASP Top 10).

- **VPC & Private Subnets:** Xây dựng kiến trúc mạng chuẩn Enterprise. Các tài nguyên tính toán lõi (Backend ECS, AI EC2) được đặt ẩn hoàn toàn trong Private Subnets để tránh rủi ro bảo mật từ Internet. Các tài nguyên này ra Internet thông qua NAT Gateway và được bảo vệ nghiêm ngặt bằng Security Groups.

**Các dịch vụ ngoại vi bổ trợ:**

- **MongoDB Atlas:** Cơ sở dữ liệu linh hoạt (NoSQL), quản lý dưới dạng managed service (Serverless/Free Tier).
- **Google Gemini API / Groq API:** Tích hợp Generative AI để hỗ trợ giáo viên tự động bóc tách, sinh câu hỏi từ tài liệu thô.

### 4. Kế hoạch Triển khai Kỹ thuật

Dự án được thiết kế để hoàn thiện trong một chu kỳ phát triển 3 tháng, áp dụng phương pháp luận Agile.

**Các giai đoạn cốt lõi:**

- **Giai đoạn 1 (Thiết kế & Khởi tạo hạ tầng):** Phác thảo kiến trúc hệ thống, thiết lập AWS VPC, mô hình hóa cơ sở dữ liệu MongoDB và huấn luyện/tinh chỉnh cấu hình mô hình YOLOv8.
- **Giai đoạn 2 (Phát triển Nghiệp vụ & Tích hợp AI):** Xây dựng hệ thống Core API bằng Spring Boot, lập trình AI Engine bằng FastAPI xử lý WebSockets. Tích hợp Generative AI để tự động hóa khâu làm đề.
- **Giai đoạn 3 (Hoàn thiện & Triển khai Cloud):** Phát triển giao diện Next.js, đóng gói Container và đẩy toàn bộ hệ thống lên môi trường AWS thực tế thông qua CI/CD Pipeline (GitHub Actions). Tiến hành Stress Test và bàn giao.

**Stack công nghệ:**

- **Frontend:** Next.js (React), WebSockets Client, MediaDevices API (Ghi hình trình duyệt).
- **Backend:** Java Spring Boot 3, Spring Security, MongoDB Data.
- **AI/Computer Vision:** Python, FastAPI, OpenCV, Ultralytics YOLOv8 Nano.
- **DevOps & Cloud:** Docker, GitHub Actions, AWS CloudFormation / Terraform (tùy chọn quản lý hạ tầng as Code).

### 5. Lộ trình & Mốc thời gian

- **Tuần 1-2 (Foundation):** Thống nhất yêu cầu, thiết kế kiến trúc hệ thống (Architecture Diagram), phác thảo UI/UX và thiết lập kho mã nguồn.
- **Tuần 3-5 (Core Features):** Hoàn thiện các tính năng cốt lõi (Tạo kỳ thi, quản lý ngân hàng câu hỏi, luồng thi) và tích hợp bóc tách đề bằng Gemini.
- **Tuần 6-9 (AI Proctoring):** Phát triển module AI, nhận và phân tích luồng WebSockets camera từ nhiều client đồng thời, tối ưu FPS.
- **Tuần 10-12 (Deployment & Validation):** Triển khai toàn hệ thống lên AWS. Xây dựng kịch bản kiểm thử giả lập mất Spot Instances. Đóng gói tài liệu báo cáo nghiệm thu.

### 6. Ước tính Ngân sách

Ngân sách được tính toán cho một môi trường quy mô nhỏ/thử nghiệm (~100 kết nối đồng thời), bám sát theo Sơ đồ Kiến trúc Chuẩn (Enterprise Architecture) bao gồm hệ thống Private Subnets và NAT Gateway:

| Hạng mục Hạ tầng     | Dịch vụ AWS              | Chi phí ước tính (Tháng) | Ghi chú                               |
| :------------------- | :----------------------- | :----------------------- | :------------------------------------ |
| **Frontend Hosting** | S3 + CloudFront          | ~$0.00                   | Sử dụng Free Tier                     |
| **Core API Compute** | ECS Fargate (Spot)       | ~$4.00 - $6.00           | Tối ưu 70% giá On-demand              |
| **AI Processing**    | EC2 CPU (t3.medium Spot) | ~$4.00 - $8.00           | Chạy YOLOv8 Nano (CPU)                |
| **Load Balancing**   | ALB                      | ~$16.00                  | Định tuyến HTTP/WebSockets            |
| **Networking**       | NAT Gateway & Elastic IP | ~$32.50                  | Trạm trung chuyển Private -> Internet |
| **Database**         | MongoDB Atlas (M0)       | ~$0.00                   | Free Tier                             |
| **Khác**             | Amazon CloudWatch        | ~$2.00                   | Ghi log & giám sát hệ thống           |
| **Tổng cộng**        |                          | **~$58.50 - $64.50**     |                                       |

### 7. Đánh giá Rủi ro

| Rủi ro                                             | Mức độ Ảnh hưởng | Xác suất   | Chiến lược Giảm thiểu (Mitigation)                                                                                                                     |
| :------------------------------------------------- | :--------------- | :--------- | :----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AWS thu hồi Spot Instance đột ngột**             | Cao              | Trung bình | Định cấu hình ECS Service tự động thay thế Task. Đối với EC2, thiết lập script fallback tự động sang On-Demand Instance khi cạn kiệt Spot pool.        |
| **Độ trễ truyền tải Video (Network Latency)**      | Trung bình       | Cao        | Tối ưu hóa client: Chỉ gửi khung hình (frame) độ phân giải thấp (480p), áp dụng nén JPEG/WebP trước khi đẩy qua WebSockets.                            |
| **Phát sinh chi phí ngoài ý muốn (Billing Shock)** | Trung bình       | Thấp       | Thiết lập AWS Budgets và Billing Alarms. Tự động gửi cảnh báo qua Email/Slack khi chi phí vượt ngưỡng $10, $20. Quản lý chặt chẽ vòng đời của máy EC2. |

### 8. Kết quả Kỳ vọng

- **Đột phá về Kỹ thuật:** Xây dựng thành công một giải pháp giám sát thi cử phân tán, thời gian thực bằng AI, chứng minh khả năng áp dụng kiến trúc Cloud-Native và các kỹ thuật xử lý WebSockets diện rộng với chi phí siêu rẻ.
- **Giá trị Thực tiễn & Thương mại:** Nền tảng hoàn toàn có tiềm năng được đóng gói thành một giải pháp phần mềm dạng dịch vụ (SaaS). Qua đó, cung cấp cho các trường học, trung tâm đào tạo một công cụ chống gian lận trực tuyến có độ tin cậy cao với chi phí vận hành chỉ bằng một phần nhỏ so với các nền tảng thương mại truyền thống.
