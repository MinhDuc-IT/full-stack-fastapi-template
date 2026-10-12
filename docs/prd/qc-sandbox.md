# PRD nháp: QC Sandbox FastAPI

Trạng thái: DRAFT — cần MinhDuc-IT duyệt yêu cầu trước khi sinh Ground-Truth.
Phạm vi: ứng dụng sandbox; DB và tài khoản đều dành riêng cho CI.
Không có lượt LLM thật nào được cho phép trong giai đoạn chuẩn bị này.

## US-1 — Kiểm tra sẵn sàng

- AC-1.1: GET /api/v1/utils/health-check/ trả HTTP 200 và JSON true khi API sẵn sàng.
- AC-1.2: GET /api/v1/openapi.json trả OpenAPI hợp lệ mô tả API đang chạy.

## US-2 — Đăng nhập

- AC-2.1: Tài khoản active nhập email/mật khẩu đúng nhận access_token; token dùng được để truy cập thông tin người dùng.
- AC-2.2: Mật khẩu sai bị từ chối, không phát access_token.
- AC-2.3: Request tới /api/v1/users/me không có token bị từ chối.
- AC-2.4: Token không hợp lệ bị từ chối.

Đăng nhập dùng POST /api/v1/login/access-token với application/x-www-form-urlencoded, trường username và password.
Adapter auth Ground-Truth hiện chỉ hỗ trợ JSON login; cần giải quyết hỗ trợ form trước khi chạy suite GT có authenticated cases.

## US-3 — Quản lý items

- AC-3.1: Người đã đăng nhập có thể tạo item có title hợp lệ và đọc lại bằng id trả về.
- AC-3.2: Chủ item có thể cập nhật title/description và đọc lại giá trị mới.
- AC-3.3: Chủ item có thể xóa item; đọc id đã xóa trả 404.
- AC-3.4: Người dùng thường không được đọc/sửa/xóa item thuộc người dùng khác.
- AC-3.5: Title rỗng bị từ chối và không tạo item.

## Ngoài phạm vi lần đo

SMTP thực, email delivery, Jira, giao diện browser và hiệu năng production.
Kỳ vọng phân quyền phải được QA xác nhận; không suy kết quả mong đợi từ implementation của LLM.
