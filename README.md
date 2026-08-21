# secteam/skills

Bộ Claude Skills cho security team. Các skill chạy ở mức **personal** trên Claude Code — file thực tế nằm ở `~/.claude/skills/<ten-skill>/`.

## Skill hiện có

| Skill | Vị trí thực tế | Mô tả |
|---|---|---|
| `codegraph-setup` | `~/.claude/skills/codegraph-setup/` | Skill hạ tầng: kiểm tra, khởi tạo (`codegraph init`) và đăng ký MCP server `codegraph` (`codegraph serve --mcp`) cho repo hiện tại. Không review/phân tích code — chỉ chuẩn bị môi trường cho các skill khác cần `codegraph_*` (vd `review-code`). Xem chi tiết ở README của skill. |
| `review-code` | `~/.claude/skills/review-code/` | Review bảo mật source code theo yêu cầu cụ thể hoặc quét toàn bộ repo, dùng checklist riêng của team + MCP server [codegraph](https://github.com/hungpham10/codegraph-rs) để trace cấu trúc/luồng gọi. Xem chi tiết ở README của skill. |

## Quy ước chung khi viết skill mới
- Vị trí: `~/.claude/skills/<ten-skill>/SKILL.md` (kebab-case, tên thư mục = tên skill).
- `description` trong frontmatter là phần quan trọng nhất — Claude dùng nó để tự quyết định kích hoạt skill, nên phải nêu rõ **skill làm gì** và **khi nào trigger**, tránh chồng lấn phạm vi với skill khác (kể cả skill built-in như `code-review`, `security-review`).
- Giữ `SKILL.md` gọn — nội dung dài (checklist, bảng tra cứu) đưa vào `references/*.md`, logic xử lý dữ liệu đưa vào `scripts/`.
- Mỗi skill nên có `README.md` riêng mô tả cấu trúc, ranh giới phạm vi, và yêu cầu môi trường (nếu skill phụ thuộc tool/MCP server ngoài).
- Kiểm thử bằng cách mở phiên Claude Code mới và thử đúng tình huống trigger mong muốn; tinh chỉnh `description` nếu Claude không nhận diện đúng lúc.

## Việc cần làm tiếp theo
- Tuỳ chỉnh `references/checklist.md` của `review-code` cho khớp rubric/severity thực tế của team.
- Bổ sung các skill khác theo nhu cầu (gợi ý: `incident-report`, `ioc-lookup`, `pentest-writeup`, `threat-model`).
