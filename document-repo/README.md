# document-repo

Claude Skill (personal, `~/.claude/skills/document-repo/`) — sinh tài liệu giới thiệu repo/project hiện tại thành 2 file markdown cố định ở root (`ARCHITECTURE.md` cho dân kỹ thuật, `OVERVIEW.md` cho dân non-tech), dùng MCP server `codegraph` để khảo sát cấu trúc thực tế thay vì suy diễn.

## Cấu trúc
```
document-repo/
  SKILL.md              # frontmatter (name/description) + quy trình sinh tài liệu
  references/
    doc-checklist.md     # khung nội dung chi tiết cho cả 2 file + tiêu chí "grounded in code"
  README.md              # file này
```

## Phạm vi & ranh giới với skill khác
- **Dùng khi**: user muốn một tài liệu giới thiệu repo **được ghi ra file**, phục vụ cả người đọc kỹ thuật lẫn không kỹ thuật.
- **Không dùng khi**:
  - Chỉ cần hỏi nhanh kiến trúc, trả lời trong chat là đủ, không cần lưu file → dùng skill `explore-architecture` (skill đó mặc định không ghi file).
  - Mục tiêu là tìm lỗ hổng/đánh giá rủi ro bảo mật → dùng skill `audit-code` (toàn repo/module cụ thể) hoặc `security-review` (pending diff).

## Khác biệt với `explore-architecture`
| | `explore-architecture` | `document-repo` |
|---|---|---|
| Output | Trả lời trong chat, không ghi file trừ khi user yêu cầu | Luôn ghi 2 file cố định (`ARCHITECTURE.md`, `OVERVIEW.md`) |
| Đối tượng đọc | 1 (người đang trò chuyện với Claude) | 2 (dân tech + dân non-tech), giọng văn khác nhau |
| Phạm vi nội dung | Kiến trúc, layering, domain model, dependency, luồng dữ liệu | Giống `explore-architecture` + danh sách chức năng cấp người dùng diễn giải phi kỹ thuật |

Hai skill dùng chung phương pháp khảo sát bằng `codegraph` (entry point, layering, domain model, luồng dữ liệu) — `document-repo` tái sử dụng đúng bộ bước đó rồi tổng hợp thêm thành 2 tài liệu lưu trữ được.

## Yêu cầu môi trường (bắt buộc trước khi skill hoạt động đúng)
1. Cài đặt tool `codegraph` (codegraph-rs) và có sẵn trong `PATH`.
2. Đăng ký `codegraph serve` làm MCP server (đã làm ở user scope):
   ```
   claude mcp add codegraph --scope user -- codegraph serve --mcp
   ```
   Kiểm tra bằng `claude mcp list` — cần thấy `codegraph ... ✔ Connected`.
3. **Mở phiên Claude Code mới** sau khi đăng ký MCP server — tool `codegraph_*` chỉ xuất hiện ở phiên khởi động sau khi server đã được đăng ký.

(Skill này dùng chung hạ tầng `codegraph` với `audit-code`/`explore-architecture` — nếu đã setup cho skill khác rồi thì không cần làm lại.)

## Cách hoạt động (tóm tắt)
1. Xác định phạm vi (toàn repo mặc định, hoặc module/service cụ thể nếu user chỉ định).
2. **Luôn xoá và dựng lại `.codegraph/` mới nhất trước khi khảo sát** (`rm -rf .codegraph && codegraph init` nếu đã tồn tại, không hỏi trước), rồi `codegraph_status` xác nhận sẵn sàng. Fallback sang skill `codegraph-setup` nếu tool `codegraph_*` chưa sẵn sàng, rồi mới tới khảo sát thủ công nếu vẫn không được.
3. Khảo sát kỹ thuật: `codegraph_files`, `codegraph_list_classes`/`codegraph_list_interfaces`, `codegraph_dependencies` (cấu trúc & layering); `codegraph_search_by_annotation` (entry point); `codegraph_class`/`codegraph_class_methods`/`codegraph_symbol`/`codegraph_context` (domain model); `codegraph_callers`/`codegraph_callees`/`codegraph_flow`/`codegraph_search_flow` (trace luồng chính, ghi hop kèm `file:line`).
4. Suy ra danh sách chức năng cấp người dùng từ mỗi entry point đã xác định — mỗi chức năng phải trỏ về `file:line` cụ thể, không suy đoán chức năng không có code hậu thuẫn.
5. Tổng hợp theo khung ở `references/doc-checklist.md`, áp dụng nguyên tắc chống suy diễn: thiếu bằng chứng → ghi "không xác định được từ code"/"chưa rõ", không bịa thêm.
6. Ghi đè toàn bộ `ARCHITECTURE.md` và `OVERVIEW.md` ở root repo (đây là snapshot tài liệu tại thời điểm chạy, không phải log tích luỹ) — đọc file cũ trước nếu đã tồn tại để biết đang ghi đè gì, không cần hỏi xác nhận mỗi lần.
7. Báo cáo ngắn gọn trong chat: đã ghi file nào, tóm tắt 1-2 dòng mỗi file — không dán lại toàn bộ nội dung vào chat.

## Việc cần tuỳ chỉnh thêm
`references/doc-checklist.md` hiện là khung chung — có thể bổ sung thêm khía cạnh đặc thù (vd template compliance/stakeholder report riêng của team) nếu cần dùng skill này cho mục đích chính thức hơn là tài liệu onboarding nhanh.
