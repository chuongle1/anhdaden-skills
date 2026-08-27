# Checklist tổng hợp kiến trúc / overview repo

## 1. Entry point
- Route/API endpoint (REST, GraphQL, RPC).
- CLI command, cron job, message queue consumer/event handler.
- Nếu nhiều loại entry point cùng tồn tại (vd vừa có REST vừa có worker), liệt kê tách biệt theo loại.

## 2. Layering & tổ chức module
- Repo tổ chức theo layer kỹ thuật (controller/service/repository) hay theo domain/feature, hay monorepo nhiều service?
- Hướng phụ thuộc giữa các layer/module có nhất quán không (vd layer dưới không được import layer trên) — chỉ nêu nếu có bằng chứng từ `codegraph_dependencies`, không suy đoán.
- Có tầng shared/common dùng chung giữa nhiều module không, mức độ coupling ra sao.

## 3. Domain model chính
- Các entity/class trung tâm đại diện cho nghiệp vụ (không phải class tiện ích/helper).
- Quan hệ giữa các domain model chính (composition, inheritance, aggregate).

## 4. Dependency ngoài
- Database, cache, message queue, external API/service quan trọng.
- Framework/library cốt lõi định hình kiến trúc (vd ORM, DI container, web framework).

## 5. Luồng dữ liệu cho use case tiêu biểu
- Chọn 1-2 use case đại diện nhất (thường là use case chính của entry point quan trọng nhất, hoặc use case user chỉ định).
- Trace đủ từ entry point tới nơi dữ liệu dừng lại (DB, response, message publish) — không dừng giữa chừng.

## 6. Pattern & convention đáng chú ý
- Pattern kiến trúc rõ ràng (CQRS, event-driven, hexagonal, layered, microservice qua message bus...).
- Convention riêng của team/project (naming, cấu trúc thư mục đặc thù, code-gen, monorepo tooling).
- Bất kỳ điều gì khác biệt so với convention thông thường của ngôn ngữ/framework đang dùng, đáng để người mới lưu ý.

## Nguyên tắc chung khi áp dụng checklist
- Chỉ đưa vào kết quả những gì có bằng chứng cụ thể (file:line, kết quả truy vấn codegraph) — không suy đoán kiến trúc lý thuyết không khớp code thực tế.
- Ưu tiên nói ít nhưng đúng trọng tâm hơn liệt kê đầy đủ nhưng hời hợt — đây là tài liệu giúp người đọc nắm nhanh, không phải liệt kê toàn bộ codebase.
