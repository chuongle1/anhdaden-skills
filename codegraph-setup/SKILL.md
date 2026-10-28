---
name: codegraph-setup
description: Kiểm tra, cài/cập nhật binary `codegraph` (codegraph-rs v2.x), đăng ký MCP server `codegraph` cho Claude Code, và bind + index repo hiện tại qua MCP (`codegraph_init`/`codegraph_index`). Dùng khi user yêu cầu "setup codegraph", "kết nối codegraph", "khởi tạo/index codegraph cho repo này", "update codegraph", "connect codegraph MCP", hoặc khi một skill khác (vd audit-code, explore-architecture, document-repo) báo tool codegraph_* không khả dụng / session chưa bind / index cũ. Không dùng để review/phân tích code — skill này chỉ lo hạ tầng.
---

## Khi nào dùng
- User yêu cầu trực tiếp: setup / kết nối / cập nhật / index codegraph cho repo.
- Skill khác không dùng được `codegraph_*` (tool không có, session chưa bind, index rỗng/cũ).

Skill có 2 giai đoạn độc lập — chỉ làm giai đoạn còn thiếu:
- **A. Kết nối (mỗi máy 1 lần)**: binary + đăng ký MCP. Cần mở phiên Claude Code mới sau khi sửa.
- **B. Chuẩn bị repo (mỗi repo/phiên)**: bind session MCP vào repo + index. Làm ngay trong phiên hiện tại qua MCP tool, không cần phiên mới.

## ⚠️ PATH theo phiên
MCP server do **tiến trình Claude Code chính** spawn với PATH lúc phiên khởi động. `source ~/.zshrc` trong Bash tool chỉ đổi PATH của shell con đó → không sửa được MCP, không dùng để "verify rồi coi như xong". Luôn đăng ký MCP bằng **absolute path** của binary.

## Validate path binary (chống binary giả mạo / command injection)
Path lấy từ `which`/`command -v`/rc file là dữ liệu môi trường, chưa đáng tin:
- Không chứa ký tự shell đặc biệt (`;`, `|`, `` ` ``, `$(`, `&&`, khoảng trắng chưa quote) — có thì dừng và báo user, không tự "làm sạch".
- Phải là file thực thi tồn tại (`test -x <path>`).
- Thư mục tin cậy: `~/.cargo/bin` (đích của `cargo install`), `/opt/homebrew/bin`, `/usr/local/bin`, `~/.local/bin`, `<repo codegraph-rs>/target/release`, hoặc path đang được dùng bởi entry MCP `codegraph` đã `✔ Connected` (user đã chấp nhận trước đó). Ngoài các thư mục này → hỏi user trước khi đăng ký.

## Giai đoạn A — Kết nối
1. **Tìm binary + version**: `command -v codegraph` → validate path (mục trên) → `codegraph --version` và `codegraph doctor` (chẩn đoán OS, version, workspace, tool ngoài như git/tar).
   - Không có binary → báo user, không tự tải từ nguồn lạ. Nếu biết vị trí source `codegraph-rs` trên máy, gợi ý: `cargo install --path <codegraph-rs>/crates/codegraph --locked` (thêm `--features fastembed` nếu cần semantic search).
   - Version < 2.0 → báo là bản cũ (tên tool khác, thiếu `trace_full_flow`, `graphcode_*`…), gợi ý cài lại như trên.
2. **Kiểm tra MCP**: `claude mcp list` → entry `codegraph`.
   - `✔ Connected` và trỏ đúng binary vừa tìm → xong giai đoạn A.
   - Trỏ binary khác (vd bản cũ ở `target/release` trong khi bản mới ở `~/.cargo/bin`), ENOENT, hoặc lỗi khác → hỏi user trước khi `claude mcp remove codegraph`, rồi add lại.
   - Chưa có → add.
   - Lệnh add chuẩn: `claude mcp add codegraph --scope user -- <absolute-path>/codegraph serve --mcp --mermaid` (`--mermaid` bật `codegraph_mermaid` cho skill explore/document).
   - Không dùng `codegraph install --agent claude` để đăng ký: lệnh này ghi `mcpServers` vào `~/.claude/settings.json`, không phải nơi Claude Code CLI đọc cấu hình MCP (`claude mcp add` ghi vào `~/.claude.json`). Nó cũng cài kèm bộ skill `codegraph-*` của upstream có thể chồng phạm vi với `audit-code`/`explore-architecture` — chỉ dùng nếu user muốn rõ ràng.
3. Sau khi add/sửa entry: nhắc user **mở phiên Claude Code mới**; phiên hiện tại không thấy tool mới, không gọi thử rồi kết luận thất bại.

## Giai đoạn B — Chuẩn bị repo (qua MCP)
1. **Xác định root**: `git rev-parse --show-toplevel`. Không phải git repo → hỏi user root nào, không index cwd mù quáng (dễ index nhầm thư mục cha chứa nhiều repo).
2. **Bind**: `codegraph_status` — nếu session chưa bind hoặc bind sai root → `codegraph_init {"path": "<root tuyệt đối>"}` (tạo `.codegraph/` + `config.toml` + `.gitignore`, không index).
3. **Index**: dựa vào `codegraph_status` (số file/symbol) và độ mới:
   - Index rỗng → `codegraph_index {}`.
   - Có index → so mtime `.codegraph/db.sqlite` với commit mới nhất (`git log -1 --format=%ct`) và `git status --porcelain`; cũ hơn hoặc working tree bẩn → báo index có thể lệch, đề xuất `codegraph_index {}`. Skill gọi từ `audit-code`/`explore-architecture`/`document-repo` thì re-index luôn (các skill đó yêu cầu index mới).
   - Không xoá `.codegraph/` bằng `rm -rf`: sẽ mất `config.toml` (gồm `[[effect_rules]]` tuỳ chỉnh). Cần dọn sạch → `codegraph clean` (giữ config) rồi `codegraph_index {}`.
4. **Git ignore**: `codegraph_init` tự tạo `.codegraph/.gitignore`; vẫn kiểm `git check-ignore -q .codegraph/db.sqlite`. Nếu chưa bị ignore → đề xuất thêm `.codegraph` vào `.gitignore` (hỏi trước khi sửa) để không commit index chứa toàn bộ code.
5. Fallback CLI (khi MCP chưa dùng được trong phiên này): `codegraph init --path <root> --progress` — index sẵn trên đĩa để phiên mới dùng ngay.

## Báo cáo
Ngắn gọn: path + version binary, trạng thái MCP (nếu vừa add/sửa → chỉ xác nhận được ở phiên mới), root đã bind, index (vừa tạo / re-index / giữ nguyên + số file/symbol từ `codegraph_status`), và cảnh báo `.gitignore` nếu có.
