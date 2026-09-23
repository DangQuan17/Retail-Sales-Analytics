# KPI Dictionary — Retail Sales Analytics

**Phiên bản:** 0.1 — business definitions, 23/09/2026  
**Grain:** `FactDailyStoreSales`: một `Store` × một `Date` (không trùng khóa).  
**Quy ước:** `Sales` là doanh số ghi nhận trong dataset; **không suy luận lợi nhuận**. `Customers` là lượt khách theo ngày–cửa hàng; **không phải khách duy nhất toàn kỳ**.

## 1. KPI lõi

| KPI | Định nghĩa nghiệp vụ / công thức khái niệm | Quy tắc & lưu ý |
|---|---|---|
| Total Sales | `SUM(Sales)` | Tính theo filter context, không gọi là profit/net profit |
| Total Customers | `SUM(Customers)` | Tổng lượt khách được ghi nhận, không phải distinct customers |
| Open Store-Days | Đếm các dòng `Open = 1` | Một cửa hàng mở một ngày = 1 store-day |
| Sales per Open Store-Day | `Sales ở Open=1 / Open Store-Days` | Không lấy mẫu số là mọi dòng/mọi ngày lịch |
| Customers per Open Store-Day | `Customers ở Open=1 / Open Store-Days` | Diễn giải là lượt khách bình quân trên ngày-cửa hàng mở |
| Sales per Customer | `Total Sales / Total Customers` | Tổng tỷ số có trọng số theo lượt khách; không lấy trung bình đơn giản của tỷ số từng dòng; chia 0 → BLANK |
| Promo Open Store-Days | Đếm dòng `Open=1 AND Promo=1` | Không nhầm `Promo` với `Promo2` |
| Non-Promo Open Store-Days | Đếm dòng `Open=1 AND Promo=0` | Dùng để đặt mẫu số so sánh hợp lệ |
| Promo Sales per Open Store-Day | `SUM(Sales | Open=1,Promo=1) / Promo Open Store-Days` | Mô tả kết quả quan sát trong ngày promo |
| Non-Promo Sales per Open Store-Day | `SUM(Sales | Open=1,Promo=0) / Non-Promo Open Store-Days` | Mô tả kết quả quan sát trong ngày không promo |
| Promo vs Non-Promo Difference (%) | `(Promo Sales/Open Day − Non-Promo Sales/Open Day) / Non-Promo Sales/Open Day` | Chỉ so sánh mô tả, không gọi “promotion uplift” theo nghĩa nhân quả; thiếu một nhóm → BLANK |

## 2. KPI phân tích chuyên sâu — ưu tiên sau khi hoàn thành lõi

| KPI | Định nghĩa / nguyên tắc | Cảnh báo |
|---|---|---|
| Sales Growth MoM (%) | `(Sales tháng hiện tại − Sales tháng trước) / Sales tháng trước` | Nếu tháng đang hiển thị chưa đủ dữ liệu, gắn nhãn partial period hoặc tránh so sánh |
| Sales Growth YoY (%) | `(Sales kỳ hiện tại − Sales đúng cùng kỳ năm trước) / Sales cùng kỳ năm trước` | 2015 chỉ có đến 31/07; so cùng khoảng ngày, không so 2015 YTD với toàn năm 2014 |
| Store Performance Gap (%) | `(Sales/Open Store-Day của cửa hàng − mức tham chiếu nhóm) / mức tham chiếu nhóm` | Định nghĩa peer: cùng `StoreType` và `Assortment`; tính trong cùng khoảng thời gian. Nên tính mức tham chiếu bằng trung bình của **KPI từng cửa hàng** để không để cửa hàng có nhiều dòng lấn át, và báo số cửa hàng của nhóm. |
| Customers Performance Gap (%) | Tương tự, dùng Customers/Open Store-Day | Hỗ trợ phân biệt vấn đề lượng khách với mức bán/khách |
| Promo vs Non-Promo Customers Difference (%) | So Customers/Open Store-Day của ngày promo và không promo | Đối chiếu trong cùng nhóm cửa hàng và các khoảng lịch tương đối tương đồng |

## 3. Thứ tự triển khai DAX

**MVP (nên hoàn thành trước):** Total Sales; Total Customers; Open Store-Days; Sales/Open Store-Day; Customers/Open Store-Day; Sales/Customer; Promo/Non-Promo Open Store-Days; Promo/Non-Promo Sales/Open Store-Day.

**Sau khi kiểm thử xong MVP:** Promo vs Non-Promo Difference (%); MoM/YoY; Store Performance Gap; Customers Performance Gap.

## 4. Quy ước về xử lý dữ liệu

1. Giữ tất cả dòng nguồn trong fact để đối chiếu; filter `Open=1` khi tính *per open store-day*.
2. Không loại mặc định những ngày `Open=1,Sales=0` hoặc `Customers=0`; gắn cờ chất lượng dữ liệu rồi điều tra.
3. `StateHoliday` có giá trị `'0'` lẫn mã chữ, nên không suy luận đây là numeric 0/1 thuần.
4. `Promo2SinceWeek`, `Promo2SinceYear`, `PromoInterval` để null cho cửa hàng không tham gia Promo2 là hợp lý; không coi số lượng null này là lỗi cần tự ý điền.
5. Trong Dashboard, báo rõ `data coverage: 2013-01-01 to 2015-07-31`; để ý bộ lọc thời gian và vùng không có dữ liệu.
6. Tránh sử dụng tên **ROI**, **profit margin**, **unique customer retention** vì nguồn không có thông tin cần thiết.

## 5. Kiểm thử tối thiểu trước khi công bố KPI

- [ ] `Total Sales` và `Total Customers` khớp tổng từ CSV, khi không áp filter.
- [ ] `Open Store-Days` bằng số dòng `Open=1` trong phạm vi lọc.
- [ ] `Promo Open Store-Days + Non-Promo Open Store-Days = Open Store-Days`.
- [ ] KPI per-day không bị thay đổi sai khi kéo `StoreType` / `Assortment`.
- [ ] Không chia cho 0, không có % bất thường do filter làm mất nhóm so sánh.
- [ ] Định nghĩa `Store Performance Gap` được diễn giải cùng peer group và kỳ thời gian.

**Ghi chú:** Đây là bản chốt *nghiệp vụ*. Khi dựng model, chúng ta sẽ viết công thức DAX thực tế và đối chiếu kết quả với dữ liệu nguồn.
