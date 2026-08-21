# review-code

Claude Skill (personal, `~/.claude/skills/review-code/`) — review bảo mật source code theo yêu cầu cụ thể của user hoặc quét toàn bộ repo/codebase, dùng checklist riêng của security team và MCP server `codegraph` để trace cấu trúc/luồng gọi thay vì đoán.

## Cấu trúc
```
review-code/
  SKILL.md              # frontmatter (name/description) + quy trình review
  references/
    checklist.md         # checklist chi tiết theo từng nhóm rủi ro (injection, authN/authZ, secrets, crypto, dependency, logging, business logic abuse)
  README.md              # file này
```

## Phạm vi & ranh giới với skill khác
- **Dùng khi**: user yêu cầu phân tích/review bảo mật một đoạn code, file, module cụ thể — hoặc audit/quét bảo mật toàn bộ repo hiện tại.
- **Không dùng khi**:
  - Chỉ cần review style/logic/hiệu năng không liên quan bảo mật → dùng skill `code-review`.
  - Review toàn bộ pending diff trên branch hiện tại theo quy trình chuẩn → dùng skill `security-review`.

## Yêu cầu môi trường (bắt buộc trước khi skill hoạt động đúng)
1. Cài đặt tool `codegraph` (codegraph-rs) và có sẵn trong `PATH`.
2. Đăng ký `codegraph serve` làm MCP server (đã làm ở user scope):
   ```
   claude mcp add codegraph --scope user -- codegraph serve --mcp
   ```
   Kiểm tra bằng `claude mcp list` — cần thấy `codegraph ... ✔ Connected`.
3. **Mở phiên Claude Code mới** sau khi đăng ký MCP server — tool `codegraph_*` chỉ xuất hiện ở phiên khởi động sau khi server đã được đăng ký, không xuất hiện ngay trong phiên đang chạy lúc đăng ký.

## Cách hoạt động (tóm tắt)
1. Xác định phạm vi review (đoạn/file/module cụ thể, hay toàn bộ repo).
2. Chạy `codegraph init` tại root repo để dựng index (`.codegraph/`, dạng SQLite nội bộ — không đọc trực tiếp).
3. Dùng các MCP tool do `codegraph serve` cung cấp để nắm cấu trúc & trace luồng gọi:
   `codegraph_status`, `codegraph_files`, `codegraph_search_by_annotation` (tìm entry point), `codegraph_search_by_call` (tìm sink nguy hiểm), `codegraph_callers`/`codegraph_callees`, `codegraph_flow`/`codegraph_search_flow` (trace input→sink), `codegraph_symbol`/`codegraph_references`/`codegraph_context`, `codegraph_impact`.
4. Áp dụng checklist ở `references/checklist.md`, chỉ báo cáo phát hiện có bằng chứng cụ thể trong code.
5. Trace khả năng khai thác bằng input → sink qua codegraph.
6. **Verify tĩnh** — chủ động tìm lý do bác bỏ (lớp validation/sanitize bị bỏ sót, input có thực sự do bên ngoài kiểm soát không, verify độc lập từng finding cùng pattern) trước khi giữ lại.
7. **Xác minh bằng unit test khi khả thi** — viết test tạm nhắm vào luồng nghi vấn để lấy bằng chứng thực nghiệm; pass → tăng độ tin cậy, không tái hiện được → hạ độ tin cậy/loại bỏ. Không commit test tạm vào repo user.
8. **Tạo PoC phi phá hoại** cho finding đã xác nhận (test pass hoặc verify tĩnh đủ chắc), để user tự chạy kiểm chứng thêm — không thực thi hành động phá hoại thật, chỉ minh hoạ khả năng khai thác trong phạm vi được cấp quyền review.
9. Xếp severity (Critical/High/Medium/Low, ưu tiên nâng cho finding có evidence từ test/PoC).
10. **Ghi kết quả vào `SECURITY_FINDING.md` ở root repo** (đọc file cũ và cập nhật nếu đã tồn tại, không ghi đè toàn bộ) — bảng kết quả + PoC/test nằm trong file này, **không in chi tiết ra chat**; trong chat chỉ báo đường dẫn file và tổng số finding theo severity.

Nếu `codegraph init` hoặc các tool `codegraph_*` không khả dụng, skill sẽ gợi ý chạy skill `codegraph-setup` (kiểm tra/kết nối lại MCP + index) trước; nếu vẫn không được mới fallback sang khảo sát thủ công (Grep/Explore) — không im lặng bỏ qua.

## Việc cần tuỳ chỉnh thêm
`references/checklist.md` hiện là bộ mặc định dạng OWASP-style — nên chỉnh lại cho khớp rubric/severity thực tế của secteam trước khi dùng chính thức.
