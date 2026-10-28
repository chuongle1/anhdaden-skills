# explore-architecture

Claude Skill (personal, `~/.claude/skills/explore-architecture/`) — khám phá và giải thích kiến trúc/tổng quan cấp cao của một repo, service hoặc module, dùng MCP server `codegraph` để dựng bản đồ cấu trúc (entry point, layering, domain model, dependency, luồng dữ liệu chính) thay vì đọc code rời rạc theo cảm tính.

## Cấu trúc
```
explore-architecture/
  SKILL.md              # frontmatter (name/description) + quy trình tổng hợp kiến trúc
  references/
    overview-checklist.md  # checklist các khía cạnh cần bao phủ (entry point, layering, domain model, dependency, luồng dữ liệu, pattern/convention)
  README.md              # file này
```

## Phạm vi & ranh giới với skill khác
- **Dùng khi**: user muốn hiểu bức tranh tổng thể của code — kiến trúc, layering, module nào phụ thuộc module nào, luồng xử lý chính cho 1 use case, hoặc onboarding vào codebase mới.
- **Không dùng khi**:
  - Mục tiêu là tìm lỗ hổng/đánh giá rủi ro bảo mật → dùng skill `audit-code` (toàn repo/module cụ thể) hoặc `security-review` (pending diff trên branch).
  - Chỉ cần review style/logic/hiệu năng → dùng skill `code-review`.
  - Chỉ cần tra cứu nhanh 1 hàm/symbol đơn lẻ ("hàm này làm gì", "định nghĩa ở đâu") — không cần invoke skill này, trả lời trực tiếp bằng `codegraph_symbol`/`codegraph_context` là đủ.

## Yêu cầu môi trường (bắt buộc trước khi skill hoạt động đúng)
- `codegraph` (codegraph-rs) **v2.x**, MCP server đăng ký bằng absolute path và có `--mermaid`:
  ```
  claude mcp add codegraph --scope user -- <abs-path>/codegraph serve --mcp --mermaid
  ```
- Kiểm tra bằng `claude mcp list`. Mở phiên Claude Code mới sau khi đăng ký hoặc nâng version.
- Cài đặt, sửa lỗi và index: xem skill `codegraph-setup`. Hạ tầng dùng chung với `audit-code`.

## Cách hoạt động (tóm tắt)
1. Xác định phạm vi (toàn repo/module/service cụ thể) và mục tiêu (overview chung hay tập trung 1 use case/luồng).
2. **Luôn dùng index mới nhất**: `codegraph_init {path: <git root>}` (bind session) → `codegraph_index {}` (re-index, không hỏi trước) → `codegraph_status`. Không `rm -rf .codegraph` (mất `config.toml`). Tool chưa sẵn sàng → skill `codegraph-setup`, rồi mới fallback thủ công.
3. Kiểm kê cấu trúc: `codegraph_graphcode_files`, `codegraph_graphcode_list_types`, `codegraph_graphcode_dependencies` (bản đồ dependency mức module); config/manifest qua `codegraph_graphdoc_ingest_dir` + `graphdoc_search`.
4. Xác định entry point bằng `codegraph_graphcode_search_by_annotation` (route/decorator/handler).
5. Đào sâu domain model/thành phần cốt lõi bằng `codegraph_graphcode_class`, tra cứu nhanh bằng `codegraph_symbol`/`codegraph_context`.
6. Trace 1-2 luồng chính end-to-end bằng `codegraph_callers`/`codegraph_callees` và `codegraph_flow`/`codegraph_search_flow` (call site kèm effect `sql_query`/`http_call`…), `codegraph_trace_full_flow` cho "mọi đường tới X", ghi lại hop kèm `file:line`.
7. Với câu hỏi dạng "sửa X ảnh hưởng gì", dùng `codegraph_impact`.
8. Tổng hợp theo khung ở `references/overview-checklist.md`, chỉ giữ lại điều có bằng chứng cụ thể.
9. Trả lời trực tiếp trong chat theo cấu trúc chuẩn (tổng quan, entry point, layering, domain model, luồng dữ liệu, dependency ngoài, pattern đáng chú ý), kèm sơ đồ Mermaid tuỳ chọn khi hữu ích. Không tự ý ghi ra file — chỉ lưu (vd `ARCHITECTURE.md`) khi user yêu cầu.

## Việc cần tuỳ chỉnh thêm
`references/overview-checklist.md` hiện là bộ khung chung — có thể bổ sung thêm khía cạnh đặc thù (vd yêu cầu compliance, chuẩn kiến trúc nội bộ của team) nếu cần dùng skill này cho mục đích viết tài liệu chính thức thay vì chỉ explore nhanh.
