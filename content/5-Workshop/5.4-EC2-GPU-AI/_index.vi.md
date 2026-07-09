---
title: "Giai đoạn 4: Cấu hình EC2 CPU cho AI"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4 </b> "
---

# Cấu hình EC2 CPU cho AI (YOLOv8 & WebSockets)

Để xử lý luồng WebSockets video từ camera của thí sinh, chúng ta sẽ dùng Amazon EC2 để chạy AI. Đặc biệt, ta sẽ giấu EC2 này vào vùng Private Subnet để an toàn tuyệt đối theo chuẩn Enterprise.

---

### Bước 1: Khởi tạo Máy chủ (Launch Instance)

1. Truy cập dịch vụ **EC2**, nhấn **Launch instance**.
2. Tên: `aura-academic-ai-server`.
3. Application and OS Images: Chọn **Ubuntu Server 24.04 LTS**.
4. Instance type: Để tối ưu chi phí nhưng vẫn đảm bảo sức mạnh, chọn **t3.medium** (2 vCPU, 4GB RAM) hoặc cao hơn. Đừng lo lắng về chi phí vì ta sẽ dùng Spot Instance.
5. Key pair: Nhấn *Create new key pair* (nếu chưa có), đặt tên là **`aura-academic-key`**, và tải file `.pem` về máy để phòng hờ (thực tế ta sẽ ưu tiên connect bằng Session Manager).
6. Configure storage: Mặc định là 8 GiB, bạn nên tăng lên **15 GiB** hoặc **20 GiB** (loại gp3) vì tải thư viện Python và Model AI khá tốn dung lượng.

![Launch Instance 1](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step1-1.png)

![Launch Instance 2](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step1-2.png)

---

### Bước 2: Cấu hình Network & Ẩn mình vào Private Subnet

1. Cuộn xuống phần **Network settings**, bấm **Edit**.
2. VPC: Chọn `aura-academic`.
3. Subnet: **BẮT BUỘC** chọn một **Private Subnet** (Ví dụ: `Private-1a`).
4. Auto-assign public IP: Bắt buộc chọn **Disable**.
5. Firewall (security groups): Chọn *Select existing security group*, tick vào `aura-academic-ai-sg`.

![Network Settings](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step2.png)

---

### Bước 3: Đăng nhập máy chủ không cần Public IP

Vì máy chủ nằm trong Private Subnet và không có IP Public, ta không thể dùng tính năng `EC2 Instance Connect` mặc định (như lỗi màu vàng AWS đang cảnh báo).

**Yêu cầu bắt buộc (Gắn IAM Role):** Để dùng được tính năng bảo mật cao hơn là SSM, con EC2 của bạn cần có quyền.
- Ra ngoài danh sách Instances, click chuột phải vào máy ảo -> **Security** -> **Modify IAM role**.
- Nếu bạn dùng tài khoản AWS Academy: Chọn thẻ `LabRole` -> Nhấn **Update IAM role**.
- **Nếu mục IAM Role trống trơn (như tài khoản cá nhân):**
  1. Click vào dòng chữ xanh **Create new IAM role** ngay bên cạnh.
  2. Trang mới hiện ra, bấm nút **Create role**. Chọn **AWS service** -> Chọn Use case là **EC2** -> Next.
  3. Ở ô tìm kiếm quyền, gõ `AmazonSSMManagedInstanceCore`, tick vào ô vuông bên cạnh cái tên đó -> Next.
  4. Đặt tên Role name là `aura-academic-ssm-role` -> Cuộn xuống dưới cùng bấm **Create role**.
  5. Quay lại tab EC2 cũ, bấm nút vòng tròn **Refresh** nhỏ nhỏ cạnh dropdown, chọn `aura-academic-ssm-role` vừa tạo -> Nhấn **Update IAM role**.

Sau khi gắn quyền, tiến hành kết nối:
1. Tick chọn máy `aura-academic-ai-server`, nhấn nút **Connect**.
2. Tại tab **In web browser**, nhìn xuống phần *Choose how to connect*, hãy chọn ô **SSM Session Manager** (Maximum security).
3. Nhấn nút **Connect** màu cam. Trình duyệt sẽ mở ra một cửa sổ Terminal đen ngòm. Bạn đã chui vào được Private Subnet thành công!

![Connect EC2](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step3.png)

---

### Bước 4: Cài đặt AI & FastAPI

1. Tại cửa sổ Terminal trình duyệt, tiến hành chạy lệnh update hệ thống. Máy tính sẽ tự động lên mạng tải các gói cập nhật (thông qua NAT Gateway):
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install python3-pip git libgl1 libglib2.0-0 -y
   ```
2. Cài đặt các thư viện AI và WebSockets cần thiết (Do Ubuntu 24.04 bảo mật cao chặn cài trực tiếp, ta cần thêm cờ `--break-system-packages` để ép cài đặt):
   ```bash
   pip3 install fastapi uvicorn websockets ultralytics opencv-python-headless --break-system-packages
   ```
3. Tải mã nguồn AI từ Github về máy chủ (nhớ chuyển về thư mục `home` trước khi clone để không bị lỗi từ chối quyền):
   ```bash
   cd ~
   git clone https://github.com/lengthanhdat/AuraAcademic_AI.git
   ```
4. Di chuyển vào thư mục code vừa tải và khởi chạy AI Engine ngầm (để chạy 24/24):
   ```bash
   cd AuraAcademic_AI
   nohup python3 -m uvicorn main:app --host 0.0.0.0 --port 8001 &
   ```

### Bước 5: Cấu hình Target Group và Load Balancer cho AI

Để hệ thống bên ngoài (Internet) có thể giao tiếp được với AI Engine đang chạy ngầm, bạn cần "bắc cầu" từ Application Load Balancer (ALB) vào EC2:

1. Vào AWS Console **EC2 -> Target Groups**, tạo mới một Target Group tên `aura-academic-ai-tg`.
2. Chọn Target type là **Instances**, Protocol là **HTTP**, Port là **8001**, và chọn VPC `aura-academic`.
3. Ở bước Register targets, tick chọn máy ảo `aura-academic-ai-server` và nhấn **Include as pending below**. Sau đó tạo Target Group.
4. Chuyển sang menu **Load Balancers**, tick chọn ALB hiện tại của bạn. Nhìn xuống nửa dưới màn hình, bấm sang tab **Listeners and rules** (ngay cạnh tab Details).
5. Bạn sẽ thấy một dòng Listener có Port là **80** (giao thức HTTP). Bấm vào liên kết ở cột **Rules** (hoặc click trực tiếp vào dòng Listener đó) -> Chọn **Add rule** (Thêm quy tắc).
6. Cấu hình quy tắc "bẻ lái" luồng AI như sau:
   - **Name**: Đặt tên là `ai-websocket-rule`.
   - **Condition** (Điều kiện): Chọn loại **Path** (đường dẫn), gõ `/ws/*` (để hứng tất cả các luồng phân tích hình ảnh AI từ Lễ tân).
   - **Action** (Hành động): Chọn **Forward** (chuyển tiếp) tới Target Group `aura-academic-ai-tg` mà bạn vừa tạo ở trên.
   - Bấm Create/Save để lưu lại. (Lúc này bạn cũng có thể tiện tay xóa cái rule cũ của EC2 port 8000 đi nếu muốn).

![Cấu hình ALB cho AI](/images/5-Workshop/5.4-EC2-GPU-AI/5.4-ec2-step5.png)
