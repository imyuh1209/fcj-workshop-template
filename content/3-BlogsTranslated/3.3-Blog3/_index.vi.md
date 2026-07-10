---
title: "AWS Security Agent: Một agent, phủ kín cả vòng đời phát triển application"
date: 2026-06-15
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

Một trong những vấn đề muôn thuở của security là nó luôn bị tách rời khỏi quy trình dev: designer thiết kế xong, dev code xong, rồi mới đến lượt security team ngồi soi lại — thường là khi đã quá muộn để sửa rẻ. AWS Security Agent (nằm trong AWS Continuum) đang cố gắng xóa bỏ sự tách rời đó, bằng cách phủ security xuyên suốt ba giai đoạn: lúc thiết kế, lúc code, và lúc deploy — tất cả trong một agentic service duy nhất.

Đợt cập nhật mới nhất (giữa tháng 6/2026) bổ sung thêm threat modeling, mở rộng code review sang nhiều Git platform, và cho phép chạy mọi thứ ngay trong IDE. Dưới đây là những gì đáng chú ý.

## Threat modeling: đưa security vào từ lúc còn trên giấy

Đây là tính năng mới và có lẽ đáng chú ý nhất. Thay vì chờ code chạy xong mới quét lỗ hổng, Security Agent đọc trực tiếp design document hoặc source code, tự dựng lại data flow, architecture và trust boundary của application — rồi từ đó xác định threat actor, attack vector có thể xảy ra, và ưu tiên threat nào cần xử lý trước theo framework STRIDE (6 nhóm: giả danh identity, tampering dữ liệu, mất khả năng truy vết, lộ thông tin, denial of service, và leo thang quyền hạn).

Điểm hay là nó hoạt động được ngay từ giai đoạn design — tức là lúc chi phí sửa một lỗ hổng vẫn còn rẻ, chưa cần refactor code đã ship.

## Code review: mở rộng nền tảng, đào sâu hơn pattern-matching

Trước đây Security Agent chỉ scan được trên GitHub. Giờ đã thêm GitLab và Bitbucket (cả bản SaaS và self-hosted), cùng khả năng kéo documentation từ Confluence vào làm context cho review.

Cách nó review cũng không đơn giản là match theo pattern lỗi đã biết như linter truyền thống — mà dùng reasoning để tìm ra vulnerability phức tạp hơn, đối chiếu với security requirement riêng của tổ chức, và quan trọng là validate lại trong môi trường simulation để xác nhận lỗ hổng đó thực sự khai thác được, tránh báo false positive tràn lan làm dev mất thời gian check nhầm.

Ngoài ra, design review cũng được nâng cấp với các compliance pack có sẵn (AWS Well-Architected Framework, NIST CSF, PCI DSS...) hoặc import security requirement riêng từ document nội bộ. Mọi finding đều map ngược lại được vào compliance posture, giúp team luôn ở trạng thái sẵn sàng cho audit.

## Chạy thẳng trong IDE, không cần đổi tab

Phần này giải quyết một pain point rất thực tế: trước đây muốn xem finding hay threat model, dev phải rời IDE, mở console riêng. Giờ với Kiro power, Claude Code plugin (tên chính thức: AWS Agents for DevSecOps), và MCP integration mở cho bất kỳ AI IDE nào, mọi thứ chạy ngay trong editor bằng prompt tự nhiên, ví dụ:

- `"Run a full security scan on this repo"` → scan toàn bộ repo
- `"Build a threat model for this application"` → threat model lưu vào file `.security-agent/threat_model.md` trong workspace
- `"help me remediate my findings"` → agent pull finding về, ưu tiên finding nghiêm trọng nhất, và mở luôn bugfix session để sửa

Cách làm này khá hợp lý: security agent không cố thay thế dev, mà cố gắng "gặp" dev ngay tại chỗ dev đang làm việc.

## Một vài lưu ý khi thử

Threat modeling và code review nâng cấp hiện vẫn ở dạng Preview, nên behavior có thể còn thay đổi. Nếu muốn nghịch thử, Security Agent có free trial 2 tháng — nhưng vẫn nên canh account/credit cẩn thận để tránh phát sinh chi phí ngoài dự kiến. Một điểm cũng nên nhớ: agent đọc trực tiếp source code để build threat model, nên nếu repo có chứa secret hay credential thì nên clean trước khi cho agent scan.

## Kết Luận

Nhìn tổng thể, hướng đi của AWS Security Agent là biến security từ một bước kiểm tra rời rạc, chạy sau cùng, thành một phần liên tục nằm ngay trong workflow — từ lúc còn là design doc, qua từng pull request, đến lúc deploy. Với các bạn đang làm project có xử lý dữ liệu nhạy cảm (như upload ảnh/video người dùng), đây là một tool đáng thử để vừa bảo vệ hệ thống tốt hơn, vừa là cách thực hành thêm về tư duy threat modeling một cách bài bản.

---

## Tài liệu tham khảo
- [AWS Security Agent adds threat modeling, Kiro power, and Claude Code plugin, and more](https://aws.amazon.com/vi/blogs/aws/aws-security-agent-adds-threat-modeling-kiro-power-and-claude-code-plugin-and-more/)
