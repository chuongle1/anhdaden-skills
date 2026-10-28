# audit-code

Claude Skill (personal, `~/.claude/skills/audit-code/`) — review bảo mật source code theo yêu cầu cụ thể của user hoặc quét toàn bộ repo/codebase, dùng checklist riêng của security team và MCP server `codegraph` để trace cấu trúc/luồng gọi thay vì đoán.

## Cấu trúc
```
audit-code/
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
1. `codegraph` (codegraph-rs) **v2.x**, ví dụ cài bằng `cargo install --path <codegraph-rs>/crates/codegraph --locked`.
2. MCP server `codegraph` đăng ký bằng absolute path, ở user scope:
   ```
   claude mcp add codegraph --scope user -- <abs-path>/codegraph serve --mcp --mermaid
   ```
   Kiểm tra bằng `claude mcp list`, cần thấy `codegraph ... ✔ Connected`. Chi tiết và cách sửa lỗi: xem skill `codegraph-setup`.
3. **Mở phiên Claude Code mới** sau khi đăng ký hoặc nâng version MCP server.

## Cách hoạt động (tóm tắt)
1. Xác định phạm vi review (đoạn/file/module cụ thể, hay toàn bộ repo).
2. **Luôn dùng index mới nhất:** `codegraph_init {path: <git root>}` để bind session, rồi `codegraph_index {}` để re-index. Không `rm -rf .codegraph`, vì sẽ mất `config.toml` (effect rules).
3. **Khai báo sink** của repo bằng `codegraph_config_effects` (`[[effect_rules]]`: wrapper SQL/exec/HTTP/file nội bộ → `sql_query`, `http_call`, `file_write`…), rồi re-index.
   Tool dùng trong các bước tiếp theo (tên theo v2.x):
   - Cấu trúc: `codegraph_graphcode_files` / `list_types` / `class` / `dependencies`
   - Entry point: `codegraph_graphcode_search_by_annotation`, `codegraph_search_symbol` (có `semantic`/`hybrid` nếu bật embedding)
   - Sink: `codegraph_references` (thay `search_by_call` của v1, gồm cả external call chưa resolve), `codegraph_search_flow`
   - Trace: `codegraph_trace_full_flow` (mọi đường caller→sink), `codegraph_flow` (guard condition + effect tại call site)
   - Ngữ cảnh: `codegraph_symbol` / `context` / `graphcode_function_scope` / `impact`
   - Bằng chứng hành vi: `codegraph_graphcode_sandbox`
   - Config/IaC: `codegraph_graphdoc_ingest_dir` + `graphdoc_search` / `search_value`
4. Áp dụng checklist ở `references/checklist.md`, chỉ báo cáo phát hiện có bằng chứng cụ thể trong code.
5. Trace khả năng khai thác bằng `codegraph_trace_full_flow` (input → sink), ghi lại chuỗi hop; hop qua interface được fan-out ra mọi implementation (entry point → hàm trung gian → sink, kèm `file:line`) — dữ liệu này bắt buộc dùng để dựng cột Attack Flow trong báo cáo.
6. **Verify tĩnh** — chủ động tìm lý do bác bỏ (lớp validation/sanitize bị bỏ sót, input có thực sự do bên ngoài kiểm soát không, verify độc lập từng finding cùng pattern) trước khi giữ lại; với guard rẽ nhánh có thể chạy `codegraph_graphcode_sandbox` ép cả 2 nhánh làm bằng chứng bổ trợ.
7. **Xác minh bằng unit test khi khả thi** — viết test tạm nhắm vào luồng nghi vấn để lấy bằng chứng thực nghiệm; pass → tăng độ tin cậy, không tái hiện được → hạ độ tin cậy/loại bỏ. Không commit test tạm vào repo user.
8. **Tạo PoC phi phá hoại** cho finding đã xác nhận (test pass hoặc verify tĩnh đủ chắc), để user tự chạy kiểm chứng thêm — không thực thi hành động phá hoại thật, chỉ minh hoạ khả năng khai thác trong phạm vi được cấp quyền review.
9. Xếp severity (Critical/High/Medium/Low, ưu tiên nâng cho finding có evidence từ test/PoC).
10. **Ghi kết quả vào `SECURITY_FINDING.md` ở root repo** (đọc file cũ và cập nhật nếu đã tồn tại, không ghi đè toàn bộ) — bảng kết quả + PoC/test nằm trong file này, **mỗi finding phải có cột Attack Flow dựng từ chuỗi hop codegraph** (entry point → hàm trung gian → sink), **không in chi tiết ra chat**; trong chat chỉ báo đường dẫn file và tổng số finding theo severity.

Nếu `codegraph_init`/`codegraph_index` lỗi hoặc các tool `codegraph_*` không khả dụng / là bản v1, skill sẽ gợi ý chạy skill `codegraph-setup` (kiểm tra/kết nối lại MCP + index) trước; nếu vẫn không được mới fallback sang khảo sát thủ công (Grep/Explore) — không im lặng bỏ qua.

## Việc cần tuỳ chỉnh thêm
`references/checklist.md` hiện là bộ mặc định dạng OWASP-style — nên chỉnh lại cho khớp rubric/severity thực tế của secteam trước khi dùng chính thức.
