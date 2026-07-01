---
title: "Worklog Tuần 2"
date: 2026-04-24
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Nắm được kiến thức cơ bản về mạng trong AWS thông qua VPC.
* Hiểu và thực hành triển khai máy chủ ảo với AWS EC2.
* Biết cách cấp quyền cho ứng dụng bằng IAM Role.
* Làm quen môi trường phát triển cloud với AWS Cloud9.
* Triển khai website tĩnh bằng AWS S3 và tối ưu bằng CDN.
* Hiểu cách hoạt động và sử dụng cơ sở dữ liệu với AWS RDS.

---

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|-----|----------|--------------|----------------|----------------|
| 2 | - Tìm hiểu AWS Networking <br> - Khái niệm VPC, Subnet, Security Group <br> - Tìm hiểu Site-to-Site VPN | 24/04/2026 | 24/04/2026 |  https://cloudjourney.awsstudygroup.com/ |
| 3 | - Thực hành EC2: <br>&emsp; + Tạo Linux & Windows instance <br>&emsp; + Kết nối SSH/RDP <br>&emsp; + Triển khai ứng dụng Node.js | 25/04/2026 | 25/04/2026 |  https://cloudjourney.awsstudygroup.com/
| 4 | - Tìm hiểu IAM Role cho EC2 <br> - So sánh Access Key và Role <br> - Thực hành gán Role cho EC2 | 26/04/2026 | 26/04/2026 |  https://cloudjourney.awsstudygroup.com/
| 5 | - Sử dụng AWS Cloud9: <br>&emsp; + Tạo môi trường dev <br>&emsp; + Sử dụng CLI trực tiếp trên cloud | 27/04/2026 | 27/04/2026 |  https://cloudjourney.awsstudygroup.com/
| 6 | - Hosting website tĩnh bằng AWS S3 <br> - Cấu hình public access <br> - Kết hợp AWS CloudFront để tăng tốc | 28/04/2026 | 28/04/2026 |  https://cloudjourney.awsstudygroup.com/
| 7 | - Tìm hiểu AWS RDS <br> - Tạo database instance <br> - Kết nối EC2 với RDS <br> - Backup & Restore | 29/04/2026 | 29/04/2026 |  https://cloudjourney.awsstudygroup.com/

---

### Kết quả đạt được tuần 2:

* Hiểu rõ kiến trúc mạng trong AWS:
  * Khái niệm VPC, Subnet  
  * Security Group và cơ chế firewall  
  * Cách kết nối mạng giữa các hệ thống  

* Thực hành thành công với AWS EC2:
  * Tạo và quản lý instance (Linux & Windows)  
  * Kết nối thông qua SSH và Remote Desktop  
  * Triển khai ứng dụng Node.js trên server  

* Hiểu và áp dụng IAM Role:
  * Phân biệt Access Key và Role  
  * Gán Role cho EC2 để truy cập tài nguyên AWS an toàn  
  * Áp dụng nguyên tắc bảo mật (không hardcode key)  

* Làm quen với môi trường phát triển AWS Cloud9:
  * Tạo môi trường lập trình trên cloud  
  * Sử dụng AWS CLI trực tiếp  
  * Quản lý code và test nhanh trên môi trường AWS  

* Triển khai website tĩnh với AWS S3:
  * Cấu hình Static Website Hosting  
  * Thiết lập quyền truy cập public  
  * Sử dụng AWS CloudFront để tăng tốc truy cập  
  * Hiểu cơ chế phân phối nội dung qua CDN  

* Làm việc với cơ sở dữ liệu AWS RDS:
  * Tạo database instance  
  * Kết nối ứng dụng từ EC2  
  * Thực hiện backup và restore dữ liệu  

---

### Tự đánh giá:

* Hoàn thành đầy đủ các lab quan trọng về Networking, Compute, Storage và Database.
* Có khả năng triển khai một hệ thống web cơ bản trên AWS bao gồm:
  * Frontend (S3 + CloudFront)  
  * Backend (EC2)  
  * Database (RDS)  

* Hiểu rõ hơn về cách các dịch vụ AWS kết hợp với nhau trong thực tế.

* Nhận thấy tầm quan trọng của:
  * Bảo mật (IAM Role, Security Group)  
  * Hiệu năng (CloudFront)  
  * Kiến trúc hệ thống  

* Cần tiếp tục:
  * Luyện tập thêm về kiến trúc hệ thống thực tế  
  * Tối ưu chi phí khi sử dụng nhiều dịch vụ  
  * Chuẩn bị cho việc xây dựng project hoàn chỉnh trong các tuần tiếp theo  

---
