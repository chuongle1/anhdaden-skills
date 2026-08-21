# codegraph-setup

Claude Skill (personal, `~/.claude/skills/codegraph-setup/`) — kiểm tra, khởi tạo và kết nối tool `codegraph` (`codegraph-rs`) làm MCP server cho Claude Code trên repo hiện tại. Đây là skill hạ tầng, dùng để chuẩn bị môi trường cho các skill khác cần `codegraph_*` (vd `review-code`), bản thân nó không review/phân tích code.

## Vì sao cần skill này
Tool `codegraph` không có subcommand cài đặt tự động (`codegraph install` không tồn tại — CLI chỉ có `init` / `deinit` / `serve`). Flow chuẩn là làm thủ công: `codegraph init` để dựng index, rồi đăng ký `codegraph serve --mcp` làm MCP server cho Claude Code. Skill này đóng gói lại đúng flow thủ công đó, có kiểm tra từng bước để không lặp lại việc đã xong hoặc ghi đè cấu hình đang hoạt động.

## Phạm vi & ranh giới với skill khác
- **Dùng khi**: cần setup/kết nối/kiểm tra lại hạ tầng `codegraph` cho một repo.
- **Không dùng khi**: cần review/phân tích bảo mật code → dùng skill `review-code` (skill đó sẽ tự gợi ý chạy `codegraph-setup` nếu phát hiện tool `codegraph_*` chưa sẵn sàng).

## Cách hoạt động (tóm tắt)
1. Kiểm tra `.codegraph/` đã tồn tại ở root repo chưa — làm trước tiên vì user có thể đã tự chạy `codegraph init` từ trước, tránh chạy `init` mù quáng hoặc ghi đè.
2. Tìm `codegraph` binary và lấy **absolute path** của nó (`which codegraph`, hoặc dò trong rc file như `~/.zshrc` rồi `ls` trực tiếp thư mục đó — không `source` rc file, xem lý do ở mục dưới). Không tìm thấy ở đâu → dừng và báo cho user, không tự ý cài đặt.
3. Kiểm tra MCP server `codegraph` đã đăng ký và đang Connected chưa (`claude mcp list`); nếu chưa có, hoặc có nhưng lỗi ENOENT do PATH → đăng ký/đăng ký lại bằng **absolute path** vừa tìm ở bước 2 (`claude mcp add codegraph --scope user -- <absolute-path>/codegraph serve --mcp`), không dùng bare `codegraph`.
4. Dựa vào kết quả bước 1: chưa có `.codegraph/` → chạy `codegraph init --progress`; đã có → hỏi trước khi re-index thay vì tự ý ghi đè.
5. Báo cáo ngắn gọn trạng thái MCP + index, và luôn nhắc mở phiên Claude Code mới sau khi add/sửa entry ở bước 3.

## Lưu ý quan trọng
- **`source rc file` trong Bash tool không sửa được MCP.** MCP subprocess do tiến trình Claude Code chính spawn với PATH của lúc phiên khởi động — không liên quan tới PATH của một shell con do Bash tool tạo ra khi chạy `source`. `which codegraph` báo "tìm thấy" ngay sau `source` chỉ đúng trong đúng lệnh Bash đó, không có nghĩa MCP đã sửa được. Vì vậy skill không dùng `source` để verify, mà đăng ký MCP bằng absolute path để né hẳn vấn đề PATH-theo-phiên.
- Tool `codegraph_*` **không xuất hiện ngay** trong phiên vừa chạy setup/sửa lỗi — cần mở phiên Claude Code mới để xác nhận, kể cả khi đã dùng absolute path.
- Skill không tự ý xoá/ghi đè cấu hình MCP hoặc index đã tồn tại mà không hỏi trước.
- Việc cài đặt binary `codegraph` (nếu chưa có ở đâu trên máy) nằm ngoài phạm vi skill này — skill chỉ kiểm tra và gợi ý, không tự tải/cài từ nguồn không xác thực.
