---
title: "Dọn dẹp tài nguyên (Cleanup)"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7 </b> "
---

# Dọn dẹp tài nguyên (Cleanup)

Sau khi hoàn thành đồ án và bảo vệ thành công, bạn **BẮT BUỘC** phải xóa các tài nguyên trên AWS để tránh việc thẻ tín dụng của bạn bị trừ tiền vào những tháng tiếp theo.

Dưới đây là thứ tự xóa tài nguyên an toàn (xóa từ ngoài vào trong):

### 1. Xóa Application Load Balancer (ALB) và Target Groups
1. Truy cập EC2 -> **Load Balancers**.
2. Chọn `aura-academic-alb`, nhấn **Actions** -> **Delete**.
3. Chuyển sang **Target Groups**, chọn cả 2 nhóm `aura-academic-tg-ecs` và `aura-academic-tg-ec2`, nhấn **Actions** -> **Delete**.

### 2. Xóa ECS Cluster và ECR
1. Truy cập **ECS**, vào cụm `aura-academic-cluster`.
2. Ở tab Services, chọn `aura-academic-be-service`, bấm **Update**, đổi số **Desired tasks** về `0` và lưu lại (Để tắt các container đang chạy).
3. Sau khi tasks báo 0, chọn service và bấm **Delete**.
4. Ở góc trên bên phải màn hình Cluster, bấm **Delete cluster**.
5. Truy cập **ECR**, tick chọn kho `aura-academic-be` và bấm **Delete** (nhập chữ 'delete' để xác nhận).

### 3. Xóa máy chủ AI (EC2)
1. Truy cập **EC2** -> **Instances**.
2. Chọn máy `aura-academic-ai-server`.
3. Bấm **Instance state** -> **Terminate instance**. Chờ một lát để máy bị hủy hoàn toàn.

### 4. Xóa Frontend (S3 & CloudFront)
1. Truy cập **CloudFront**, chọn Distribution `aura-academic-fe-cdn`, bấm **Disable**. Chờ khoảng 2-3 phút cho quá trình disable hoàn tất.
2. Sau đó chọn lại Distribution này và bấm **Delete**.
3. Truy cập **S3**, vào bucket `aura-academic-fe-2024`.
4. Bấm nút **Empty** để xóa toàn bộ file bên trong (bạn sẽ cần gõ chữ `permanently delete`).
5. Sau khi rỗng, quay ra danh sách Bucket, chọn nó và bấm **Delete**.

### 5. Xóa NAT Gateway & Mạng ảo (VPC)

🔥 **Cảnh báo Đỏ:** NAT Gateway và Elastic IP (EIP) nếu để quên sẽ bị trừ tiền khá "đau ví" (hơn 1$/ngày). Bạn **bắt buộc phải xóa thủ công** chúng trước khi xóa VPC, vì lệnh xóa VPC sẽ bị chặn nếu NAT vẫn còn sống.

**A. Xóa NAT Gateway & Giải phóng IP:**
1. Ở menu bên trái của dịch vụ VPC, cuộn xuống phần Virtual private cloud, chọn **NAT gateways**.
2. Chọn NAT Gateway của `aura-academic`, bấm **Actions** -> **Delete NAT gateway** (gõ chữ `delete` để xác nhận).
3. **Pha trà, ngồi đợi khoảng 2-3 phút** cho đến khi trạng thái của NAT chuyển hẳn sang *Deleted*.
4. Ngay sau đó, chọn **Elastic IPs** ở menu bên trái. Chọn địa chỉ IP vừa được thả rông, bấm **Actions** -> **Release Elastic IP addresses**. *(Tuyệt đối không quên bước này, IP thả rông trên AWS sẽ bị tính tiền rác)*.

**B. Xóa VPC (Bước cuối cùng):**
1. Truy cập lại **VPC** -> **Your VPCs**.
2. Chọn `aura-academic`, bấm **Actions** -> **Delete VPC**. Lệnh quyền năng này sẽ tự động dọn dẹp quét sạch toàn bộ Subnets, Route Tables, Internet Gateway và các Security Groups còn sót lại.

---

🎉 **Chúc mừng!** Bạn đã dọn dẹp sạch sẽ không tì vết toàn bộ môi trường Cloud. Chuyến tàu thực hành kiến trúc Enterprise của AuraAcademic đến đây là kết thúc hoàn mỹ! Đêm nay ngủ ngon không lo thẻ Visa báo tin nhắn trừ tiền nhé!
