---
title: "Triển khai Thực tế (Workshop)"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Hướng dẫn Triển khai AuraAcademic lên AWS (Step-by-Step)

#### Tổng quan

Trong phần này, chúng ta sẽ bắt tay vào thực hành triển khai toàn bộ kiến trúc hệ thống của **AuraAcademic** lên môi trường điện toán đám mây Amazon Web Services (AWS) thực tế.

<div align="center">

![Sơ đồ kiến trúc triển khai](/images/5-Workshop/BanFINAL.drawio.png)

**Sơ đồ kiến trúc: Triển khai hệ thống AuraAcademic trên AWS**

</div>

**Lưu ý quan trọng về Kiến trúc triển khai (Cost-Optimized vs Enterprise):**
- **Sơ đồ kiến trúc (Enterprise):** Trong thiết kế lý thuyết, chúng ta sử dụng **Private Subnets** và **NAT Gateways** để đảm bảo bảo mật tối đa (High Security & High Availability).
- **Triển khai thực tế (Cost-Optimized):** Do NAT Gateway có chi phí duy trì rất cao (~$86/tháng), trong bài thực hành này, chúng ta sẽ áp dụng **Kiến trúc Tối ưu chi phí cho sinh viên**. Các máy chủ ECS và EC2 sẽ được đặt trong **Public Subnets** và được bảo vệ nghiêm ngặt bằng **Security Groups (Tường lửa)**. Cách này giúp bạn hoàn thành đồ án xuất sắc với chi phí duy trì chỉ khoảng $30-$50/tháng (thậm chí chưa tới $10 nếu dùng Spot Instances).

#### Nội dung thực hành

Quá trình triển khai được chia thành 7 giai đoạn chính:

1. [Giai đoạn 1: Thiết lập Phân quyền (IAM)](5.1-iam-security/) - Tạo tài khoản và cấp quyền cho GitHub Actions, ECS.
2. [Giai đoạn 2: Khởi tạo Mạng ảo (VPC)](5.2-vpc-network/) - Xây dựng kiến trúc Public Subnets và Routing tối ưu chi phí.
3. [Giai đoạn 3: Triển khai Backend (ECS Fargate)](5.3-ecs-backend/) - Đưa Spring Boot API lên Container Serverless.
4. [Giai đoạn 4: Triển khai AI Engine (EC2 GPU)](5.4-ec2-gpu-ai/) - Cài đặt máy ảo chạy YOLOv8 và LiteLLM xử lý luồng WebSockets.
5. [Giai đoạn 5: Triển khai Frontend (S3 & CloudFront)](5.5-s3-cloudfront-frontend/) - Lưu trữ web tĩnh và CDN phân phối toàn cầu.
6. [Giai đoạn 6: Tự động hóa CI/CD (GitHub Actions)](5.6-cicd-github/) - Gắn kết quy trình đẩy code tự động.
7. [Dọn dẹp tài nguyên (Cleanup)](5.7-cleanup/) - Hướng dẫn xóa các dịch vụ để tránh phát sinh chi phí.

> **Lưu ý:** Hãy chuẩn bị sẵn sàng tài khoản AWS và bắt đầu đi tuần tự từng bước nhé!