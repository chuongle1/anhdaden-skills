---
name: codegraph-setup
description: Kiểm tra, khởi tạo (`codegraph init`) và kết nối MCP server `codegraph` (`codegraph serve --mcp`) cho Claude Code trên repo hiện tại. Dùng khi user yêu cầu "setup codegraph", "kết nối codegraph", "khởi tạo codegraph cho repo này", "connect codegraph MCP", hoặc khi một skill khác (vd review-code) báo tool codegraph_* không khả dụng và cần setup lại. Không dùng để review/phân tích code — skill này chỉ lo phần cài đặt/kết nối hạ tầng.
---

## Khi nào dùng
- User yêu cầu trực tiếp: "setup codegraph", "kết nối codegraph cho repo này", "khởi tạo codegraph", "connect codegraph MCP".
- Một skill khác (vd `review-code`) không dùng được tool `codegraph_*` và cần setup/kiểm tra lại trước khi tiếp tục.

Không có cơ chế `codegraph install` tự động nào có sẵn trong bản thân tool `codegraph` — skill này đóng gói lại quy trình thủ công (init + đăng ký MCP) thành các bước lặp lại được, tự kiểm tra từng phần trước khi làm để tránh lặp lại việc đã xong.

## ⚠️ Điểm mấu chốt: `source rc file` trong Bash tool KHÔNG sửa được MCP

`claude mcp add`/`serve --mcp` được spawn bởi **tiến trình Claude Code chính**, dùng PATH mà tiến trình đó có sẵn từ lúc khởi động phiên. Bash tool chạy mỗi lệnh trong một shell con riêng, không chia sẻ state với tiến trình chính lẫn với các lệnh Bash khác:
- `source ~/.zshrc && which codegraph` trong **cùng một lệnh** Bash tool sẽ báo "tìm thấy" — nhưng đó là PATH của shell con đó, không phải PATH của tiến trình Claude Code.
- Ngay lệnh Bash tool **tiếp theo** (hoặc `claude mcp list`, vốn cũng do tiến trình chính spawn) sẽ vẫn ENOENT, dù bước trước "thành công".

⇒ Không dùng `source` để "verify rồi coi như xong". Cách duy nhất PATH mới có hiệu lực với MCP là **mở phiên Claude Code mới** (source rc file chỉ hữu ích để tự kiểm tra binary chạy được, vd trước khi build/init).

⇒ Cách né hẳn vấn đề PATH-theo-phiên: khi đăng ký MCP, luôn dùng **absolute path** của binary (`claude mcp add codegraph --scope user -- /path/to/codegraph serve --mcp`) thay vì bare `codegraph`. Absolute path không phụ thuộc PATH của phiên nào cả.

## Quy trình
1. **Kiểm tra `.codegraph/` đã tồn tại ở root repo chưa** (`ls -la .codegraph` tại root) — làm trước tiên vì user có thể đã tự chạy `codegraph init` từ trước; kết quả này quyết định cách xử lý ở bước 4, không cần đoán hay chạy `init` mù quáng.
2. **Tìm binary và lấy absolute path** — chạy `which codegraph` hoặc `command -v codegraph`.
   - Có → đã có absolute path, ghi nhớ path này (dùng cho bước 3), sang bước 3.
   - Không có → tìm trong rc file khai báo PATH liên quan tới `codegraph` (`grep -n codegraph ~/.zshrc` hoặc rc file khớp `$SHELL`), **không cần `source`** — chỉ cần lấy đường dẫn thư mục khai báo rồi kiểm tra trực tiếp binary có tồn tại ở đó không (`ls <dir>/codegraph`):
     - Tồn tại → đã có absolute path (`<dir>/codegraph`), ghi nhớ path này, sang bước 3. Không cần source hay `which` lại — như ghi chú ở trên, việc đó không phản ánh đúng trạng thái PATH của tiến trình chính.
     - Không tồn tại ở path đó, hoặc rc file không có khai báo nào → báo rõ cho user là chưa có `codegraph` binary và dừng lại. Không tự ý tải/cài từ nguồn không xác thực. Nếu biết vị trí mã nguồn trên máy (vd repo `codegraph-rs`), có thể gợi ý build bằng `cargo build --release` — chỉ là gợi ý tham khảo, tuỳ máy.
3. **Kiểm tra MCP server đã đăng ký & còn sống chưa**: chạy `claude mcp list`, tìm entry `codegraph`.
   - Có và `✔ Connected` → coi như đã sẵn sàng, sang bước 4.
   - Có nhưng lỗi `ENOENT: Executable not found in $PATH` **và** bước 2 đã xác định binary tồn tại ở một absolute path cụ thể → nguyên nhân là entry hiện tại đăng ký bằng bare `codegraph` (hoặc phiên Claude Code hiện tại được mở trước khi PATH có dòng export đó), không phải do thiếu binary. Hỏi user trước khi remove (`claude mcp remove codegraph`), rồi add lại **bằng absolute path**: `claude mcp add codegraph --scope user -- <absolute-path>/codegraph serve --mcp`. Báo rõ: sau khi sửa vẫn phải **mở phiên Claude Code mới** để xác nhận `✔ Connected` — `claude mcp list` chạy trong phiên hiện tại có thể vẫn báo lỗi cũ vì đó cũng là tiến trình được spawn trước khi sửa.
   - Có nhưng lỗi khác (không phải ENOENT do PATH) → báo lỗi cụ thể cho user, hỏi trước khi remove và add lại — không tự ý xoá cấu hình MCP hiện có mà không hỏi.
   - Chưa có entry nào → đăng ký MCP server bằng **absolute path** đã có từ bước 2: `claude mcp add codegraph --scope user -- <absolute-path>/codegraph serve --mcp` (ưu tiên absolute path hơn bare `codegraph` ngay từ đầu, để tránh lặp lại vấn đề PATH-theo-phiên ở các phiên sau). Sau đó chạy lại `claude mcp list` — nếu vẫn thấy lỗi ở phiên hiện tại thì đó là bình thường (xem lưu ý dưới), không phải thất bại.
   - **Luôn nhắc user**: các tool `codegraph_*` (và trạng thái `✔ Connected` đáng tin cậy) chỉ xuất hiện ở **phiên Claude Code mới** mở sau khi add/sửa entry — phiên hiện tại không phản ánh đúng, không cố gọi thử `codegraph_*` hoặc chạy lại `claude mcp list` nhiều lần trong phiên hiện tại rồi kết luận là thất bại.
4. **Xử lý index dựa trên kết quả bước 1**:
   - Bước 1 cho thấy **chưa có** `.codegraph/` → chạy `codegraph init --progress` tại root repo.
   - Bước 1 cho thấy **đã có** `.codegraph/` → không tự ý chạy lại `codegraph init`; hỏi user có muốn re-index hay giữ nguyên, mặc định giữ nguyên nếu user không yêu cầu re-index.
5. **Báo cáo kết quả ngắn gọn**: index đã có sẵn từ trước hay vừa mới tạo, trạng thái MCP (Connected/chưa, và nếu vừa add/sửa entry thì nói rõ trạng thái này chỉ xác nhận được ở phiên mới), kết quả index nếu vừa chạy (số file/symbol/chain/call từ output `codegraph init`), và luôn nhắc mở phiên Claude Code mới sau khi add/sửa entry ở bước 3 — dù là lần đầu đăng ký hay sửa lỗi ENOENT do PATH.
