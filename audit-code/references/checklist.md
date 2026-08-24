# Checklist review bảo mật source code

## 1. Input validation & Injection
- SQL injection: query build bằng string concat/format thay vì parameterized query hoặc ORM an toàn.
- NoSQL injection:
  - Operator injection: input user (thường từ JSON body) được đưa thẳng vào query object mà không ép kiểu/whitelist field, cho phép lọt operator như `$where`, `$ne`, `$gt`, `$regex`, `$or` (vd MongoDB nhận `{"username": {"$ne": null}}` thay vì string do parser tự parse JSON thành object).
  - `$where`/`mapReduce`/`$function`/`$accumulator` dùng JS expression build từ string concat với input user — tương đương command injection trong ngữ cảnh NoSQL, cho phép thực thi JS tuỳ ý phía DB.
  - Query bằng driver dạng "raw"/string (vd `db.eval`, query string tự ghép) thay vì query builder có kiểu (typed) của driver.
  - Thiếu whitelist field/key cho phép build query động: object injection qua key lồng nhau (vd `req.body` đưa thẳng vào `$set`/`$unset`/filter mà không lọc field nhạy cảm như `role`, `isAdmin`).
  - Input dùng để build tên collection/field động (dynamic collection/field name) mà không validate, cho phép truy cập ngoài phạm vi dự kiến.
  - ORM/ODM (Mongoose, v.v.) tắt schema strict mode hoặc dùng `.lean()`/cast tuỳ ý khiến operator injection lọt qua tầng validate của model.
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

## 8. Prompt Injection (LLM/AI integration)
- Prompt injection trực tiếp: input user được nối thẳng vào system/instruction prompt mà không có ranh giới rõ ràng (delimiter, cấu trúc message riêng) giữa instruction và data.
- Prompt injection gián tiếp: nội dung từ nguồn không tin cậy (web page, file upload, kết quả tool/API, email, tài liệu bên thứ ba) được đưa vào ngữ cảnh của LLM và có thể chứa chỉ thị giả mạo.
- LLM có quyền gọi tool/function (đọc file, gọi API, exec code, truy vấn DB) mà không có lớp kiểm soát/allowlist độc lập với chính LLM — để LLM tự quyết định hành động nhạy cảm dựa trên nội dung không tin cậy.
- Thiếu tách bạch trust boundary: output của LLM được tin dùng trực tiếp làm input cho hành động có side-effect (query DB, gọi API, ghi file, gửi request) mà không validate/sanitize lại như với input từ user.
- System prompt/hướng dẫn bảo mật bị lộ hoặc có thể bị trích xuất qua input crafted (system prompt leakage), làm lộ logic guardrail.
- Thiếu giới hạn phạm vi ngữ cảnh: toàn bộ conversation history/RAG context được đưa vào không lọc, cho phép injection từ lượt trước ảnh hưởng lượt sau.
- Không có cơ chế phát hiện/giảm thiểu (output filtering, permission theo tool, human-in-the-loop cho hành động nhạy cảm) khi LLM xử lý nội dung từ nguồn ngoài.

## Nguyên tắc chung khi áp dụng checklist
- Chỉ báo cáo phát hiện có bằng chứng cụ thể trong code (dòng, biến, luồng dữ liệu) — không liệt kê rủi ro lý thuyết không khớp với code thực tế.
- Ưu tiên trace luồng input → xử lý → sink để xác nhận khả năng khai thác trước khi xếp severity.
