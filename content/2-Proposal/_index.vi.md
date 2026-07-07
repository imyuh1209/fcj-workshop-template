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

AuraAcademic là nền tảng thi trắc nghiệm trực tuyến tích hợp AI giám sát (AI-Powered Proctoring & Exam Platform) nhằm đảm bảo tính công bằng và minh bạch cho các kỳ thi từ xa. Hệ thống ứng dụng mô hình YOLOv8 để phát hiện gian lận qua camera theo thời gian thực. Bằng việc kết hợp kiến trúc AWS (CloudFront, ECS Fargate, EC2 GPU Spot) và mô hình Serverless/Managed Services, AuraAcademic cung cấp một giải pháp giám sát tự động, độ trễ thấp và tối ưu hóa chi phí vận hành (có thể giảm tới 70% chi phí thông qua Spot Instances), phục vụ nhu cầu tổ chức thi trực tuyến cho các cơ sở giáo dục.

### 2. Tuyên bố vấn đề

**Vấn đề hiện tại**  
Các nền tảng thi trực tuyến hiện nay thiếu cơ chế giám sát tự động hiệu quả, dẫn đến tình trạng gian lận dễ dàng xảy ra. Các giải pháp giám sát có ứng dụng AI trên thị trường (như ProctorU, Respondus) thường có chi phí cấp phép rất đắt đỏ, tích hợp phức tạp, và đòi hỏi băng thông lớn hoặc tài nguyên máy chủ khổng lồ.

**Giải pháp**  
AuraAcademic giải quyết bài toán này bằng cách xây dựng một hệ thống hoàn chỉnh:

- **Frontend (Next.js)** lưu trữ tĩnh trên Amazon S3 và phân phối qua CloudFront, mang lại trải nghiệm mượt mà.
- **Backend Core API (Spring Boot)** chạy trên ECS Fargate Spot để xử lý logic đề thi, bài làm, và xác thực.
- **AI Proctoring (FastAPI + YOLOv8)** chạy trên máy ảo EC2 GPU (dòng G4dn) xử lý luồng WebSockets trực tiếp từ thí sinh để phát hiện hành vi gian lận (như quay cóp, có người lạ, rời khỏi khung hình).
- **Video Evidence** được lưu lên S3 thông qua NAT Instance để làm bằng chứng vi phạm an toàn.
- **Xác thực và Bảo mật** sử dụng Amazon Cognito, kết hợp AWS WAF và IAM.

**Lợi ích và hoàn vốn đầu tư (ROI)**  
Hệ thống tự động hóa quá trình giám sát, giảm thiểu nhân lực canh thi truyền thống. Bằng cách sử dụng Spot Instances cho cả ECS Fargate và EC2 GPU, hệ thống giảm thiểu chi phí hạ tầng tới 70% so với chạy On-Demand. Tổng chi phí duy trì hệ thống ở mức thử nghiệm/quy mô nhỏ chỉ khoảng 26 - 30 USD/tháng. Giải pháp này giúp các trường học/tổ chức giáo dục dễ dàng tiếp cận công nghệ giám sát thi cử tiên tiến với mức phí hợp lý, nâng cao uy tín của các kỳ thi trực tuyến.

### 3. Kiến trúc giải pháp

AuraAcademic áp dụng kiến trúc Microservices trên AWS, phân tách rõ ràng giữa Core API và AI Processing.

**Dịch vụ AWS sử dụng**

- **Amazon S3 & CloudFront**: Lưu trữ và phân phối ứng dụng web Frontend (Next.js) toàn cầu, lưu trữ video bằng chứng.
- **Amazon Cognito**: Quản lý xác thực người dùng bằng JWT.
- **ALB (Application Load Balancer)**: Định tuyến request REST API vào ECS và WebSockets vào EC2 GPU.
- **Amazon ECS (Fargate Spot)**: Chạy Spring Boot Backend container.
- **Amazon EC2 (G4dn Spot)**: Chạy FastAPI và model YOLOv8 xử lý video thời gian thực.
- **AWS WAF**: Tường lửa bảo vệ ứng dụng khỏi các cuộc tấn công độc hại.
- **Amazon SES**: Gửi email thông báo, OTP.
- **VPC, Public/Private Subnets & NAT Instance**: Đảm bảo phân tách mạng nội bộ và cấp phép truy cập Internet an toàn.
- **Dịch vụ ngoại vi**: MongoDB Atlas (Database), Google Gemini API (Sinh câu hỏi tự động).

**Thiết kế thành phần**

- **Giao diện Web**: Sinh viên làm bài và truyền luồng camera; Giáo viên quản lý đề thi và xem báo cáo vi phạm.
- **Core API**: Xử lý logic kỳ thi, xử lý bất đồng bộ (@Async) bóc tách đề thi bằng Gemini, và lưu trữ kết quả.
- **AI Engine**: Nhận stream camera qua WebSockets, nhận diện gian lận bằng YOLOv8 và tự động upload video vi phạm lên S3.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai (Dự án thực tập 3 tháng)**

- **Tháng 1 (Khởi tạo & Thiết kế)**: Phân tích kiến trúc VPC, thiết kế database MongoDB, chuẩn bị model YOLOv8.
- **Tháng 2 (Phát triển Backend & AI)**: Lập trình Spring Boot API, tích hợp Gemini bóc tách đề thi. Xây dựng FastAPI xử lý WebSockets camera.
- **Tháng 3 (Hoàn thiện & Triển khai)**: Xây dựng UI Next.js, đưa hệ thống lên AWS (S3, CloudFront, ECS, EC2 Spot). Kiểm thử và báo cáo nghiệm thu thực tập.

**Yêu cầu kỹ thuật**

- **Frontend**: Next.js, WebSockets client, ghi hình trên trình duyệt.
- **Backend**: Spring Boot, Spring Security, kết nối MongoDB Atlas.
- **AI**: FastAPI, OpenCV, YOLOv8 object detection, xử lý video streaming.
- **DevOps/AWS**: Docker, VPC (Public/Private Subnets, NAT Instance).

### 5. Lộ trình & Mốc triển khai

- **Tuần 1-2**: Phân tích thiết kế hệ thống, UI/UX và thiết lập code repository.
- **Tuần 3-5**: Phát triển tính năng cốt lõi (Quản lý kỳ thi) và bóc tách đề bằng Gemini.
- **Tuần 6-9**: Tích hợp AI YOLOv8 nhận diện luồng camera theo thời gian thực.
- **Tuần 10-12**: Triển khai lên AWS, kiểm thử tính chịu lỗi của Spot Instances và hoàn thiện tài liệu báo cáo thực tập.

### 6. Ước tính ngân sách

Sử dụng mô hình tối ưu chi phí với Spot Instances, ước tính cho môi trường Test/Quy mô nhỏ (~100 sinh viên đồng thời):

- **Frontend (S3 + CloudFront)**: ~0,00 USD/tháng (Free Tier).
- **ECS Fargate Spot (Backend)**: ~4,00 - 6,00 USD/tháng.
- **EC2 GPU Spot (AI Engine - g4dn.xlarge)**: ~3,00 USD/tháng (chỉ bật khi có lịch thi, giá ~0,15 USD/giờ).
- **NAT Instance (t4g.nano)**: ~2,50 USD/tháng.
- **Application Load Balancer (ALB)**: ~16,00 USD/tháng.
- **MongoDB Atlas**: ~0,00 USD/tháng (Gói M0 Free).
- **SES & Khác**: ~2,00 USD/tháng.
  **Tổng ước tính:** Khoảng 28 - 30 USD/tháng (Có thể cover hoàn toàn bằng Credit).

### 7. Đánh giá rủi ro

**Ma trận rủi ro**

- **Mất Spot Instance đột ngột**: Ảnh hưởng cao, xác suất trung bình. (Do AWS có thể thu hồi Spot máy ảo bất cứ lúc nào).
- **Độ trễ mạng camera lớn**: Ảnh hưởng trung bình, xác suất cao (Tùy thuộc đường truyền mạng của thí sinh).
- **Quá tải chi phí (Billing overage)**: Ảnh hưởng trung bình, xác suất thấp (Nếu quên tắt EC2 GPU).

**Chiến lược giảm thiểu**

- **Spot Instance**: Cấu hình Auto Scaling group hoặc dùng script tự động fallback sang On-Demand nếu Spot không khả dụng.
- **Độ trễ**: Tối ưu độ phân giải video gửi lên server (VD: 480p thay vì 1080p), nén video trước khi gửi.
- **Chi phí**: Đặt AWS Budgets Alarm cảnh báo qua email khi chi phí vượt ngưỡng 10 USD, 20 USD.

### 8. Kết quả kỳ vọng

- **Cải tiến kỹ thuật**: Cung cấp giải pháp giám sát thi tự động thời gian thực bằng AI với kiến trúc Cloud-Native có tính sẵn sàng cao, tối ưu hóa băng thông bằng WebSockets.
- **Giá trị dài hạn**: Nền tảng có thể đóng gói thành giải pháp SaaS bán cho các trường đại học/trung tâm đào tạo, giải quyết bài toán chống gian lận trực tuyến với chi phí vận hành rẻ hơn nhiều so với các giải pháp hiện tại.
