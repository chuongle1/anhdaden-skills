# codegraph-setup

Claude Skill (personal, `~/.claude/skills/codegraph-setup/`): skill hạ tầng cho [codegraph-rs](https://github.com/hungpham10/codegraph-rs) **v2.x**. Nó lo việc cài/kiểm tra binary, đăng ký MCP server cho Claude Code, rồi bind và index repo qua MCP. Skill không review hay phân tích code, mà chuẩn bị môi trường cho các skill dùng `codegraph_*` (`audit-code`, `explore-architecture`, `document-repo`).

## Hai giai đoạn
| Giai đoạn | Làm gì | Tần suất | Cần phiên mới? |
|---|---|---|---|
| A. Kết nối | Tìm và validate binary, `codegraph --version` / `doctor`, `claude mcp add ... serve --mcp --mermaid` | Mỗi máy 1 lần, hoặc khi nâng version | Có, sau khi add/sửa MCP |
| B. Chuẩn bị repo | `codegraph_status` → `codegraph_init {path}` → `codegraph_index {}`, kiểm tra index cũ và `.gitignore` | Mỗi repo / mỗi phiên | Không |

## Thay đổi so với bản cho codegraph v1
- CLI v2 có thêm `install`, `uninstall`, `doctor`, `clean`, `embed`. Server có thêm `serve --http` (MCP Streamable HTTP) và `serve --graphql` (API + web UI).
- MCP session phải được bind vào workspace (`codegraph_init {path}`) trước khi truy vấn. Index qua `codegraph_index {}`, không cần mở phiên mới.
- Không dùng `rm -rf .codegraph` để re-index nữa, vì sẽ mất `config.toml` (`[[effect_rules]]`). Thay bằng `codegraph_index {}`, hoặc `codegraph clean` nếu cần dọn sạch.
- Đăng ký MCP kèm `--mermaid` để bật `codegraph_mermaid`.
- Không dùng `codegraph install --agent claude`. Lệnh này ghi vào `~/.claude/settings.json`, không phải nơi `claude mcp` quản lý, và còn cài kèm bộ skill upstream dễ chồng phạm vi với skill của team.

## Cài binary (gợi ý, skill không tự làm)
```bash
cargo install --path <codegraph-rs>/crates/codegraph --locked --features fastembed
```
`fastembed` là tuỳ chọn: bật để có semantic search (`codegraph_search_symbol` với `match: semantic|hybrid`). Có thể tải sẵn model bằng `codegraph embed`.

## Lưu ý
- `source rc file` trong Bash tool không sửa được PATH của MCP. Luôn đăng ký bằng absolute path.
- Path binary phải được validate (ký tự shell lạ, file thực thi, thư mục tin cậy) trước khi đưa vào `claude mcp add`.
- Không tự xoá hay ghi đè cấu hình MCP, index, hay sửa `.gitignore` mà không hỏi.
