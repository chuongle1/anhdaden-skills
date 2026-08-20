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
   - Nếu `codegraph init` không chạy được (tool không có sẵn, repo không hỗ trợ) hoặc các tool `codegraph_*` không xuất hiện trong danh sách tool khả dụng, nêu rõ điều này rồi fallback sang khảo sát thủ công (Grep/Explore) trước khi tiếp tục — không im lặng bỏ qua bước này.
3. Thu thập code cần đọc theo phạm vi, dựa trên kết quả truy vấn codegraph ở bước 2:
   - **Phạm vi cụ thể**: đọc toàn bộ đoạn code liên quan, dùng `codegraph_callers`/`codegraph_callees`/`codegraph_references` để lần ra các hàm/module liên đới nếu cần hiểu luồng dữ liệu thực tế — không suy đoán hành vi.
   - **Toàn bộ repo**: dùng `codegraph_search_by_annotation`/`codegraph_files` để xác định entry point, và `codegraph_search_by_call` để liệt kê toàn bộ nơi gọi tới các sink nguy hiểm theo từng nhóm checklist — thay vì đọc ngẫu nhiên. Mục tiêu là bao phủ đủ các nhóm rủi ro trên toàn repo, không phải đọc hết mọi dòng code.
4. Áp dụng từng nhóm rủi ro trong `references/checklist.md`. Với mỗi nhóm, chỉ báo cáo nếu tìm thấy bằng chứng cụ thể trong code (dòng, biến, luồng gọi) — không liệt kê rủi ro lý thuyết chung chung.
5. Với mỗi phát hiện, xác nhận lại bằng `codegraph_flow`/`codegraph_search_flow` (hoặc `codegraph_callers`+`codegraph_callees` nối tay) để trace input → xử lý → sink, đảm bảo không bỏ sót lời gọi gián tiếp và thực sự có thể khai thác trong ngữ cảnh code này.
6. Xếp hạng mức độ nghiêm trọng: Critical / High / Medium / Low (dựa trên khả năng khai thác + tác động).
7. Xuất kết quả theo format ở mục "Output" bên dưới. Nếu review toàn bộ repo, nhóm kết quả theo file/module để dễ theo dõi, và nêu rõ những phần nào đã quét/chưa quét nếu repo quá lớn để đọc hết.

## Output
Bảng theo severity giảm dần:

| Severity | File:Line | Vấn đề | Rủi ro / Kịch bản khai thác | Đề xuất khắc phục |
|---|---|---|---|---|
| High | app.py:42 | ... | ... | ... |

Nếu không tìm thấy vấn đề nào đáng báo cáo sau khi áp dụng đầy đủ checklist, nói rõ "không phát hiện vấn đề bảo mật đáng kể" thay vì cố tạo ra phát hiện để có nội dung trả lời.

## Tham khảo thêm
Xem `references/checklist.md` để biết chi tiết từng nhóm rủi ro cần kiểm tra.
