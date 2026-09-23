# Business Requirements — Retail Sales Analytics

**Project:** Retail Store Performance & Promotion Optimization  
**Tên tiếng Việt:** Phân tích hiệu quả kinh doanh và khuyến mãi trong chuỗi bán lẻ  
**Phiên bản:** 0.1 — phạm vi BI, 23/09/2026  
**Nguồn:** Rossmann Store Sales (`train.csv`, `store.csv`), dữ liệu lịch sử 01/01/2013–31/07/2015.

## 1. Bối cảnh và vấn đề kinh doanh

Ban quản lý chuỗi cửa hàng cần biết tình hình kinh doanh toàn hệ thống, phát hiện những cửa hàng có kết quả thấp hơn nhóm tương đồng và đánh giá **mối liên hệ** giữa ngày khuyến mãi với doanh số/lượt khách. Mục đích là lựa chọn những vấn đề cần kiểm tra và đề xuất các thử nghiệm kinh doanh có KPI theo dõi.

> Đây là bài toán phân tích trên dữ liệu lịch sử, không khẳng định phản ánh tình hình hiện tại của Rossmann. Chúng ta không thể tính lợi nhuận hay ROI khuyến mãi vì dữ liệu thiếu chi phí, giá vốn và ngân sách chương trình.

## 2. Mục tiêu (business objectives)

- Theo dõi doanh số, lượt khách và doanh số trên mỗi lượt khách theo thời gian.
- So sánh công bằng hơn giữa các cửa hàng bằng chỉ số **trên mỗi ngày cửa hàng mở** và nhóm tham chiếu cùng `StoreType`, `Assortment`.
- Phân tích kết quả trong ngày `Promo = 1` và `Promo = 0`, có phân tách theo cửa hàng, thời gian, thứ trong tuần khi cần.
- Xác định nhóm cửa hàng/thời điểm cần điều tra và đề xuất hành động **có thể thử nghiệm, đo lường**.

## 3. Người dùng báo cáo và quyết định được hỗ trợ

| Người dùng giả định | Câu hỏi quyết định |
|---|---|
| Quản lý chuỗi cửa hàng | Nhóm cửa hàng nào cần được xem xét trước? |
| Quản lý khu vực/cửa hàng | Doanh số thấp đi cùng thiếu lượt khách hay doanh số trên mỗi khách thấp? |
| Nhóm marketing/bán hàng | Nên thử nghiệm điều chỉnh lịch hoặc cách triển khai khuyến mãi ở nhóm cửa hàng nào? |

## 4. Câu hỏi phân tích ưu tiên

1. Doanh số và lượt khách của hệ thống biến động theo tháng, quý, năm như thế nào?
2. Cửa hàng nào có doanh số trên ngày mở cửa thấp hơn **những cửa hàng cùng nhóm**? Chênh lệch đi cùng lượt khách hay doanh số/lượt khách?
3. Kết quả trong ngày có và không có khuyến mãi khác nhau thế nào trong từng nhóm cửa hàng, thứ trong tuần và tháng?
4. Có mô hình mùa vụ/ngày trong tuần nào đáng chú ý khi cân nhắc bố trí hoạt động bán hàng?
5. Với mỗi phát hiện, đề xuất hành động gì và dùng KPI nào theo dõi sau thử nghiệm?

## 5. Phạm vi dữ liệu và quy tắc chính

- **Dữ liệu nguồn:** `train.csv` (1,017,209 dòng; grain: Store × Date), `store.csv` (1,115 dòng; grain: Store). Không dùng `test.csv` do không có Sales thực tế.
- **Đơn vị phân tích chính:** 1 cửa hàng trong 1 ngày; `Store` + `Date` là khóa nghiệp vụ của bảng fact.
- **Thời gian:** 01/01/2013–31/07/2015. **2015 chưa đủ 12 tháng**: không so tổng cả năm 2015 với 2014. Nếu so YoY, dùng cùng kỳ có dữ liệu của cả hai năm và nêu rõ khoảng thời gian.
- **Ngày đóng cửa (`Open = 0`):** giữ để phân tích lịch hoạt động; loại khỏi mẫu số của KPI tính theo ngày mở cửa. Không tính doanh số bình quân bằng cách chia tùy tiện cho mọi ngày lịch.
- **`Promo`:** trạng thái khuyến mãi theo ngày tại `train.csv`; **`Promo2`:** cửa hàng tham gia chương trình khuyến mãi kéo dài/định kỳ trong `store.csv`. Không coi hai trường là đồng nghĩa.
- **Giá trị thiếu:** giữ null có ý nghĩa (vd. Promo2Since* của cửa hàng không tham gia). Gắn cờ các bản ghi `Open = 1`, `Sales = 0`/`Customers = 0`; không tự xóa.
- **Chỉ số `Customers`:** số khách được ghi nhận ở cấp ngày–cửa hàng; không coi là số khách hàng duy nhất qua nhiều ngày.

## 6. Dashboard và đầu ra mong đợi

| Trang | Câu hỏi nghiệp vụ | Nội dung chính |
|---|---|---|
| 1. Executive Overview | Hệ thống đang hoạt động ra sao? | Sales, Customers, Sales/Customer, xu hướng theo tháng; bộ lọc ngày, StoreType, Assortment |
| 2. Store Performance | Cửa hàng nào cần kiểm tra? | Sales/Open Store-Day, Customers/Open Store-Day, Sales/Customer, so sánh theo nhóm tương đồng |
| 3. Promotion Diagnostics | Ngày khuyến mãi gắn với thay đổi kết quả thế nào? | Promo/Non-Promo: số ngày mở, sales/ngày mở, customers/ngày mở; lọc cửa hàng và lịch |
| 4. Business Actions | Nên điều tra/thử nghiệm ở đâu? | Bảng Finding → Evidence → Proposed Action → KPI to Monitor; giới hạn và độ tin cậy |

**MVP nếu thiếu thời gian:** hoàn thiện 3 trang đầu + một trang kết luận/khuyến nghị đơn giản trước khi làm tính năng nâng cao.

## 7. Mẫu logic đưa ra hành động (chưa phải kết luận dữ liệu)

| Nếu quan sát được… | Kiểm tra thêm | Hành động thử nghiệm | KPI theo dõi |
|---|---|---|---|
| Sales/ngày mở của một cửa hàng thấp hơn nhóm tham chiếu | Customers/ngày mở và Sales/Customer | Nếu chủ yếu thiếu lượt khách: thử hoạt động thu hút khách ở phạm vi nhỏ | Customers/Open Store-Day, Sales/Open Store-Day |
| Lượt khách tương đương, Sales/Customer thấp | Loại cửa hàng, Assortment và giai đoạn | Rà soát cơ cấu hàng và thử gợi ý bán kèm; cần dữ liệu SKU để đánh giá cụ thể | Sales/Customer |
| Chênh lệch Promo–Non-Promo nhỏ ở một nhóm cửa hàng | Thứ trong tuần, tháng, các giai đoạn khuyến mãi | Thử điều chỉnh lịch hoặc nội dung khuyến mãi, ghi nhận chi phí bổ sung | Sales/Open Store-Day; Customers/Open Store-Day; chi phí nếu thu thập thêm |

Không phát biểu “khuyến mãi **gây ra** tăng trưởng” chỉ dựa vào chênh lệch quan sát. Không hứa hẹn một hành động chắc chắn nâng doanh số nếu chưa thử nghiệm.

## 8. Yêu cầu kỹ thuật cho bài BI

- **Power Query M:** chuẩn hóa ngày, kiểu dữ liệu, kiểm tra khóa, dữ liệu thiếu và bản ghi bất thường; dựng các bảng đầu ra.
- **Data model:** Star Schema với `FactDailyStoreSales`, `DimStore`, `DimDate`; có thể tạo dimension thêm khi có nhu cầu phân tích thực tế.
- **DAX:** KPI cơ bản + chỉ số trên ngày mở + so sánh cùng kỳ/phân nhóm + promo/non-promo.
- **Chất lượng:** kiểm tra khóa `Store`–`Date`, quan hệ từ fact sang dimension, tổng Sales/Customers trước và sau ETL; không nhân bản dòng khi ghép bảng.
- **Sản phẩm nộp:** file Power BI, dataset theo yêu cầu môn học, slides/PDF và diễn giải hành động.

## 9. Ngoài phạm vi phiên bản BI đầu tiên

- Dự báo Sales/ML, phân tích nhân quả khuyến mãi, Profit/ROI, giá vốn, tồn kho, dữ liệu SKU, các bộ nguồn/API cập nhật trực tiếp.
- Python, PostgreSQL, dbt, Airflow, Docker thuộc **giai đoạn Data Engineering sau khi nộp bài BI**, không phải điều kiện để hoàn thành môn.

## 10. Tiêu chí hoàn thành (acceptance criteria)

- [ ] Định nghĩa và kiểm chứng KPI trong `kpi-dictionary.md`.
- [ ] Data model không gây double-count khi lọc theo cửa hàng/ngày.
- [ ] Dashboard có filter nhất quán và xử lý ngày đóng cửa đúng.
- [ ] Có ít nhất 3 phát hiện từ **dữ liệu thực tế**, mỗi phát hiện gắn bằng chứng, hành động, KPI theo dõi và giới hạn suy luận.
- [ ] Slides trình bày được vấn đề → ETL/model → KPI → insight → action.
