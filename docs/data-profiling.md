# Rossmann Store Sales — Data Profiling (Initial)

**Project:** Retail Store Performance & Promotion Optimization  
**Nguồn:** Hai tệp `train.csv` và `store.csv` do thành viên dự án tải từ Rossmann Store Sales trên Kaggle.  
**Phạm vi:** Chỉ tập dữ liệu huấn luyện (`train.csv`) và thông tin cửa hàng (`store.csv`); không bao gồm `test.csv`.

## 1. Cấu trúc và grain

| File | Số dòng | Số cột | Grain | Khóa cần kiểm tra |
|---|---:|---:|---|---|
| `train.csv` | 1,017,209 | 9 | Một cửa hàng trong một ngày | (`Store`, `Date`) |
| `store.csv` | 1,115 | 10 | Một cửa hàng | `Store` |

- Khoảng ngày trong `train.csv`: **2013-01-01 đến 2015-07-31**; có **942 ngày khác nhau**.
- `train.csv` có **1,115 mã cửa hàng** và tất cả đều tìm thấy trong `store.csv`.
- Không phát hiện khóa (`Store`, `Date`) trùng trong `train.csv`; không phát hiện `Store` trùng trong `store.csv`.
- `DayOfWeek` khớp với ngày tương ứng theo quy ước thứ Hai = 1, Chủ nhật = 7.
- Có cửa hàng xuất hiện trong ít ngày hơn cửa hàng khác (758 đến 942 bản ghi mỗi cửa hàng); không mặc định điền thêm bản ghi thiếu.

## 2. Các cột quan trọng

**`train.csv`:**
`Store`, `DayOfWeek`, `Date`, `Sales`, `Customers`, `Open`, `Promo`, `StateHoliday`, `SchoolHoliday`.

**`store.csv`:**
`Store`, `StoreType`, `Assortment`, `CompetitionDistance`, `CompetitionOpenSinceMonth`,
`CompetitionOpenSinceYear`, `Promo2`, `Promo2SinceWeek`, `Promo2SinceYear`, `PromoInterval`.

## 3. Data quality: kết quả đo được

| Hạng mục | Kết quả |
|---|---:|
| `train.csv`: ô trống theo cách đọc CSV | 0 |
| `store.csv`: thiếu `CompetitionDistance` | 3 |
| Thiếu `CompetitionOpenSinceMonth` / `CompetitionOpenSinceYear` | 354 / 354 |
| Thiếu `Promo2SinceWeek` / `Promo2SinceYear` / `PromoInterval` | 544 / 544 / 544 |
| `Promo2 = 0` | 544 cửa hàng |
| `Promo2 = 1` | 571 cửa hàng |
| `Open = 0` | 172,817 ngày-cửa hàng |
| `Open = 1` nhưng `Sales = 0` | 54 ngày-cửa hàng |
| `Open = 1` nhưng `Customers = 0` | 52 ngày-cửa hàng |
| `Open = 0` nhưng `Sales > 0` | 0 |
| `Sales` hoặc `Customers` âm | 0 |

**Diễn giải missing:**
- Cả 544 cửa hàng `Promo2 = 0` đều không có tuần/năm bắt đầu và chu kỳ `Promo2`: đây có thể là thiếu có chủ đích vì không tham gia chương trình, **không điền 0 hoặc tự suy đoán tháng bắt đầu**.
- 351 cửa hàng có `CompetitionDistance` nhưng không có thông tin thời điểm đối thủ mở cửa; 3 cửa hàng thiếu cả ba trường cạnh tranh. Giữ `null`/nhóm “Unknown” tùy báo cáo, không tùy ý gán ngày mở cửa.
- Cần kiểm tra 54 bản ghi `Open = 1` nhưng doanh số 0 trước khi quyết định cách xử lý. Không xóa mặc định.

## 4. Thăm dò sơ bộ: khuyến mãi (chỉ mô tả)

Chỉ tính **ngày-cửa hàng có `Open = 1`**:

| Trạng thái | Số ngày-cửa hàng | Sales bình quân / ngày-cửa hàng | Customers bình quân / ngày-cửa hàng |
|---|---:|---:|---:|
| `Promo = 0` | 467,496 | 5,929.41 | 696.86 |
| `Promo = 1` | 376,896 | 8,228.28 | 844.43 |

**Cảnh báo nghiệp vụ:** Đây chỉ là chênh lệch quan sát trong dữ liệu, **không phải tác động nhân quả của khuyến mãi**. Trước khi đề xuất hành động cần phân nhóm theo cửa hàng/loại cửa hàng, thứ trong tuần, tháng và thời điểm; dữ liệu không cung cấp đủ chi phí để tính ROI hoặc lợi nhuận.

## 5. Kế hoạch ETL trong Power Query M

1. Import hai CSV, đặt đúng kiểu dữ liệu và giữ bản raw query để đối chiếu.
2. Kiểm tra khóa và đối chiếu liên kết `Store`.
3. Chuẩn hóa `Date`, kiểm tra `DayOfWeek`, chuẩn hóa cờ `Open`, `Promo`, `Promo2`.
4. Phân loại missing theo ý nghĩa nghiệp vụ; không tự động loại bỏ các cửa hàng thiếu thời điểm đối thủ mở cửa.
5. Gắn cờ 54 trường hợp cửa hàng mở nhưng không có doanh số, đối chiếu số khách.
6. Xây `DimStore` từ `store.csv`, `DimDate` từ lịch, `FactDailyStoreSales` từ `train.csv`.
7. Kiểm thử số bản ghi, khóa, tổng `Sales` và `Customers` giữa nguồn và fact.

## 6. Các KPI đề xuất để xây dựng sau khi chốt định nghĩa

- Total Sales; Total Customers; Sales per Customer.
- Open Store-Days; Sales per Open Store-Day; Customers per Open Store-Day.
- Sales Growth theo kỳ phù hợp; Promo vs Non-Promo Sales per Open Store-Day.
- Store Performance Gap so với nhóm tham chiếu có cùng đặc điểm (định nghĩa nhóm trước khi triển khai).

**Bước tiếp theo:** Viết Business Requirements + KPI Dictionary trước khi vẽ dashboard.
