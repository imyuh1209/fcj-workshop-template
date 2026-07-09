---
title: "Amazon S3 Files – Khi Amazon S3 không chỉ là Object Storage"
date: 2026-07-08
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---


## Giới thiệu

Nếu đã từng học hoặc làm việc với AWS, chắc hẳn bạn đều biết đến **Amazon S3 (Simple Storage Service)** – dịch vụ lưu trữ đối tượng (Object Storage) nổi tiếng của AWS với khả năng mở rộng gần như không giới hạn, độ bền dữ liệu lên tới **99.999999999% (11 số 9)** và chi phí tối ưu.

Tuy nhiên, trong nhiều năm qua, S3 luôn có một hạn chế đối với các ứng dụng truyền thống: **S3 là Object Storage chứ không phải File Storage**.

Điều đó có nghĩa là thay vì mở một file bằng những thao tác quen thuộc như `open()`, `read()` hay `write()`, lập trình viên phải sử dụng API hoặc SDK của AWS để làm việc với dữ liệu. Với các ứng dụng được thiết kế theo mô hình file system, đây là một rào cản không nhỏ.

Để giải quyết bài toán này, AWS đã giới thiệu **Amazon S3 Files** – một tính năng mới giúp các bucket Amazon S3 có thể được truy cập theo cách tương tự một hệ thống tệp (File System), giúp việc phát triển và triển khai ứng dụng trên AWS trở nên đơn giản hơn.

---

## Amazon S3 Files là gì?

Amazon S3 Files cho phép người dùng **mount một bucket Amazon S3 như một hệ thống tệp**, từ đó ứng dụng có thể thực hiện các thao tác quen thuộc như:

- Đọc file
- Ghi file
- Đổi tên
- Xóa file
- Duyệt thư mục

Điều quan trọng là **dữ liệu vẫn được lưu trữ dưới dạng object trong Amazon S3**. Amazon S3 Files chỉ cung cấp một lớp truy cập theo mô hình file system, giúp các ứng dụng làm việc với dữ liệu theo cách tự nhiên hơn mà không cần phải viết nhiều đoạn mã sử dụng AWS SDK.

Nói cách khác, Amazon S3 Files đóng vai trò như một "cầu nối" giữa **Object Storage** và **File System**.

---

## Trước đây chúng ta thường gặp khó khăn gì?

Đối với các ứng dụng cloud-native, việc sử dụng API của Amazon S3 không phải là vấn đề lớn.

Tuy nhiên, với các:

- Ứng dụng truyền thống (Legacy Applications)
- Công cụ xử lý dữ liệu
- Thư viện AI/ML
- Phần mềm Linux

...chúng thường chỉ được thiết kế để làm việc với file system.

Để giải quyết, nhiều doanh nghiệp phải triển khai thêm:

- Amazon EFS
- Amazon FSx

Sau đó đồng bộ dữ liệu giữa File Storage và Amazon S3.

Điều này khiến:

- Chi phí tăng lên
- Kiến trúc hệ thống phức tạp hơn
- Khó mở rộng và quản lý

Amazon S3 Files được tạo ra nhằm giảm bớt những hạn chế đó.

---

# Những lợi ích nổi bật

## 1. Đơn giản hóa việc phát triển ứng dụng

Thay vì phải gọi API để tải lên hoặc tải xuống dữ liệu, ứng dụng có thể thao tác trực tiếp với các tệp bằng những thao tác quen thuộc.

Điều này giúp:

- Giảm lượng mã nguồn cần viết
- Dễ bảo trì
- Rút ngắn thời gian phát triển

---

## 2. Tận dụng chi phí thấp của Amazon S3

Dữ liệu vẫn được lưu trực tiếp trong Amazon S3 nên người dùng vẫn được hưởng:

- Chi phí lưu trữ thấp
- Khả năng mở rộng gần như không giới hạn
- Độ bền dữ liệu rất cao

Trong nhiều trường hợp, doanh nghiệp có thể giảm nhu cầu triển khai thêm các hệ thống lưu trữ dạng file.

---

## 3. Khả năng mở rộng gần như không giới hạn

Amazon S3 được thiết kế để lưu trữ khối lượng dữ liệu rất lớn.

Amazon S3 Files kế thừa ưu điểm này, giúp các ứng dụng xử lý lượng dữ liệu lớn mà không cần lo lắng về việc mở rộng dung lượng lưu trữ.

---

## 4. Dễ dàng tích hợp với hệ sinh thái AWS

Amazon S3 Files có thể kết hợp với nhiều dịch vụ AWS như:

- Amazon EC2
- Amazon ECS
- Amazon EKS
- Các dịch vụ AI/ML
- Phân tích dữ liệu

Nhờ đó nhiều ứng dụng có thể cùng truy cập một nguồn dữ liệu mà không cần tạo thêm nhiều bản sao.

---

## Những trường hợp sử dụng tiêu biểu

Amazon S3 Files đặc biệt phù hợp với:

- Huấn luyện mô hình AI và Machine Learning
- Xây dựng Data Lake
- Phân tích dữ liệu quy mô lớn
- Xử lý hình ảnh và video
- Lưu trữ tài liệu số
- Chạy ứng dụng trên Kubernetes với Amazon EKS
- Di chuyển các ứng dụng truyền thống lên AWS mà không cần thay đổi quá nhiều mã nguồn

---

## Có điều gì cần lưu ý?

Mặc dù Amazon S3 Files mang lại trải nghiệm tương tự một file system, **Amazon S3 vẫn là Object Storage**.

Điều này có nghĩa là một số hành vi hoặc tính năng đặc thù của file system truyền thống có thể hoạt động khác hoặc chưa được hỗ trợ đầy đủ.

Ngoài ra, đây vẫn là một tính năng mới của AWS. Trước khi triển khai cho môi trường production, người dùng nên đọc kỹ tài liệu chính thức và thử nghiệm với các ứng dụng thực tế.

---

## Kết luận

Amazon S3 Files là một bước tiến đáng chú ý của AWS trong việc thu hẹp khoảng cách giữa **Object Storage** và **File Storage**.

Nhờ lớp truy cập theo mô hình file system, các ứng dụng có thể làm việc với dữ liệu trên Amazon S3 theo cách tự nhiên hơn mà vẫn tận dụng được khả năng mở rộng, độ bền và chi phí tối ưu của S3.

Đối với các doanh nghiệp đang xây dựng hệ thống AI, Machine Learning, Data Analytics hoặc xử lý lượng lớn tệp trên AWS, Amazon S3 Files hứa hẹn sẽ giúp đơn giản hóa kiến trúc, giảm chi phí và rút ngắn thời gian phát triển.

Nếu bạn đang học AWS hoặc tìm hiểu về các dịch vụ lưu trữ trên nền tảng này, Amazon S3 Files là một tính năng rất đáng để khám phá.

---

## Tài liệu tham khảo

- https://aws.amazon.com/blogs/aws/launching-s3-files-making-s3-buckets-accessible-as-file-systems/