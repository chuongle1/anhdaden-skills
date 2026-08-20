# Checklist review bảo mật source code

## 1. Input validation & Injection
- SQL/NoSQL injection: query build bằng string concat/format thay vì parameterized query hoặc ORM an toàn.
- Command injection: input người dùng đi thẳng vào shell exec, subprocess, os.system.
- Path traversal: input dùng để build file path không được sanitize/normalize.
- XSS: output ra HTML/JS mà không escape, đặc biệt render trực tiếp input user.
- SSRF: input user quyết định URL/host mà server gọi tới.
- Deserialization không an toàn: pickle/yaml.load/unserialize trên input không tin cậy.

## 2. Authentication & Authorization
- Thiếu kiểm tra quyền (authz) ở endpoint/hàm xử lý dữ liệu nhạy cảm — chỉ dựa vào authN mà quên authZ.
- IDOR: truy cập resource qua ID mà không verify ownership/quyền.
- Session token: không đủ entropy, không có expiry, không invalidate khi logout.
- So sánh secret/token dùng `==` thay vì so sánh constant-time.

## 3. Secrets & Configuration
- Secret/API key/credential hardcode trong code hoặc log ra output.
- Config mặc định không an toàn (debug=True, CORS mở toàn bộ, verify SSL tắt).
- Secret bị commit vào repo hoặc bị expose qua error message/stack trace.

## 4. Cryptography
- Dùng thuật toán yếu (MD5/SHA1 cho password, DES, ECB mode).
- Tự chế cơ chế mã hoá/ký thay vì dùng thư viện đã kiểm chứng.
- Random không đủ an toàn (dùng `random` thay vì CSPRNG) cho token/session/nonce.

## 5. Dependency & Supply chain
- Dependency có lỗ hổng đã biết (kiểm tra version trong lockfile nếu có thể).
- Cài package từ nguồn không đáng tin, hoặc dùng version pin lỏng lẻo (`*`, không lock).

## 6. Error handling & Logging
- Log dữ liệu nhạy cảm (password, token, PII) ra log thường.
- Error message trả về client lộ stack trace/thông tin hệ thống.
- Exception bị nuốt (bare except/catch) che giấu lỗi bảo mật.

## 7. Business logic abuse
- Race condition ở thao tác nhạy cảm (thanh toán, đổi quyền, rate limit).
- Thiếu rate limit/throttle ở endpoint dễ bị brute-force hoặc abuse.
- Giả định sai về trust boundary (tin dữ liệu từ client mà lẽ ra phải validate lại ở server).

## Nguyên tắc chung khi áp dụng checklist
- Chỉ báo cáo phát hiện có bằng chứng cụ thể trong code (dòng, biến, luồng dữ liệu) — không liệt kê rủi ro lý thuyết không khớp với code thực tế.
- Ưu tiên trace luồng input → xử lý → sink để xác nhận khả năng khai thác trước khi xếp severity.
