---
title: "Giai đoạn 5: Lưu trữ Frontend bằng S3 & CloudFront"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5 </b> "
---

# Triển khai Frontend (Next.js) với S3 & CloudFront

Mảng Frontend của ứng dụng (Next.js) sẽ được build ra các tệp tin tĩnh (HTML, CSS, JS) và lưu trữ không giới hạn trên Amazon S3, sau đó phân phối với tốc độ siêu nhanh ra toàn cầu bằng Mạng phân phối nội dung (CDN) - Amazon CloudFront.

---

### Bước 1: Tạo Kho lưu trữ S3 Bucket

1. Truy cập dịch vụ **S3**, nhấn **Create bucket**.
2. Đặt tên Bucket (Ví dụ: `aura-academic-frontend-2024` - tên này phải là duy nhất trên toàn cầu).
3. Ở mục **Object Ownership**, cứ để mặc định `ACLs disabled`.
4. Ở mục **Block Public Access**, giữ nguyên dấu tick **Block all public access**. Hệ thống sẽ khóa chặt bucket này lại, chỉ cho phép CloudFront được chui vào lấy file (Bảo mật Enterprise).
5. Cuộn xuống dưới cùng và nhấn **Create bucket**.

![Create S3](/images/5-Workshop/5.5-S3-CloudFront-Frontend/5.5-s3-step1.png)

---

### Bước 2: Khai báo ALB & Đẩy mã nguồn tĩnh lên S3

Vì hệ thống Backend cũ đã bị gỡ bỏ, bạn cần khai báo cho Frontend biết "cánh cửa" mới của hệ thống (Load Balancer).

1. Mở file `.env` (hoặc `.env.local`, `.env.production`) trong mã nguồn Next.js.
2. Tìm biến `NEXT_PUBLIC_API_URL` (hoặc biến chứa domain gọi API) và đổi giá trị của nó thành **DNS name của ALB** (Ví dụ: `http://aura-academic-alb-12345.ap-southeast-1.elb.amazonaws.com`).
3. Mở Terminal và tiến hành Build dự án:

```bash
npm install
npm run build
```

Lệnh trên sẽ tự động nén toàn bộ giao diện thành các tệp tin tĩnh (HTML, CSS, JS) và đưa vào thư mục `out/`. 4. Mở AWS Console S3, upload **toàn bộ nội dung bên trong thư mục `out/`** vào S3 Bucket vừa tạo.

---

### Bước 3: Tạo Mạng phân phối CDN (CloudFront)

_(Giao diện CloudFront mới của AWS 2024)_

1. Truy cập dịch vụ **CloudFront**, nhấn **Create distribution**.
2. **Step 1 - Choose a plan**: Chọn gói **Free ($0/month)**. Bấm _Next_.
3. **Step 2 - Get started**: Ở mục **Distribution name**, bạn đặt tên là `aura-academic-fe-cdn`. Bấm _Next_.
4. **Step 3 - Specify origin**:
   - **Origin type**: Chọn **Amazon S3**.
   - **S3 origin**: Bấm nút **Browse S3** và chọn bucket của bạn. _(Lưu ý: Bỏ qua nút cảnh báo "Use website endpoint" màu vàng nếu có, ta bắt buộc phải dùng bucket endpoint để giữ bảo mật private cho S3)_.
   - Cuộn xuống phần **Origin access**: Cực kỳ quan trọng. Chọn **Origin access control settings (recommended)** -> Nhấn **Create control setting** -> **Create**.
   - Cuối cùng bấm _Next_.
5. **Step 4 - Enable security**: Để nguyên cấu hình bảo mật WAF mặc định. Bấm _Next_.
6. **Step 5 - Review and create**:
   - Nhờ tính năng mới của AWS, CloudFront sẽ tự động cập nhật Policy cho S3 Bucket luôn nên ta không cần làm bằng tay nữa.
   - Bạn chỉ việc nhấn **Create distribution** ở dưới cùng.

![Create CloudFront](/images/5-Workshop/5.5-S3-CloudFront-Frontend/5.5-s3-step3.png)

---

### Bước 4: Cấu hình bổ sung (Bắt buộc)

Vì giao diện tạo nhanh của AWS đã ẩn đi một số cài đặt, chúng ta cần bổ sung thủ công sau khi tạo xong:

**1. Khai báo file trang chủ (Default Root Object):**

- Tại trang chi tiết của CloudFront vừa tạo, ở tab **General**, bấm nút **Edit** ở mục Settings.
- Kéo xuống mục **Default root object**, gõ vào chữ `index.html`. Bấm **Save changes**.

**2. Ép trình duyệt dùng HTTPS (Để bật được Camera):**

- Chuyển sang tab **Behaviors**.
- Tick chọn cái behavior duy nhất đang có (Default (\*)), rồi bấm **Edit**.
- Kéo xuống mục **Viewer protocol policy**, đổi từ _HTTP and HTTPS_ sang **Redirect HTTP to HTTPS**. Bấm **Save changes**.

---

### Bước 5: Kiểm tra Website thành quả

Trở lại giao diện CloudFront, copy đường dẫn ở mục **Distribution domain name** (Ví dụ: `d1234abcd.cloudfront.net`).
Dán vào trình duyệt, bạn sẽ thấy ứng dụng Web của mình hiện lên mượt mà với ổ khóa bảo mật HTTPS, sẵn sàng cho luồng Camera.
