# BỘ HỒ SƠ DỰ ÁN QUẢN LÝ SIÊU THỊ ĐIỆN MÁY

**Học phần:** Cơ sở dữ liệu  
**Phiên bản:** 1.0 — 15/11/2026  
**Nhóm:** [Tên nhóm] · **Lớp:** [Lớp] · **Giảng viên:** [Họ tên]  
**Căn cứ thiết kế:** ERD được dẫn tại https://github.com/taiseve7/pttc_qldapm/blob/main/dienmay-backend/docs/erd.md và tệp `schema-sqlserver.sql` do người học cung cấp.

> **Quy ước độ tin cậy.** Nội dung đánh dấu **Đã xác nhận từ schema** được suy ra trực tiếp từ DDL SQL Server. Nội dung **Đề xuất** là kế hoạch/mô tả để nhóm rà soát. Các trường trong ngoặc vuông và bằng chứng thực địa phải được nhóm bổ sung; tài liệu không khẳng định dự án đã khảo sát, triển khai, kiểm thử hoặc nghiệm thu khi chưa có bằng chứng.

## Danh mục sản phẩm nộp theo Phần IX

| STT | Sản phẩm | Tình trạng trong bộ hồ sơ |
|---:|---|---|
| 1 | Báo cáo dự án | Có tại mục A |
| 2 | Slide thuyết trình | Dàn ý 12 slide tại mục B, có thể chuyển sang PowerPoint |
| 3 | Căn cứ pháp lý, tài liệu liên quan | Khung tra cứu tại mục C; cần đối chiếu quy định hiện hành và yêu cầu giảng viên |
| 4 | Kế hoạch dự án | Mục D, lịch mẫu cần cập nhật ngày thực tế |
| 5 | Khảo sát thu thập thông tin | Bộ câu hỏi và phiếu khảo sát tại mục E; chưa điền kết quả giả định |
| 6 | Tài liệu phân tích | Mục F: phạm vi, tác nhân, yêu cầu, quy tắc nghiệp vụ |
| 7 | Tài liệu thiết kế | Mục G: mô hình dữ liệu, quan hệ, ràng buộc, nhận xét schema |
| 8 | Code / kịch bản chạy thử | Mục H: mã nguồn tham chiếu và luồng demo; mã nguồn đầy đủ nằm ở repository dự án |
| 9 | Kiểm thử phần mềm | Mục I: kịch bản kiểm thử đề xuất, cột kết quả chờ chạy thực tế |
| 10 | Đào tạo, chuyển giao | Mục J: đề cương và biên bản mẫu |
| 11 | Rủi ro | Mục K: danh mục và ứng phó |
| 12 | Nhật ký thi công, giám sát | Mục L: biểu mẫu ghi theo ngày thực tế |
| 13 | Biên bản làm việc | Mục M: mẫu |
| 14 | Tờ trình, quyết định | Mục N: không áp dụng như hồ sơ pháp nhân; mẫu quyết định nội bộ nhóm nếu môn học yêu cầu |
| 15 | Thương thảo / hợp đồng / nghiệm thu hợp đồng | Mục O: không phát sinh với bài tập học thuật; chỉ dùng mẫu minh họa, không ký thay giao dịch thật |
| 16 | Biên bản bàn giao, nghiệm thu | Mục P: mẫu học thuật, cần điền kết quả thực tế |
| 17 | Video chạy thử | Mục Q: checklist quay và chỗ chèn liên kết |
| 18 | Phụ lục bổ sung | Mục R: dữ liệu mẫu, từ điển dữ liệu rút gọn, truy vấn báo cáo gợi ý |

---

# A. Báo cáo dự án

## A1. Tóm tắt

Đề tài xây dựng cơ sở dữ liệu cho phần mềm quản lý siêu thị điện máy. Schema SQL Server trong tệp được cung cấp định nghĩa 14 bảng phục vụ tài khoản và phân quyền, danh mục/sản phẩm, nhà cung cấp, nhập hàng và từng thiết bị theo serial/IMEI, khách hàng, hóa đơn, bảo hành/sửa chữa và đổi trả. Thiết kế dùng khóa chính, khóa ngoại, CHECK, UNIQUE, chỉ mục và giao dịch DDL để bảo vệ tính toàn vẹn. Hàng tồn được xác định từ số thiết bị `product_item` có trạng thái `IN_STOCK`, phù hợp hàng điện máy cần truy vết từng chiếc.

Báo cáo này tập trung vào phân tích và thiết kế dữ liệu theo schema được cung cấp. Việc ứng dụng chạy thành công, hiệu năng thực tế, mức độ đáp ứng người dùng và kết quả kiểm thử chỉ được kết luận sau khi nhóm thực thi, ghi bằng chứng và hoàn tất các ô kết quả.

## A2. Bối cảnh và vấn đề

Hoạt động cửa hàng điện máy liên kết nhiều nghiệp vụ: quản lý mặt hàng, theo dõi số máy/IMEI, nhập hàng từ nhà cung cấp, bán và thanh toán, lưu khách hàng, kích hoạt bảo hành, tiếp nhận sửa chữa và xử lý đổi trả. Nếu lưu dữ liệu rời rạc, cửa hàng khó truy ra nguồn nhập, lịch sử bán, trạng thái thiết bị và quyền xử lý của nhân viên. Cơ sở dữ liệu quan hệ giúp liên kết các giao dịch bằng khóa ngoại, hạn chế dữ liệu trùng và áp đặt các quy tắc cơ bản ngay tại tầng dữ liệu.

## A3. Mục tiêu

- Thiết kế lược đồ SQL Server nhất quán cho các quy trình cốt lõi của cửa hàng.
- Bảo đảm định danh sản phẩm/thiết bị và truy vết vòng đời nhập–tồn–bán–bảo hành/đổi trả.
- Thể hiện quan hệ dữ liệu, khóa, ràng buộc và chỉ mục để phục vụ truy vấn nghiệp vụ.
- Xây dựng kịch bản cài đặt, demo và kiểm thử có thể lặp lại.
- Tạo tài liệu để nhóm hoàn thiện bằng kết quả khảo sát và chạy thử thực tế.

## A4. Phạm vi

**Trong phạm vi dữ liệu:** tài khoản nhân viên với bốn vai trò; danh mục; sản phẩm; nhà cung cấp; phiếu nhập và dòng hàng; serial/IMEI theo dòng nhập; từng thiết bị; khách hàng; hóa đơn và chi tiết; bảo hành; phiếu sửa chữa; phiếu đổi trả. Schema có thông tin giao hàng trên hóa đơn.

**Ngoài phạm vi được xác nhận bởi DDL:** quản lý khuyến mại/thuế chi tiết, kế toán tổng hợp, chấm công, nhiều chi nhánh, phân quyền theo từng thao tác, lịch sử audit chuyên biệt, tích hợp hãng vận chuyển/cổng thanh toán, báo cáo lợi nhuận đầy đủ. Có thể bổ sung khi yêu cầu thực tế đòi hỏi.

## A5. Đối tượng sử dụng

- Quản lý (`MANAGER`): theo dõi hàng hóa, nhân viên và tình hình giao dịch theo quyền ứng dụng.
- Thu ngân (`CASHIER`): tạo hóa đơn, gắn thiết bị và thanh toán.
- Thủ kho (`WAREHOUSE_KEEPER`): quản lý nhà cung cấp, phiếu nhập, serial và tồn kho.
- Kỹ thuật viên (`TECHNICIAN`): xử lý phiếu bảo hành/sửa chữa.

Phân vai trên là diễn giải từ danh sách `role`; quyền cụ thể phải xác nhận trong mã nguồn/API.

## A6. Kiến trúc dữ liệu và quy trình nghiệp vụ

Luồng nhập: `supplier → goods_receipt → goods_receipt_line → goods_receipt_line_serial → product_item`. Khi phiếu nhập hoàn tất, ứng dụng cần tạo/cập nhật thiết bị tương ứng và ghi nhận giá vốn, ngày nhập. Luồng bán: `customer (tùy chọn) → invoice → invoice_item → product_item`; thanh toán chuyển hóa đơn sang `PAID`, cập nhật thiết bị `SOLD` và tạo bảo hành theo chính sách. Luồng hậu mãi: thiết bị liên kết với `warranty`, `warranty_ticket` hoặc `return_ticket`.

Các thay đổi nhiều bảng trong cùng nghiệp vụ nên chạy trong một transaction ở ứng dụng. DDL có `version` trên `product_item` như một tín hiệu cho optimistic locking; cần xác minh entity/JPA và xử lý xung đột trong code.

## A7. Kết quả thiết kế có thể xác nhận

- Khóa chính identity cho thực thể chính; SKU, tên đăng nhập, mã chứng từ, serial và số điện thoại khách hàng được đặt duy nhất.
- Thiết bị có trạng thái `IN_STOCK`, `SOLD`, `RETURNED`, `DEFECTIVE`; tồn kho nên tính từ trạng thái thay vì duy trì một số tổng dễ lệch.
- Hóa đơn và phiếu nhập có trạng thái; chi tiết hóa đơn lưu giá bán/giảm giá tại thời điểm giao dịch.
- `CHECK` giới hạn một số miền giá trị và số tiền; chỉ mục hỗ trợ tra cứu theo danh mục, trạng thái, khách hàng, nhân viên và thời gian.
- `NVARCHAR` hỗ trợ văn bản tiếng Việt; `DATETIME2(6)` lưu mốc thời gian; tiền dùng `DECIMAL` không có phần thập phân, phù hợp giả định tiền VND nguyên đồng cần xác nhận.

## A8. Đánh giá và điểm cần rà soát

1. `invoice.ck_invoice_payment_method` cho phép NULL do SQL CHECK với UNKNOWN không bị loại; cần quyết định ràng buộc `payment_method` bắt buộc khi `PAID`.
2. `invoice.ck_invoice_paid` kiểm tra `amount_paid >= total`, nhưng chưa ràng buộc `change_amount = amount_paid - total`, `discount <= subtotal`, hay công thức `total = subtotal - discount`.
3. `invoice_item` không ràng buộc `product_item_id` thuộc cùng `product_id`; ứng dụng cần kiểm tra hoặc bổ sung cơ chế phù hợp.
4. `goods_receipt_line_serial` có thể lặp serial/IMEI trong cùng hoặc khác dòng; nên thêm unique thích hợp và kiểm tra serial đã tồn tại trong `product_item`.
5. `warranty` cho phép nhiều bản ghi cùng thiết bị; nếu nghiệp vụ chỉ có một bảo hành hiệu lực thì cần filtered unique index hoặc quy tắc ứng dụng.
6. `return_ticket` có thể có `refund_amount` với loại EXCHANGE; cần làm rõ quy tắc theo loại. DDL cũng chưa bảo đảm thiết bị trả thuộc hóa đơn đã nêu.
7. Bảng line không có unique `(goods_receipt_id, product_id)`; nếu mỗi sản phẩm chỉ có một dòng/phiếu, cân nhắc thêm.
8. Dữ liệu kiểm toán (`created_by`, `updated_by`) là chuỗi, không có FK tới người dùng; cần cân nhắc chính sách lưu vết và xóa tài khoản.
9. Trạng thái `RETURNED` của thiết bị cần quy tắc chuyển trạng thái khi xử lý đổi trả; cần tránh thiết bị vừa bị bán vừa còn tồn.
10. Phải xác nhận thứ tự chạy và idempotency của `sp_addextendedproperty`, vì chạy lại script có thể báo thuộc tính đã tồn tại.

## A9. Kết luận

Schema cung cấp nền dữ liệu hợp lý cho quản lý cửa hàng điện máy theo từng thiết bị. Các ưu điểm rõ nhất là truy xuất serial/IMEI, liên kết chứng từ bằng FK, giới hạn miền trạng thái và chỉ mục tra cứu. Trước khi nghiệm thu cần chạy script trên SQL Server phiên bản mục tiêu, kiểm tra ánh xạ JPA `ddl-auto=validate`, xác thực quy tắc liên bảng và ghi kết quả kiểm thử. Nhóm cần hoàn thiện khảo sát, mã nguồn chạy được, ảnh/video minh chứng và thông tin nhóm.

# B. Dàn ý slide thuyết trình (12 slide)

1. **Quản lý siêu thị điện máy** — tên đề tài, nhóm/lớp, thành viên.
2. **Bài toán nghiệp vụ** — quản lý nhập, serial, bán, bảo hành, đổi trả.
3. **Mục tiêu và phạm vi** — những gì hệ thống xử lý; những phần chưa bao gồm.
4. **Tác nhân và phân quyền** — MANAGER, CASHIER, WAREHOUSE_KEEPER, TECHNICIAN.
5. **Quy trình nhập hàng** — nhà cung cấp → phiếu nhập → dòng hàng → thiết bị.
6. **Quy trình bán hàng** — khách hàng → hóa đơn → từng thiết bị → thanh toán.
7. **Bảo hành và hậu mãi** — warranty, warranty_ticket, return_ticket.
8. **Mô hình dữ liệu** — chèn hình ERD từ repository; nhấn mạnh quan hệ FK và serial.
9. **Ràng buộc dữ liệu** — PK/FK/UNIQUE/CHECK, trạng thái, chỉ mục.
10. **Demo và kiểm thử** — ảnh chụp màn hình thật, ca kiểm thử đã chạy, kết quả thực tế.
11. **Rủi ro và hướng cải tiến** — đồng thời bán một thiết bị, dữ liệu serial lặp, giao dịch nhiều bảng.
12. **Kết luận và hỏi đáp** — kết quả đã xác minh, phần còn lại, liên kết video.

*Lưu ý làm slide: không đưa kết quả chưa chạy hoặc dữ liệu khảo sát chưa thu thập; ghi nguồn ERD/schema trong ghi chú slide cuối.*

# C. Căn cứ pháp lý và tài liệu liên quan

Đây là bài tập học thuật; các văn bản dưới đây là danh mục cần giảng viên/nhóm kiểm tra tính áp dụng và tình trạng hiệu lực tại ngày nộp, không phải tư vấn pháp lý. Không đưa dữ liệu cá nhân thật vào bản demo.

- Tài liệu yêu cầu môn học và đề cương học phần: [tên trường/khoa, phiên bản, ngày].
- Tài liệu dự án: ERD trong repository được nêu ở bìa; `schema-sqlserver.sql`; README và API docs của repository; hướng dẫn chạy của nhóm.
- Microsoft SQL Server: tài liệu chính thức về `CREATE TABLE`, khóa ngoại, CHECK constraints, filtered indexes, transactions, `sp_addextendedproperty`; ghi phiên bản SQL Server đã dùng.
- Java/Spring Boot/Hibernate: tài liệu chính thức đúng phiên bản trong dự án; xác nhận cấu hình `ddl-auto=validate` và nationalized character data.
- Quy định bảo vệ dữ liệu cá nhân và giao dịch điện tử tại Việt Nam: tra cứu văn bản hiện hành từ Cơ sở dữ liệu quốc gia về văn bản pháp luật/Cổng thông tin Chính phủ trước khi trích dẫn. Chỉ mô tả dữ liệu tối thiểu, mục đích sử dụng, phân quyền và thời hạn lưu theo chính sách được duyệt.
- Quy định về bảo vệ quyền lợi người tiêu dùng, hóa đơn/chứng từ, bảo hành hàng hóa: chỉ liệt kê nếu nhóm khảo sát nghiệp vụ thực tế và xác định phạm vi có liên quan.

| Tài liệu | Cơ quan/phiên bản | Điều khoản được dùng | Ngày tra cứu | Người xác nhận |
|---|---|---|---|---|
| [Tên văn bản] | [Cơ quan, số hiệu] | [Điều/khoản] | [dd/mm/yyyy] | [Tên] |

# D. Kế hoạch dự án

Lịch dưới đây là **mẫu theo tuần**, thay bằng lịch môn học thực tế.

| Giai đoạn | Công việc | Sản phẩm/bằng chứng | Phụ trách | Thời lượng mẫu |
|---|---|---|---|---|
| 1. Khởi động | Chốt mục tiêu, phân vai, repository | Biên bản khởi động, backlog | [Tên] | Tuần 1 |
| 2. Khảo sát | Phỏng vấn/quan sát, ghi quy trình | Phiếu khảo sát đã xác nhận | [Tên] | Tuần 1–2 |
| 3. Phân tích | Yêu cầu, use case, quy tắc nghiệp vụ | Tài liệu phân tích | [Tên] | Tuần 2 |
| 4. Thiết kế | ERD, chuẩn hóa, DDL, từ điển dữ liệu | ERD, schema, review | [Tên] | Tuần 3 |
| 5. Phát triển | Backend/frontend, seed dữ liệu demo | Commit, hướng dẫn cài đặt | [Tên] | Tuần 4–6 |
| 6. Kiểm thử | Chạy ca kiểm thử, sửa lỗi, regression | Bảng kết quả, lỗi và ảnh chụp | [Tên] | Tuần 7 |
| 7. Hoàn thiện | Báo cáo, slide, video, tập dượt | Hồ sơ nộp | [Tên] | Tuần 8 |

**Theo dõi:** họp [tần suất]; kênh trao đổi [ ]; repo/nhánh [ ]; tiêu chí hoàn thành một việc: có người phụ trách, bằng chứng, review và trạng thái. Ràng buộc: lịch giảng viên, thiết bị SQL Server, kết nối mạng, dữ liệu mẫu.

# E. Khảo sát thu thập thông tin

**Trạng thái:** chưa có dữ liệu khảo sát đầu vào; dùng phiếu sau để thu thập và đính kèm bằng chứng được phép chia sẻ.

## E1. Thông tin buổi khảo sát

Ngày/giờ: [ ] · Hình thức/địa điểm: [ ] · Người được hỏi/chức danh: [ ] · Người ghi: [ ] · Đồng ý ghi chú/ghi âm: [có/không] · Bằng chứng lưu: [ ]

## E2. Câu hỏi phỏng vấn

1. Hiện cửa hàng ghi nhận hàng hóa, serial/IMEI và tồn kho bằng cách nào?
2. Khi nhập hàng, ai tạo phiếu, ai duyệt, dữ liệu nào bắt buộc? Có trường hợp nhập thiếu serial không?
3. Quy tắc đặt mã sản phẩm/phiếu nhập/hóa đơn là gì?
4. Một hóa đơn có thể bán thiết bị chưa có serial không? Có đặt cọc, trả góp, nhiều phương thức thanh toán không?
5. Khi hủy hóa đơn hoặc hoàn tiền, thiết bị và tồn kho chuyển trạng thái thế nào?
6. Cách xác định thời hạn bảo hành? Bảo hành hãng và bảo hành cửa hàng khác nhau ra sao?
7. Luồng nhận sửa chữa gồm các trạng thái, người xử lý, linh kiện và chi phí nào?
8. Điều kiện đổi trả, thời hạn và cách xử lý máy đổi/máy hoàn tiền?
9. Vai trò nhân viên được xem/sửa/duyệt dữ liệu nào?
10. Báo cáo cần xem theo ngày, sản phẩm, nhân viên, nhà cung cấp hoặc tồn kho nào?
11. Dữ liệu khách hàng cần thu thập tối thiểu là gì? Ai được xem và trong bao lâu?
12. Tình huống lỗi thường gặp và cách xử lý hiện nay?

## E3. Phiếu ghi kết quả

| Chủ đề | Hiện trạng được người tham gia xác nhận | Vấn đề | Yêu cầu/ràng buộc | Bằng chứng |
|---|---|---|---|---|
| Nhập hàng | [ ] | [ ] | [ ] | [ ] |
| Bán hàng | [ ] | [ ] | [ ] | [ ] |
| Bảo hành/đổi trả | [ ] | [ ] | [ ] | [ ] |
| Phân quyền/báo cáo | [ ] | [ ] | [ ] | [ ] |

# F. Tài liệu phân tích

## F1. Yêu cầu chức năng suy ra từ schema

- FR01 Quản lý tài khoản, trạng thái hoạt động và vai trò nhân viên.
- FR02 Quản lý danh mục, sản phẩm, SKU, giá bán, bảo hành, ngưỡng tồn.
- FR03 Quản lý nhà cung cấp và thông tin liên hệ/mã số thuế.
- FR04 Lập phiếu nhập, thêm dòng sản phẩm và serial/IMEI; theo dõi trạng thái phiếu.
- FR05 Theo dõi từng thiết bị, nguồn phiếu nhập, giá vốn, thời điểm nhập/bán và trạng thái.
- FR06 Quản lý khách hàng và tra cứu giao dịch theo khách hàng.
- FR07 Lập hóa đơn, chọn thiết bị, ghi giảm giá, thanh toán, giao hàng và hủy.
- FR08 Ghi nhận bảo hành thiết bị và thông tin khách hàng/hóa đơn liên quan.
- FR09 Tiếp nhận, phân công và cập nhật tiến độ phiếu sửa chữa.
- FR10 Tiếp nhận yêu cầu hoàn tiền/đổi máy và lưu quyết định xử lý.
- FR11 Tra cứu báo cáo cơ bản về tồn, hóa đơn, nhập hàng, bảo hành và đổi trả.

FR11 là nhu cầu đề xuất, cần đối chiếu khả năng của ứng dụng trong repository.

## F2. Yêu cầu phi chức năng đề xuất

- NFR01 Tính toàn vẹn: tuân thủ khóa và kiểm tra dữ liệu tại DB; nghiệp vụ liên bảng được transaction.
- NFR02 Đồng thời: ngăn hai thu ngân bán cùng một thiết bị; kiểm tra optimistic locking/atomic update.
- NFR03 Bảo mật: BCrypt hash, xác thực/phân quyền ở ứng dụng, không lưu mật khẩu gốc, không commit secret.
- NFR04 Bảo vệ dữ liệu: hạn chế dữ liệu cá nhân, phân quyền truy cập và dùng dữ liệu demo giả.
- NFR05 Khả năng bảo trì: migration có version, script cài đặt và seed riêng, sao lưu/khôi phục được hướng dẫn.
- NFR06 Hiệu năng: xác định SLA và số bản ghi dự kiến qua khảo sát; chưa có số liệu đo trong hồ sơ này.

## F3. Quy tắc nghiệp vụ

- BR01 SKU, username, mã hóa đơn/phiếu nhập, serial thiết bị là duy nhất.
- BR02 Tồn kho khả dụng = số `product_item` đang `IN_STOCK` (xác nhận theo mô tả extended property).
- BR03 Phiếu nhập chỉ có `DRAFT`, `COMPLETED`, `CANCELLED`; hoàn tất phải nhất quán với thiết bị được tạo.
- BR04 Hóa đơn chỉ có `DRAFT`, `PAID`, `CANCELLED`; hóa đơn PAID yêu cầu thời điểm trả, phương thức và số tiền đã trả tối thiểu bằng tổng.
- BR05 Chi tiết hóa đơn là từng chiếc máy; mỗi thiết bị không lặp hai lần trong cùng hóa đơn.
- BR06 Giá/giảm giá/tiền không âm; giảm giá từng dòng không vượt đơn giá.
- BR07 Thời hạn bảo hành không âm và không vượt 120 tháng; ngày hết hạn không trước ngày bắt đầu.
- BR08 Đổi hàng (`EXCHANGE`) yêu cầu thiết bị đổi; chỉ có tối đa một yêu cầu PENDING cho một thiết bị.
- BR09 Các quy tắc chuyển trạng thái và hủy/đổi/hoàn kho cần xác nhận với nghiệp vụ, vì CHECK chỉ giới hạn giá trị chứ không kiểm soát luồng chuyển.

## F4. Tác nhân và use case

| Tác nhân | Use case chính |
|---|---|
| Quản lý | Quản lý tài khoản/sản phẩm/nhà cung cấp; xem báo cáo; kiểm tra giao dịch |
| Thu ngân | Tìm khách hàng/sản phẩm; lập hóa đơn; thanh toán; xem lịch sử bán |
| Thủ kho | Quản lý nhập hàng; thêm serial; kiểm kê; tra cứu tồn |
| Kỹ thuật viên | Xem ticket được giao; chẩn đoán; cập nhật tiến độ/kết quả |
| Khách hàng | Là đối tượng dữ liệu/nhận hóa đơn, bảo hành, sửa chữa; không có tài khoản trong schema |

# G. Tài liệu thiết kế

## G1. Danh sách thực thể

1 `app_user` — nhân viên; 2 `category` — danh mục; 3 `product` — mặt hàng; 4 `supplier` — nhà cung cấp; 5 `goods_receipt` — phiếu nhập; 6 `goods_receipt_line` — dòng nhập; 7 `goods_receipt_line_serial` — serial khai báo khi nhập; 8 `product_item` — từng thiết bị; 9 `customer` — khách hàng; 10 `invoice` — hóa đơn; 11 `invoice_item` — từng máy bán; 12 `warranty` — bảo hành; 13 `warranty_ticket` — yêu cầu sửa chữa/bảo hành; 14 `return_ticket` — đổi trả.

*Kiểm kê tệp DDL được cung cấp có 14 lệnh `CREATE TABLE` và danh sách thực thể nêu ngay trên gồm 14 bảng. Cần so khớp ERD/repository trước khi chốt, vì ERD có thể thuộc phiên bản khác.*

## G2. Quan hệ chính

- `category 1 — N product`; `supplier 1 — N goods_receipt`.
- `goods_receipt 1 — N goods_receipt_line`; `product 1 — N goods_receipt_line`.
- `goods_receipt_line 1 — N goods_receipt_line_serial`; `product 1 — N product_item`; `goods_receipt 1 — N product_item` (FK tùy chọn).
- `customer 1 — N invoice` (khách trên hóa đơn tùy chọn); `app_user 1 — N invoice` với vai trò thu ngân.
- `invoice 1 — N invoice_item`; `product_item 1 — N invoice_item` theo cấu trúc FK hiện tại nhưng unique theo invoice + item.
- `product_item 1 — N warranty`; `invoice/customer 1 — N warranty` (tùy chọn).
- `product_item 1 — N warranty_ticket`; `warranty/customer/app_user 1 — N ticket` tùy chọn.
- `invoice 1 — N return_ticket`; `product_item 1 — N return_ticket`; `product_item` có thể tham chiếu thiết bị đổi.

## G3. Chỉ số và kiểu dữ liệu

- PK: `BIGINT IDENTITY`; mã/chuỗi: `NVARCHAR`; ngày giờ: `DATETIME2(6)`; ngày bảo hành: `DATE`; tiền: `DECIMAL(15,0)`/`DECIMAL(18,0)`; cờ: `BIT`.
- Index trên quan hệ FK thường dùng và cột truy vấn trạng thái/thời gian; filtered unique index cho IMEI/MST/phiếu đổi trả pending.
- Cấu hình Hibernate theo đầu tệp: `hibernate.use_nationalized_character_data=true`, `ddl-auto=validate`. Phải xác nhận runtime SQL Server và entity tương ứng.

## G4. Từ điển dữ liệu rút gọn

| Bảng | Khóa/thuộc tính tiêu biểu | Ý nghĩa |
|---|---|---|
| app_user | id PK, username UQ, role CHECK | tài khoản nhân viên |
| category | id PK, name UQ | nhóm hàng |
| product | id PK, sku UQ, category_id FK, price CHECK | danh mục sản phẩm bán |
| supplier | id PK, tax_code filtered UQ | nhà cung cấp |
| goods_receipt | id PK, code UQ, supplier_id FK, status | chứng từ nhập |
| goods_receipt_line | id PK, goods_receipt_id/product_id FK | sản phẩm và số lượng/giá nhập |
| goods_receipt_line_serial | line_id FK, serial_number, imei | serial được khai báo cho dòng nhập |
| product_item | id PK, serial_number UQ, imei filtered UQ, product_id FK, status | một thiết bị vật lý |
| customer | id PK, phone UQ | hồ sơ khách hàng |
| invoice | id PK, code UQ, cashier_id FK, customer_id FK | hóa đơn/thanh toán/giao hàng |
| invoice_item | id PK, invoice_id/product_item_id/product_id FK | thiết bị bán và snapshot giá |
| warranty | id PK, product_item_id FK, start/end date | kỳ bảo hành |
| warranty_ticket | id PK, code UQ, item/technician FK | tiếp nhận sửa chữa |
| return_ticket | id PK, code UQ, invoice/item FK, type/status | yêu cầu đổi trả |

## G5. ERD

Dùng sơ đồ `docs/erd.md` trong repository để chèn hình ERD nguyên bản. Tệp SQL được cung cấp có 14 lệnh `CREATE TABLE`; cần đối chiếu nội dung GitHub vì con số này có thể khác phiên bản ERD. ERD cuối báo cáo phải khớp DDL thực thi.

# H. Hướng dẫn chạy thử chương trình

**Điều kiện tiên quyết:** kiểm tra README của repository; cài đúng JDK, build tool, Node (nếu frontend), SQL Server, driver và biến môi trường theo dự án. Không ghi mật khẩu thật vào báo cáo, slide, lệnh hoặc video.

1. Tải mã nguồn từ repository: `https://github.com/taiseve7/pttc_qldapm`.
2. Tạo database trống trên SQL Server phiên bản tương thích (DDL ghi >= 2016).
3. Chạy `schema-sqlserver.sql` bằng SQL Server Management Studio/sqlcmd. Trong hướng dẫn tệp có dạng `sqlcmd -S <host> -d <db> -U <user> -i schema-sqlserver.sql`; truyền mật khẩu an toàn qua biến `SQLCMDPASSWORD` theo môi trường, không commit.
4. Xác minh tạo bảng, index, FK, CHECK và extended properties; ghi phiên bản SQL Server và log thành công/lỗi.
5. Cấu hình kết nối theo README/profile `prod,sqlserver`; dùng secret local. Đặt nationalized-character setting theo ghi chú schema và `ddl-auto=validate`.
6. Chạy migration/seed đúng hướng dẫn repository (không chạy song song một script khởi tạo khác nếu chưa xác định thứ tự).
7. Khởi động backend/frontend theo README. Ghi lệnh, commit hash, thời điểm và kết quả.
8. Đăng nhập bằng tài khoản demo do dự án cung cấp; nếu không có, tạo dữ liệu demo qua quy trình seed an toàn.
9. Quay luồng: nhập hàng có serial → xem tồn → tạo hóa đơn và thanh toán → tra cứu bảo hành → tạo phiếu sửa chữa → tạo yêu cầu đổi trả.
10. Ghi lại lỗi/ảnh màn hình và dọn dữ liệu demo nếu cần.

**Thông tin thực tế để điền:** commit `[ ]`; SQL Server `[ ]`; JDK `[ ]`; OS `[ ]`; ngày chạy `[ ]`; người chạy `[ ]`; kết quả `[ ]`; lỗi `[ ]`.

# I. Kịch bản kiểm thử phần mềm

Điền trạng thái sau khi chạy; PASS chỉ khi có bằng chứng (ảnh/log/query). Tất cả dữ liệu là giả lập.

| ID | Tình huống | Dữ liệu/tiền điều kiện | Bước kiểm tra | Kết quả mong đợi | Thực tế |
|---|---|---|---|---|---|
| TC01 | Tạo sản phẩm hợp lệ | category tồn tại, SKU mới | insert qua ứng dụng | tạo thành công | [ ] |
| TC02 | Trùng SKU | SKU đã có | tạo sản phẩm trùng | bị từ chối bởi unique | [ ] |
| TC03 | Giá âm | `price=-1` | insert/update | bị CHECK từ chối | [ ] |
| TC04 | Category không tồn tại | category_id sai | lưu product | FK từ chối | [ ] |
| TC05 | Trùng serial thiết bị | serial đã tồn tại | nhập thiết bị trùng | bị từ chối | [ ] |
| TC06 | Trạng thái thiết bị sai | status ngoài miền | cập nhật product_item | CHECK từ chối | [ ] |
| TC07 | Hoàn tất nhập hàng | supplier + line + serial hợp lệ | hoàn tất qua UI/API | phiếu, thiết bị, tồn nhất quán | [ ] |
| TC08 | Hủy phiếu nhập | phiếu DRAFT | hủy | status CANCELLED; không sinh tồn sai | [ ] |
| TC09 | Thanh toán hợp lệ | thiết bị IN_STOCK, cashier hợp lệ | tạo và trả đủ | invoice PAID; thiết bị SOLD; warranty nếu áp dụng | [ ] |
| TC10 | Thanh toán thiếu | amount_paid < total | xác nhận thanh toán | bị từ chối | [ ] |
| TC11 | Bán thiết bị đã bán | item SOLD | lập hóa đơn khác | thao tác thất bại, không bán trùng | [ ] |
| TC12 | Hai thu ngân bán cùng máy | hai phiên đồng thời | cùng xác nhận một item | chỉ một giao dịch thành công | [ ] |
| TC13 | Hóa đơn không có payment method | status PAID, phương thức NULL | lưu | ứng dụng/DB từ chối theo quy tắc đã thống nhất | [ ] |
| TC14 | Tạo phiếu đổi thiếu máy đổi | type EXCHANGE, exchange_item_id NULL | lưu | CHECK từ chối | [ ] |
| TC15 | Hai phiếu pending cùng máy | một phiếu PENDING tồn tại | tạo phiếu thứ hai | filtered unique từ chối | [ ] |
| TC16 | Warranty ngày kết thúc trước ngày bắt đầu | end < start | lưu | CHECK từ chối | [ ] |
| TC17 | Người dùng không đủ vai trò | tài khoản cashier gọi chức năng quản lý | gọi API/UI | quyền truy cập bị từ chối | [ ] |
| TC18 | Chạy lại schema | database đã có bảng/extended props | chạy lại theo quy trình | kết quả có kiểm soát; ghi script có hỗ trợ rerun không | [ ] |

**Tổng kết:** số ca chạy [ ]; PASS [ ]; FAIL [ ]; BLOCKED [ ]; lỗi mức nghiêm trọng [ ]; người xác nhận [ ]; ngày [ ]. Không ghi PASS trước khi chạy.

# J. Đào tạo và chuyển giao công nghệ

## J1. Đề cương buổi hướng dẫn (60–90 phút)

- 10 phút: mục tiêu, vai trò tài khoản và lưu ý bảo mật.
- 15 phút: danh mục, sản phẩm và tra cứu thiết bị theo serial.
- 15 phút: nhập hàng, kiểm tra serial và tồn kho.
- 15 phút: bán hàng, thanh toán, hóa đơn, giao hàng.
- 15 phút: bảo hành/sửa chữa/đổi trả.
- 10 phút: sao lưu, báo lỗi, tài liệu và hỏi đáp.

## J2. Bàn giao kỹ thuật

| Hạng mục | Vị trí/phiên bản | Người bàn giao | Người nhận | Xác nhận |
|---|---|---|---|---|
| Mã nguồn | [URL/commit] | [ ] | [ ] | [ ] |
| Database script | [tên/phiên bản] | [ ] | [ ] | [ ] |
| Hướng dẫn chạy | [đường dẫn] | [ ] | [ ] | [ ] |
| Tài khoản demo | [cấp qua kênh riêng] | [ ] | [ ] | [ ] |
| Video/tài liệu | [liên kết] | [ ] | [ ] | [ ] |

# K. Rủi ro dự án

| Rủi ro | Khả năng/ảnh hưởng | Ứng phó | Chủ sở hữu |
|---|---|---|---|
| ERD và DDL khác phiên bản | Trung bình/Cao | Chốt commit, so sánh số bảng/quan hệ, sinh ERD từ schema cuối | [ ] |
| SQL Server không tương thích cấu hình/JPA | Trung bình/Cao | Chạy validate trên DB sạch, lưu log phiên bản và driver | [ ] |
| Serial/IMEI trùng hoặc thiếu | Cao/Cao | Unique index, chuẩn hóa đầu vào, kiểm tra lô nhập trước commit | [ ] |
| Hai phiên bán cùng thiết bị | Trung bình/Cao | transaction, optimistic lock/version, test đồng thời | [ ] |
| Nghiệp vụ đổi trả mơ hồ | Trung bình/Trung bình | Phỏng vấn, lập ma trận trạng thái trước code | [ ] |
| Rò rỉ thông tin khách hàng/secret | Thấp/Cao | dữ liệu giả, phân quyền, không ghi secret vào repo/video | [ ] |
| Thiếu thời gian/bằng chứng hoàn thiện | Trung bình/Cao | mốc review hàng tuần, chia người phụ trách và lưu bằng chứng | [ ] |
| Script không chạy lại an toàn | Trung bình/Trung bình | dùng migration versioned, thử trên DB sạch và DB đã cài | [ ] |

# L. Nhật ký thi công và giám sát

| Ngày/giờ | Công việc | Người thực hiện | Kết quả/bằng chứng | Vấn đề | Người giám sát/xác nhận |
|---|---|---|---|---|---|
| [ ] | [ ] | [ ] | [link/commit/ảnh] | [ ] | [ ] |

# M. Biên bản làm việc

**BIÊN BẢN LÀM VIỆC NHÓM**  
Thời gian: [ ] · Địa điểm/kênh họp: [ ] · Chủ trì: [ ] · Thư ký: [ ]  
Thành phần: [ ] · Vắng mặt/lý do: [ ]  

**Nội dung:** [ ]  
**Ý kiến và căn cứ:** [ ]  
**Kết luận/quyết định:** [ ]  
**Việc cần làm:** việc [ ], người phụ trách [ ], hạn [ ], bằng chứng [ ].  
**Cuộc họp tiếp theo:** [ ]  

Chủ trì [ký] · Thư ký [ký] · Thành viên [ký nếu môn yêu cầu]

# N. Tờ trình / quyết định

Với bài tập học thuật không phát sinh quyết định hành chính/pháp nhân. Nếu giảng viên yêu cầu biểu mẫu quản trị nhóm, dùng tiêu đề **Quyết định phân công nhiệm vụ nội bộ nhóm**; không mạo danh đơn vị/cơ quan.

Số: [ ]/QĐ-NHÓM · Ngày [ ]  
Căn cứ: yêu cầu học phần [ ]; thống nhất nhóm tại biên bản ngày [ ].  
**Quyết định:** phân công [thành viên] phụ trách [hạng mục], thời hạn [ ]; người review [ ]; sản phẩm bàn giao [ ].  
Hiệu lực nội bộ nhóm từ [ ]. Thành viên xác nhận: [ ].

# O. Thương thảo / hợp đồng / nghiệm thu hợp đồng

**Không phát sinh trong phạm vi bài tập nhóm**, trừ khi nhóm thực sự ký kết với một bên cung cấp dịch vụ. Không sử dụng mẫu này để tạo hợp đồng giả hoặc ký thay tổ chức. Nếu giảng viên yêu cầu minh họa, gắn nhãn rõ **TÌNH HUỐNG GIẢ ĐỊNH — KHÔNG CÓ GIÁ TRỊ GIAO DỊCH** và chỉ trình bày điều khoản giả lập: phạm vi, sản phẩm, thời gian, trách nhiệm, tiêu chí nghiệm thu, bảo mật, xử lý thay đổi.

# P. Biên bản bàn giao và nghiệm thu học thuật

Ngày [ ] tại [ ], nhóm [ ] trình bày sản phẩm học phần cho [giảng viên/người đánh giá]. Thành phần: [ ].

| Hạng mục | Tiêu chí | Bằng chứng | Kết quả (Đạt/Chưa đạt) | Ghi chú |
|---|---|---|---|---|
| Database script | Cài trên SQL Server mục tiêu | Log, ảnh/query | [ ] | [ ] |
| Chức năng demo | Theo luồng đã chọn | Video/ảnh | [ ] | [ ] |
| Tài liệu | Đủ danh mục yêu cầu | Bộ hồ sơ | [ ] | [ ] |
| Kiểm thử | Có kết quả thực chạy | Bảng ca test | [ ] | [ ] |

Kết luận: [ ]. Kiến nghị/sửa chữa: [ ]. Hạn hoàn thành: [ ].  
Đại diện nhóm [ ] · Người đánh giá [ ] · Ngày ký [ ].

# Q. Video thử nghiệm phần mềm

**Liên kết video:** [chèn URL chia sẻ có quyền xem] · **Ngày quay:** [ ] · **Commit/version:** [ ] · **Người trình bày:** [ ]

Checklist quay: 1) dùng dữ liệu giả; 2) ẩn mật khẩu, token, thông tin cá nhân; 3) giới thiệu phiên bản và môi trường; 4) trình diễn nhập có serial; 5) tra cứu tồn; 6) thanh toán; 7) tra cứu bảo hành/tạo ticket; 8) thử một trường hợp từ chối hợp lệ (serial trùng hoặc bán máy đã bán); 9) nêu hạn chế đã biết; 10) kiểm tra quyền truy cập link.

# R. Phụ lục

## R1. Dữ liệu demo an toàn (ví dụ)

Dùng tên và thông tin giả như `Khách Demo`, `0900000000`, `DEMO-SKU-001`, `SERIAL-DEMO-0001`; không dùng số điện thoại/email của người thật. Mật khẩu chỉ dùng tài khoản seed cục bộ và không ghi vào hồ sơ nộp.

## R2. Truy vấn báo cáo gợi ý

**Tồn thiết bị theo SKU**

```sql
SELECT p.sku, p.name, COUNT(*) AS stock_quantity
FROM dbo.product AS p
JOIN dbo.product_item AS pi ON pi.product_id = p.id
WHERE pi.status = N'IN_STOCK'
GROUP BY p.sku, p.name
ORDER BY p.name;
```

**Hóa đơn đã thanh toán theo ngày**

```sql
SELECT CAST(paid_at AS date) AS sale_date, COUNT(*) AS invoice_count,
       SUM(total) AS revenue
FROM dbo.invoice
WHERE status = N'PAID'
GROUP BY CAST(paid_at AS date)
ORDER BY sale_date;
```

**Cảnh báo dưới ngưỡng tồn tối thiểu**

```sql
SELECT p.sku, p.name, p.min_stock, COUNT(pi.id) AS stock_quantity
FROM dbo.product AS p
LEFT JOIN dbo.product_item AS pi
  ON pi.product_id = p.id AND pi.status = N'IN_STOCK'
GROUP BY p.sku, p.name, p.min_stock
HAVING COUNT(pi.id) < p.min_stock;
```

## R3. Danh sách cần nhóm xác nhận trước khi nộp

- [ ] Thành viên, lớp, giảng viên, tên trường/khoa.
- [ ] Kiểm tra lại số bảng: DDL đính kèm hiện có 14 bảng, trong khi mô tả ban đầu ở báo cáo tạm nói 16; thống nhất với ERD và repository.
- [ ] Lấy đúng phiên bản/commit ERD và schema đưa vào bài.
- [ ] Điền kết quả khảo sát thật và xin phép chia sẻ bằng chứng.
- [ ] Chạy DDL trên SQL Server thực; lưu log và sửa hướng dẫn nếu có lỗi.
- [ ] Đối chiếu yêu cầu với chức năng thật trong source code.
- [ ] Chạy kiểm thử, điền PASS/FAIL/BLOCKED và dẫn chứng.
- [ ] Chèn ảnh ERD, ảnh ứng dụng, link video.
- [ ] Kiểm tra căn cứ pháp lý/tài liệu theo văn bản và ngày tra cứu thực tế.
- [ ] Chỉ dùng biên bản/hợp đồng/quyết định khi có đúng bối cảnh; không ký/ghi nhận giả.

