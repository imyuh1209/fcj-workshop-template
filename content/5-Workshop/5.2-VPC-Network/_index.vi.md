---
title: "Giai đoạn 2: Khởi tạo Mạng ảo (VPC)"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2 </b> "
---

# Khởi tạo Mạng ảo (VPC) chuẩn Enterprise

Amazon Virtual Private Cloud (VPC) là trái tim mạng lưới hạ tầng AWS của chúng ta. 
Trong bài thực hành này, chúng ta sẽ thiết lập kiến trúc mạng lưới đạt chuẩn **Enterprise (Doanh nghiệp)**: sử dụng Public Subnets để đón nhận request từ Internet (qua Load Balancer), và giấu hoàn toàn các Server xử lý (Backend, AI) vào trong Private Subnets, giao tiếp với bên ngoài thông qua NAT Gateway.

---

### Bước 1: Tạo VPC với "VPC and more"

Tính năng "VPC and more" của AWS giúp vẽ và khởi tạo toàn bộ hạ tầng mạng (VPC, Subnet, Route Tables, Internet Gateway, NAT Gateway) chỉ qua vài cú click.

1. Truy cập dịch vụ **VPC** trên giao diện AWS Console.
2. Bấm nút màu cam **Create VPC** ở góc trên cùng bên phải.
3. Chọn tùy chọn **VPC and more**.
4. Cấu hình các thông số như sau:
   - **Name tag auto-generation:** `aura-academic`
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - **Number of Availability Zones (AZs):** Chọn `2` (Bắt buộc phải có 2 AZ để cài đặt được Load Balancer).
   - **Number of public subnets:** Chọn `2`
   - **Number of private subnets:** Chọn `2`
   - **NAT gateways:** Chọn `1 in 1 AZ` (Tạo 1 trạm kiểm soát để các Private Subnet đi ra ngoài).
   - **VPC endpoints:** Chọn **None**.
5. Bấm nút **Create VPC** và chờ AWS hoàn tất khởi tạo.

![Create VPC](/images/5-Workshop/5.2-VPC-Network/5.2-vpc-step1.png)

---

### Bước 2: Kích hoạt cấp phát IP tự động cho Public Subnets

Để Load Balancer nằm trong Public Subnet tự động nhận được địa chỉ IP Public, ta cần đảm bảo tính năng này được bật (Thường AWS sẽ tự bật, nhưng ta cần kiểm tra lại):

1. Ở menu bên trái của dịch vụ VPC, chọn **Subnets**.
2. Tìm và tick chọn subnet thứ nhất có chữ `public` trong tên: **`aura-academic-subnet-public1-ap-southeast-1a`**.
3. Nhấn nút **Actions** (ở góc trên bên phải) -> Chọn **Edit subnet settings**.
4. Tick vào ô **Enable auto-assign public IPv4 address** -> **Save**.
5. Lặp lại y chang các bước 2-4 cho subnet public thứ hai: **`aura-academic-subnet-public2-ap-southeast-1b`**.
*(Lưu ý: Tuyệt đối không bật tính năng này cho 2 cái Subnets có chữ `private`).*

![Auto assign IP](/images/5-Workshop/5.2-VPC-Network/5.2-vpc-step2.png)

---

### Bước 3: Thiết lập Security Groups (Tường lửa)

Security Group (SG) đóng vai trò là "chốt chặn sống còn" để kiểm soát dòng dữ liệu ra vào giữa các Subnet.

1. Ở menu bên trái, cuộn xuống phần Security, chọn **Security groups** -> **Create security group**.
2. **ALB Security Group (Cho Load Balancer ở Public Subnet):**
   - Tên: `aura-academic-alb-sg`
   - VPC: Chọn VPC `aura-academic` vừa tạo.
   - Inbound rules: Cho phép `HTTP` (Port 80) và `HTTPS` (Port 443) từ `Anywhere-IPv4`.

![ALB Security Group](/images/5-Workshop/5.2-VPC-Network/5.2-vpc-step3-alb.png)

3. **ECS Backend Security Group (Cho vùng Private):**
   - Tên: `aura-academic-ecs-sg`
   - VPC: Chọn VPC `aura-academic`.
   - Inbound rules: 
     - Custom TCP, Port `8080`, Source: Click vào ô kính lúp và gõ `alb`, sau đó chọn đúng tên **`aura-academic-alb-sg`** (Chỉ cho phép traffic đi từ ALB vào Backend).

![ECS Security Group](/images/5-Workshop/5.2-VPC-Network/5.2-vpc-step3-ecs.png)

4. **EC2 AI Security Group (Cho vùng Private):**
   - Tên: `aura-academic-ai-sg`
   - VPC: Chọn VPC `aura-academic`.
   - Inbound rules: 
     - Custom TCP, Port `8001`, Source: Click vào ô kính lúp và gõ `alb`, sau đó chọn **`aura-academic-alb-sg`** (Cho luồng WebSockets từ ALB truyền vào).
     - Custom TCP, Port `4000`, Source: Click vào ô kính lúp và gõ `ecs`, sau đó chọn **`aura-academic-ecs-sg`** (Cho API nội bộ từ Backend gọi sang).
     - (Tuyệt đối **KHÔNG CẦN** mở Port 22 SSH vì chúng ta sẽ dùng tính năng Session Manager siêu bảo mật của AWS để connect thẳng vào Private Subnet).

![EC2 AI Security Group](/images/5-Workshop/5.2-VPC-Network/5.2-vpc-step3-ec2.png)

---

Đến đây, khung xương hạ tầng mạng Enterprise vững chắc của hệ thống đã sẵn sàng! Chúng ta có thể chuyển sang [Giai đoạn tiếp theo](../5.3-ecs-backend) để cấu hình Backend.
