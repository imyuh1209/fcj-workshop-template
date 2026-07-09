---
title: "Giai đoạn 1: Thiết lập Phân quyền (IAM)"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1 </b> "
---

# Thiết lập Phân quyền (IAM)

Việc thiết lập Identity and Access Management (IAM) là nền tảng đầu tiên và quan trọng nhất để đảm bảo các dịch vụ trong kiến trúc **AuraAcademic** có thể giao tiếp với nhau một cách bảo mật, và hệ thống CI/CD (GitHub Actions) có đủ quyền để cập nhật mã nguồn.

Trong bước này, chúng ta sẽ tạo **IAM User** cho GitHub Actions và đảm bảo **ECS Task Execution Role** đã tồn tại.

---

### Bước 1: Tạo IAM User cho GitHub Actions

GitHub cần một tài khoản có quyền (Credentials) để tự động hóa quá trình đẩy Docker Image lên ECR và ra lệnh cho ECS cập nhật ứng dụng.

1. Đăng nhập vào **AWS Management Console**, trên thanh tìm kiếm gõ `IAM` và chọn **IAM**.
2. Ở menu bên trái, chọn **Users**, sau đó nhấn nút **Create user** (Tạo người dùng).
3. Tại phần **User details**:
   - Tên người dùng (User name): nhập `github-actions-deploy`.
   - Bấm **Next**.
4. Tại phần **Set permissions**:
   - Chọn tuỳ chọn **Attach policies directly**.
   - Trong ô tìm kiếm, hãy tìm và tick vào các quyền (policies) sau:
     1. `AmazonEC2ContainerRegistryFullAccess` (Quyền thao tác với kho chứa ảnh Docker - ECR)
     2. `AmazonECS_FullAccess` (Quyền cập nhật ứng dụng trên ECS)
     3. `AmazonS3FullAccess` (Quyền tải file giao diện Frontend lên S3)
     4. `CloudFrontFullAccess` (Quyền xoá cache CDN sau khi tải frontend mới)
5. Bấm **Next** đến trang Review, sau đó nhấn **Create user**.

![Tạo IAM User cho GitHub Actions](/images/5-Workshop/5.1-IAM-Security/1-create-user.png)

---

### Bước 2: Tạo Access Key (Credentials)

Sau khi tạo User thành công, chúng ta cần sinh ra mã khoá để cấp cho GitHub.

1. Bấm vào tên user `github-actions-deploy` vừa tạo.
2. Chuyển sang tab **Security credentials**.
3. Cuộn xuống phần **Access keys**, nhấn nút **Create access key**.
4. Chọn mục **Third-party service**, check vào ô đồng ý xác nhận, rồi bấm **Next**.
5. Nhấn **Create access key**.
6. **QUAN TRỌNG:** Màn hình sẽ hiển thị `Access key ID` và `Secret access key`.
   - Mở Notepad/Text editor và copy 2 đoạn mã này lưu lại.
   - _Secret access key sẽ không thể xem lại lần thứ hai sau khi bạn bấm Done!_

![Tạo Access Key thành công](/images/5-Workshop/5.1-IAM-Security/2-access-key.png)

---

### Bước 3: Kiểm tra ECS Task Execution Role

Khi ứng dụng Backend chạy trên Amazon ECS Fargate, nó cần mượn một Role của AWS để có quyền: Tải Docker Image từ ECR và ghi Log lên CloudWatch.

1. Ở menu bên trái của màn hình IAM, chọn **Roles**.
2. Trong ô tìm kiếm, gõ `ecsTaskExecutionRole`.
   - **Trường hợp 1:** Nếu role này đã tồn tại (do AWS tự sinh ra trước đó), bạn nhấn vào xem và kiểm tra xem nó đã có policy `AmazonECSTaskExecutionRolePolicy` hay chưa. Nếu có rồi, bạn đã hoàn tất!
   - **Trường hợp 2:** Nếu chưa có, bạn bấm **Create role**. Chọn Trusted entity type là **AWS service**, Use case chọn **Elastic Container Service** -> **Elastic Container Service Task**. Tìm và gán policy `AmazonECSTaskExecutionRolePolicy`. Đặt tên role là `ecsTaskExecutionRole` và tạo.

![Cấu hình ecsTaskExecutionRole](/images/5-Workshop/5.1-IAM-Security/3-ecs-role.png)

---

Sau khi hoàn thành tạo User và cấu hình các quyền cơ bản này, hệ thống của bạn đã sẵn sàng cho [Giai đoạn tiếp theo](../5.2-vpc-network).
