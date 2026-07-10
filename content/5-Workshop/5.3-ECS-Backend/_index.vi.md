---
title: "Giai đoạn 3: Triển khai Backend (ECS Fargate)"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3 </b> "
---

# Triển khai Backend (ECS Fargate) chuẩn Enterprise

Trong giai đoạn này, chúng ta sẽ thiết lập hạ tầng cốt lõi cho ứng dụng Backend Spring Boot, đặt nó an toàn bên trong Private Subnets.

---

### Bước 1: Khởi tạo Kho chứa Docker Image (Amazon ECR)

1. Truy cập dịch vụ **Elastic Container Registry (ECR)**.
2. Nhấn **Create repository**.
3. Visibility settings: Chọn **Private**.
4. Repository name: **`aura-academic-be`**.
5. Nhấn **Create repository**.

![Create ECR](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step1.png)

---

### Bước 2: Thiết lập Application Load Balancer (ALB)

ALB sẽ đóng vai trò "Lễ tân", đứng ở Public Subnet đón request và chuyển vào Private Subnet.

1. Truy cập dịch vụ **EC2**, menu trái chọn **Load Balancers** -> **Create load balancer**.
2. Chọn **Application Load Balancer**.
3. Tên: `aura-academic-alb`.
4. Scheme: Đảm bảo đã tick chọn **Internet-facing** (Để ALB giao tiếp ra ngoài mạng internet).
5. IP address type: Đảm bảo tick chọn **IPv4**.
6. Network mapping:
   - VPC: Chọn `aura-academic`.
   - Mappings: Chọn **CẢ 2 PUBLIC SUBNETS** (`Public-1a` và `Public-1b`). Bắt buộc phải là 2 vùng khác nhau.
7. Security groups: Bỏ default, chọn `aura-academic-alb-sg`.
8. Listeners and routing:
   - Click vào dòng chữ nhỏ màu xanh lam **`create target group`** (Nằm dưới chữ Forward to target group). Trình duyệt sẽ mở ra một tab mới.
   - Ở tab mới thiết lập như sau:
     - Target type: Bắt buộc chọn **IP addresses** (Vì Fargate dùng IP chứ không phải Instance).
     - Target group name: `aura-academic-ecs-tg`
     - Port: `8080` (Port mà Backend Spring Boot đang chạy).
     - VPC: Chọn `aura-academic`.
     - Health checks: Sửa đường dẫn thành **`/api/health`** (Đây là endpoint check sức khỏe chuẩn của Backend hiện tại).
     - Cuộn xuống cuối bấm **Next** -> Bấm tiếp **Create target group**.
   - Tạo xong, bạn **quay lại tab cũ** (tab tạo Load Balancer).
   - Bấm vào nút hình **mũi tên xoay tròn (Refresh)** bên cạnh ô tìm kiếm.
   - Click vào ô tìm kiếm và chọn cái **`aura-academic-ecs-tg`** vừa tạo.
9. Nhấn **Create load balancer**.

![Create ALB](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step2.png)

---

### Bước 3: Tạo Cụm tính toán (ECS Cluster)

1. Truy cập dịch vụ **Elastic Container Service (ECS)** -> **Clusters** -> **Create cluster**.
2. Tên Cluster: `aura-academic-cluster`.
3. Infrastructure: Chọn **AWS Fargate** (Serverless).
4. Nhấn **Create**.

![Create Cluster](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step3.png)

---

### Bước 4: Tạo Khuôn mẫu chạy Code (Task Definition)

1. Trong giao diện ECS, menu trái chọn **Task definitions** -> **Create new task definition**.
2. Tên: `aura-academic-task`.
3. Launch type: AWS Fargate.
4. OS, Architecture, Network: Linux/X86_64, AWSVPC.
5. CPU: `1 vCPU`, Memory: `2 GB`.
6. Container details:
   - Name: `backend-container`.
   - Image URI: Dán URI của ECR vừa tạo ở Bước 1.
   - Container port: `8080`.
7. Nhấn **Create**.

![Task Definition](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step4.png)

---

### Bước 5: Triển khai Dịch vụ (ECS Service) vào Private Subnet

Đây là bước quan trọng nhất để đạt chuẩn Enterprise.

1. Click vào cái tên màu xanh **`aura-academic-cluster`** (Như trong màn hình danh sách Clusters).
2. Khi giao diện chi tiết của Cluster mở ra, nhìn xuống dưới sẽ thấy tab **Services**. Bấm nút **Create** ở trong tab đó.
3. Compute options: Launch type (Fargate).
4. Task definition: Chọn `aura-academic-task`.
5. Service name: **`aura-academic-backend-enterprise`** (Khác tên cũ để chạy song song 2 cái - Zero Downtime).
6. Desired tasks: `1`.
6. **Networking (QUAN TRỌNG):**
   - VPC: `aura-academic`.
   - Subnets: Xóa các public subnets mặc định đi. **CHỈ CHỌN 1 PRIVATE SUBNET (`Private-1a`)** (Bắt buộc phải chọn Subnet nằm chung Availability Zone với NAT Gateway để có internet).
   - Security group: Chọn `aura-academic-ecs-sg`.
   - **Public IP: Tắt (DISABLED)**. Hệ thống sẽ đi ra mạng qua NAT Gateway.
7. Load balancing:
   - Type: Application Load Balancer.
   - Container: `backend-container: 8080`.
   - Application Load Balancer: Chọn **Use an existing load balancer** -> Chọn `aura-academic-alb`.
   - Listener: Chọn **Use an existing listener** -> Chọn `80 HTTP`.
   - Target group: Chọn **Use an existing target group** -> Chọn `aura-academic-ecs-tg`.
8. Nhấn **Create**.

![Create Service](/images/5-Workshop/5.3-ECS-Backend/5.3-ecs-step5.png)

Backend của bạn giờ đây đã được bảo vệ tối đa và chỉ tiếp xúc với thế giới bên ngoài thông qua ALB!
