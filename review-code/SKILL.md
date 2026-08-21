---
name: review-code
description: Phân tích/review bảo mật source code theo yêu cầu cụ thể của user — có thể là một đoạn code/file/module cụ thể, hoặc quét toàn bộ repository/codebase hiện tại — theo checklist riêng của security team (input validation, injection, authN/authZ, secrets, crypto, deserialization, dependency, business logic abuse). Dùng khi user yêu cầu "phân tích code", "review code này", "audit bảo mật repo/project này", "quét toàn bộ codebase tìm lỗ hổng", hoặc dán một đoạn/file code kèm yêu cầu đánh giá. Không dùng khi chỉ cần review style/logic/hiệu năng không liên quan bảo mật (dùng skill code-review), và không dùng cho quy trình review pending diff trên branch hiện tại (dùng skill security-review).
---

## Khi nào dùng
- User đưa ra một yêu cầu review bảo mật cụ thể — có thể ở 2 dạng phạm vi:
  1. **Có chỉ định cụ thể**: một đoạn code, một file, hoặc một module.
  2. **Toàn bộ repo/codebase**: user yêu cầu audit/quét bảo mật cho cả project hiện tại, không giới hạn vào diff đang pending.

## Quy trình
1. Xác định phạm vi thực tế của yêu cầu: đoạn/file/module cụ thể, hay toàn bộ repo. Nếu chưa rõ, hỏi lại thay vì đoán.
2. **Dựng chỉ mục repo bằng codegraph** (bắt buộc, áp dụng cho cả 2 phạm vi):
   - Chạy `codegraph init` tại thư mục gốc của repo đang review. Lệnh này tạo thư mục `.codegraph/` chứa index dạng SQLite nội bộ — **không đọc trực tiếp file này**, nó chỉ được truy vấn qua MCP server `codegraph` (đã đăng ký sẵn ở user scope, cần mở phiên Claude Code mới để tool xuất hiện nếu vừa mới đăng ký).
   - Sau khi index xong, dùng các MCP tool sau để nắm cấu trúc và trace luồng gọi (thay vì đọc code rời rạc theo cảm tính):
     - `codegraph_status` — kiểm tra index đã sẵn sàng.
     - `codegraph_files` / `codegraph_list_classes` / `codegraph_list_interfaces` — nắm cấu trúc tổng thể repo.
     - `codegraph_search_by_annotation` — tìm entry point qua route/decorator (vd `@app.route`, `@RequestMapping`).
     - `codegraph_search_by_call` — tìm mọi nơi gọi tới một sink nguy hiểm cụ thể (vd `exec`, `os.system`, `subprocess`, hàm build query, `pickle.loads`) trên toàn repo.
     - `codegraph_callers` / `codegraph_callees` — trace ai gọi hàm này / hàm này gọi gì, để đi ngược từ sink về entry point hoặc xuôi từ entry point tới sink.
     - `codegraph_flow` / `codegraph_search_flow` — trace luồng dữ liệu input → sink trực tiếp.
     - `codegraph_symbol` / `codegraph_search_symbol` / `codegraph_references` / `codegraph_context` — tra định nghĩa, nơi dùng, ngữ cảnh quanh 1 symbol.
     - `codegraph_impact` — đánh giá phạm vi ảnh hưởng nếu 1 symbol bị khai thác.
   - Nếu `codegraph init` không chạy được (tool không có sẵn, repo không hỗ trợ) hoặc các tool `codegraph_*` không xuất hiện trong danh sách tool khả dụng, gợi ý chạy skill `codegraph-setup` để kiểm tra/kết nối lại trước. Nếu sau đó vẫn không được (vd đang ở phiên hiện tại nên tool `codegraph_*` chưa load), nêu rõ điều này rồi fallback sang khảo sát thủ công (Grep/Explore) — không im lặng bỏ qua bước này.
3. Thu thập code cần đọc theo phạm vi, dựa trên kết quả truy vấn codegraph ở bước 2:
   - **Phạm vi cụ thể**: đọc toàn bộ đoạn code liên quan, dùng `codegraph_callers`/`codegraph_callees`/`codegraph_references` để lần ra các hàm/module liên đới nếu cần hiểu luồng dữ liệu thực tế — không suy đoán hành vi.
   - **Toàn bộ repo**: dùng `codegraph_search_by_annotation`/`codegraph_files` để xác định entry point, và `codegraph_search_by_call` để liệt kê toàn bộ nơi gọi tới các sink nguy hiểm theo từng nhóm checklist — thay vì đọc ngẫu nhiên. Mục tiêu là bao phủ đủ các nhóm rủi ro trên toàn repo, không phải đọc hết mọi dòng code.
4. Áp dụng từng nhóm rủi ro trong `references/checklist.md`. Với mỗi nhóm, chỉ báo cáo nếu tìm thấy bằng chứng cụ thể trong code (dòng, biến, luồng gọi) — không liệt kê rủi ro lý thuyết chung chung.
5. Với mỗi phát hiện nghi ngờ, trace lại bằng `codegraph_flow`/`codegraph_search_flow` (hoặc `codegraph_callers`+`codegraph_callees` nối tay) để xác nhận input → xử lý → sink, đảm bảo không bỏ sót lời gọi gián tiếp.
6. **Verify từng phát hiện trước khi đưa vào báo cáo** — chủ động tìm lý do bác bỏ, không mặc định tin phát hiện ban đầu là đúng:
   - Kiểm tra xem giữa input và sink có lớp validation/sanitize/escape nào mà bước trace ở trên có thể đã bỏ sót không (middleware, decorator, ORM tự escape, framework mặc định an toàn).
   - Xác nhận input thực sự do bên ngoài/user kiểm soát được, không phải config nội bộ cố định hay giá trị đã qua allowlist trước đó.
   - Nếu nhiều finding cùng pattern (vd nhiều endpoint giống nhau), verify độc lập từng cái — không suy ra "cái còn lại chắc cũng vậy" từ 1 cái đã xác nhận.
7. **Xác minh bằng thực nghiệm (unit test) khi có thể** — ưu tiên bằng chứng chạy được hơn suy luận tĩnh:
   - Nếu repo có sẵn test framework và finding có thể tái hiện trong 1 unit test (không cần hạ tầng ngoài như DB/service thật), viết 1 test nhắm thẳng vào hàm/luồng nghi vấn, dùng input độc/malicious tối thiểu để chứng minh hành vi (vd input chứa payload injection và assert nó lọt qua sink mà không bị chặn) — đặt ở vị trí tạm/scratch, không commit vào repo của user trừ khi được yêu cầu.
   - Chạy test và dùng kết quả làm bằng chứng:
     - **Pass (tái hiện được hành vi khai thác)** → coi là evidence xác thực, giữ nguyên/tăng độ tin cậy của finding, ghi lại đoạn code test + output làm bằng chứng.
     - **Không tái hiện được** → hạ độ tin cậy, ưu tiên bằng chứng thực nghiệm hơn nghi ngờ ban đầu; cân nhắc loại khỏi báo cáo hoặc ghi rõ "không tái hiện được bằng test".
   - Nếu không thể viết test (cần hạ tầng phức tạp, phạm vi ngoài repo, rủi ro tác động hệ thống thật), bỏ qua bước này và dựa vào kết quả bước 6.
   - Sau khi test xác nhận, xoá/không để lại file test tạm trong working tree của user trừ khi họ muốn giữ.
8. **Tạo PoC cho finding đã xác nhận (test pass hoặc verify tĩnh đủ chắc chắn)** — soạn PoC ngắn gọn, **phi phá hoại**, để user tự chạy kiểm chứng thêm (vd request mẫu, input mẫu, script minh hoạ đọc 1 giá trị vô hại/echo/sleep để chứng minh khả năng khai thác — không thực thi hành động phá hoại thật như xoá dữ liệu, gửi dữ liệu ra ngoài, hay khai thác thật trên hệ thống production). Nêu rõ PoC chỉ dùng trong phạm vi review nội bộ mà user đã cấp quyền.
   - Finding chỉ dừng ở mức nghi ngờ, không viết được test và verify tĩnh chưa đủ chắc chắn → giữ lại nhưng hạ severity và ghi chú "cần xác minh thêm", không tạo PoC cho trường hợp này.
   - Finding bị bác bỏ ở bước 6/7 thì loại khỏi kết quả, không liệt kê "để tham khảo".
9. Xếp hạng mức độ nghiêm trọng: Critical / High / Medium / Low (dựa trên khả năng khai thác + tác động, ưu tiên nâng severity cho finding đã có evidence từ unit test/PoC).
10. **Ghi kết quả vào file `SECURITY_FINDING.md` ở thư mục gốc repo — không in chi tiết finding ra console/chat cho user.** Nếu file đã tồn tại, đọc trước rồi cập nhật (thêm finding mới, cập nhật finding trùng nếu có thông tin mới, giữ nguyên phần khác) thay vì ghi đè toàn bộ file. Trong chat, chỉ phản hồi ngắn gọn: đường dẫn file, tổng số finding theo từng severity — không dán lại nội dung bảng/PoC ra chat.

## Output — file `SECURITY_FINDING.md`
Nội dung file gồm bảng theo severity giảm dần, chỉ gồm finding đã qua bước verify ở trên:

| Severity | File:Line | Vấn đề | Rủi ro / Kịch bản khai thác | Bằng chứng | Đề xuất khắc phục |
|---|---|---|---|---|---|
| High | app.py:42 | ... | ... | Unit test pass (kèm PoC) / Verify tĩnh / Cần xác minh thêm | ... |

Với finding có unit test/PoC, đính kèm ngay bên dưới bảng (trong file, không phải trong chat): đoạn code test (nếu có) và PoC tương ứng, kèm hướng dẫn ngắn để user tự chạy thử. Nếu review toàn bộ repo, nhóm kết quả theo file/module trong cùng file này để dễ theo dõi, và nêu rõ những phần nào đã quét/chưa quét nếu repo quá lớn để đọc hết.

Nếu không tìm thấy vấn đề nào đáng báo cáo sau khi áp dụng đầy đủ checklist và verify, vẫn ghi vào `SECURITY_FINDING.md` dòng "không phát hiện vấn đề bảo mật đáng kể" kèm thời điểm/phạm vi đã review, thay vì cố tạo ra phát hiện để có nội dung.

## Tham khảo thêm
Xem `references/checklist.md` để biết chi tiết từng nhóm rủi ro cần kiểm tra.
