Mình sẽ giải thích file [feature_ml.ipynb](/Users/hungpn23/PycharmProjects/Datathon2026/feature_ml.ipynb) theo đúng thứ tự cell. Vì notebook này khá dài, mình sẽ đi theo kiểu:

- Mỗi cell làm gì
- Từng dòng hoặc cụm dòng liền nhau đang làm gì
- Nếu có từ khóa AI/ML/DS, mình giải thích ngay lúc gặp

**Ý tưởng cực ngắn trước khi vào chi tiết**

Notebook này làm việc sau:

1. Đọc dữ liệu cũ 2012-2022.
2. Biến dữ liệu thô thành các “dấu hiệu” giúp đoán doanh thu.
3. Huấn luyện mô hình để học từ các dấu hiệu đó.
4. Kiểm tra xem mô hình đoán năm 2022 có ổn không.
5. Dùng toàn bộ lịch sử để đoán 2023-2024.
6. Xuất file `submission.csv`.

## Cell 0

Đây là cell markdown, tức là chỉ để viết giải thích, không chạy code.

- Dòng 1: tiêu đề notebook. `Feature-Based ML Forecasting` nghĩa là dự báo bằng machine learning có dùng các “features”.
- Dòng 3: nói rằng notebook này không làm kiểu đơn giản là kéo dài xu hướng từ `sales.csv`, mà dùng thêm nhiều tín hiệu khác.
- Dòng 6: `sales.csv` vẫn là dữ liệu mục tiêu.
  - `Revenue` = doanh thu
  - `COGS` = giá vốn
- Dòng 7: các bảng khác trong `dataset/` được gom lại thành tín hiệu theo ngày.
- Dòng 8: vì dữ liệu phụ chỉ có tới cuối 2022, nên notebook biến nó thành “profile theo lịch”.
  - Ví dụ: “ngày 1/1 thường thế nào”, “thứ Hai trong tháng 12 thường thế nào”.
  - Đây là cách dùng dữ liệu cũ để suy ra tương lai mà không nhìn lén test.
- Dòng 9: mô hình dùng `ridge regression` trên `log target`.
  - `ridge regression` là một dạng hồi quy tuyến tính có thêm cơ chế chống overfit.
  - `log target` nghĩa là thay vì học trực tiếp doanh thu, mô hình học `log(1 + doanh thu)` để dữ liệu bớt lệch.
- Dòng 10: dự báo theo kiểu `recursive walk-forward`.
  - Nghĩa là đoán từng ngày một, ngày sau có thể dùng kết quả dự đoán của ngày trước.
- Dòng 13: notebook có bước kiểm tra nhanh trên năm 2022.
- Dòng 14: cuối cùng tạo file nộp bài.

## Cell 1

Markdown, không chạy code.

- Dòng 1: tiêu đề phần import và config.
- Dòng 3-6: nói trước rằng cell sau chỉ có 3 việc:
  - import thư viện
  - khai báo đường dẫn file
  - đặt vài tham số chung

## Cell 2

Đây là cell code đầu tiên.

- Dòng 1: `from pathlib import Path`
  - Dùng `Path` để làm việc với đường dẫn file cho gọn và an toàn hơn.
- Dòng 3: `import warnings`
  - Thư viện xử lý cảnh báo.
- Dòng 5: `import matplotlib.pyplot as plt`
  - Thư viện vẽ biểu đồ.
- Dòng 6: `import numpy as np`
  - Thư viện tính toán số học nhanh.
- Dòng 7: `import pandas as pd`
  - Thư viện quan trọng nhất để xử lý bảng dữ liệu.

- Dòng 9-13:
  - Cố gắng import `display` từ Jupyter.
  - Nếu không import được, tự tạo hàm `display(obj)` bằng `print(obj)`.
  - Ý nghĩa: notebook linh hoạt hơn, chạy ở môi trường khác vẫn ít lỗi hơn.

- Dòng 15:
  - Tắt cảnh báo loại `PerformanceWarning` của pandas.
  - Cảnh báo này thường chỉ nói “code có thể chưa tối ưu”, không phải lỗi logic.

- Dòng 17:
  - Chọn style biểu đồ.
- Dòng 18:
  - Cho pandas hiện tối đa 200 cột khi in bảng.
- Dòng 19:
  - Định dạng số thực cho đẹp hơn, ví dụ `1234567.89` thành `1,234,567.89`.

- Dòng 21:
  - `DATA_DIR = Path('dataset')`
  - Nói rằng toàn bộ dữ liệu nằm trong thư mục `dataset`.
- Dòng 22:
  - `TRAIN_FILE` là file train `sales.csv`.
- Dòng 23:
  - `TEST_FILE` là `sample_submission.csv`.
  - Ở đây file này chủ yếu được dùng để lấy danh sách ngày cần dự báo.
- Dòng 24:
  - `OUT_FILE` là file kết quả xuất ra.

- Dòng 26:
  - `LAGS = [1, 7, 14, 28, 56, 91, 182, 364]`
  - `lag` nghĩa là “giá trị của mấy ngày trước”.
  - Ví dụ `lag 7` = doanh thu cách đây 7 ngày.
- Dòng 27:
  - `ROLL_WINDOWS` là các cửa sổ để tính trung bình trượt, độ lệch chuẩn trượt.
  - Ví dụ cửa sổ 28 ngày = nhìn 28 ngày gần nhất.
- Dòng 28:
  - `MARGIN_LAGS` là các lag cho tỷ suất lợi nhuận.
- Dòng 29:
  - `RIDGE_ALPHA = 50.0`
  - Đây là độ mạnh của phần “phạt” trong ridge regression.
  - Phạt mạnh hơn thì mô hình bớt học quá đà.

## Cell 3

Markdown.

- Dòng 1: tiêu đề phần load data.
- Dòng 4: chỉ đọc những cột cần thiết.
- Dòng 5: `sales.csv` là target train.
- Dòng 6: `sample_submission.csv` dùng để biết các ngày phải điền dự báo.

## Cell 4

Cell này tạo hàm đọc dữ liệu.

- Dòng 1: định nghĩa hàm `load_data(data_dir: Path)`.
  - Hàm này nhận thư mục dữ liệu và trả về tất cả bảng cần dùng.

- Dòng 2:
  - Đọc `sales.csv`.
  - `parse_dates=['Date']` bảo pandas hiểu cột `Date` là ngày tháng.
  - `.rename(columns={'Date': 'date'})` đổi tên `Date` thành `date` cho đồng nhất.
- Dòng 3:
  - Đọc `sample_submission.csv`.
  - Cũng đổi `Date` thành `date`.

- Dòng 5-9:
  - Đọc `orders.csv`.
  - `usecols=[...]` nghĩa là chỉ lấy các cột cần thiết.
  - Có thêm `zip` để sau này nối với `geography`.
- Dòng 10-13:
  - Đọc `order_items.csv`.
  - Đây là chi tiết từng món trong từng đơn.
- Dòng 14-17:
  - Đọc `products.csv`.
  - Chỉ lấy `product_id`, `category`, `cogs`.
- Dòng 18-21:
  - Đọc `payments.csv`.
- Dòng 22-26:
  - Đọc `shipments.csv`.
  - `ship_date`, `delivery_date` được parse thành ngày.
- Dòng 27-31:
  - Đọc `returns.csv`.
- Dòng 32-36:
  - Đọc `reviews.csv`.
- Dòng 37-41:
  - Đọc `customers.csv`.
- Dòng 42-46:
  - Đọc `promotions.csv`.
- Dòng 47-51:
  - Đọc `inventory.csv`.
- Dòng 52-56:
  - Đọc `web_traffic.csv`.
- Dòng 57-60:
  - Đọc `geography.csv`, chỉ lấy `zip` và `region`.

- Dòng 62-77:
  - Trả về một `dictionary`.
  - `dictionary` là một cấu trúc kiểu “tên -> dữ liệu”.
  - Ví dụ `data['sales']` là bảng sales, `data['orders']` là bảng orders.

- Dòng 80:
  - Gọi hàm `load_data(DATA_DIR)` và gán kết quả vào biến `data`.

## Cell 5

Cell này chỉ để xem nhanh dữ liệu đã load.

- Dòng 1:
  - Tạo danh sách rỗng `summary_rows`.
- Dòng 2:
  - `for name, df in data.items():`
  - Lặp qua từng bảng trong `data`.
  - `name` là tên bảng, `df` là dataframe của bảng đó.
- Dòng 3:
  - Tìm các cột có chữ `date` trong tên.
- Dòng 4-5:
  - Khởi tạo `date_min`, `date_max` là `None`.
  - `None` nghĩa là chưa có giá trị.
- Dòng 6:
  - Nếu bảng có cột ngày.
- Dòng 7:
  - Lấy cột ngày đầu tiên.
- Dòng 8:
  - Kiểm tra cột đó có thật sự là kiểu ngày không.
- Dòng 9-10:
  - Lấy ngày nhỏ nhất và lớn nhất.

- Dòng 11-18:
  - Thêm một dòng tóm tắt vào `summary_rows`.
  - Mỗi dòng gồm:
    - tên bảng
    - số dòng
    - số cột
    - tên cột ngày
    - ngày nhỏ nhất
    - ngày lớn nhất

- Dòng 20:
  - Biến danh sách tóm tắt thành dataframe.
- Dòng 21:
  - Hiển thị dataframe đó.

## Cell 6

Markdown.

- Dòng 3: `safe_div` là chia an toàn.
- Dòng 5-7: `build_auxiliary_profiles` gom dữ liệu phụ theo ngày rồi biến thành profile theo lịch.
- Dòng 9-11: `build_model_frame` tạo bảng feature cuối cùng.
- Dòng 13-15: `fit_ridge` và `recursive_forecast` dùng để train và forecast.

## Cell 7

Đây là cell quan trọng nhất. Nó chứa gần như toàn bộ “bộ máy” của notebook.

### Phần 1: `safe_div`

- Dòng 1: định nghĩa hàm chia an toàn.
- Dòng 2:
  - Ép `a` thành mảng số thực.
- Dòng 3:
  - Ép `b` thành mảng số thực.
- Dòng 4:
  - Tạo mảng kết quả ban đầu toàn số 0, cùng kích thước với `a`.
- Dòng 5:
  - Chỉ chia ở những nơi mà mẫu số `b` không quá gần 0.
  - `1e-9` là một số rất nhỏ.
- Dòng 6:
  - Trả kết quả.
- Ý nghĩa: tránh lỗi chia cho 0 hoặc ra vô cực.

### Phần 2: `build_auxiliary_profiles`

- Dòng 9:
  - Định nghĩa hàm xây profile phụ trợ.
- Dòng 10-22:
  - Lấy từng bảng từ `data_dict` ra thành biến riêng cho dễ đọc.

#### Nhóm order item + product

- Dòng 24:
  - Ghép `items` với `products` theo `product_id`.
  - Việc này giúp mỗi dòng item biết nó thuộc category nào, cogs bao nhiêu.
- Dòng 25:
  - Tạo `net_item_revenue`.
  - Công thức: `quantity * unit_price - discount_amount`.
  - Đây là doanh thu thuần của dòng hàng đó.
- Dòng 26-27:
  - Với mỗi category `Streetwear`, `Outdoor`, `Casual`,
  - tạo một cột số lượng của category đó.
  - Nếu dòng đó không thuộc category đang xét thì cho 0.

- Dòng 29-41:
  - Ghép `item_prod` với `orders` để biết item thuộc ngày nào.
  - `groupby('order_date')`: gom theo ngày đơn hàng.
  - `agg(...)`: tính các tổng theo ngày:
    - tổng số units
    - tổng discount
    - tổng doanh thu thuần
    - tổng số lượng từng category
  - Cuối cùng đổi tên `order_date` thành `date`.

- Dòng 42-43:
  - Tạo tỷ trọng category.
  - Ví dụ `streetwear_unit_share` = số món Streetwear / tổng số món ngày đó.

#### Nhóm order header

- Dòng 45-60:
  - Gom `orders` theo ngày.
  - Tính:
    - số đơn
    - số khách hàng duy nhất
    - số đơn returned
    - số đơn cancelled
    - số đơn mobile
    - số đơn credit card
    - số đơn từ paid search, organic, social, email
- Dòng 61-62:
  - Chuyển các con số ở trên thành tỷ lệ trên tổng số đơn.

#### Nhóm geography

- Dòng 64:
  - Ghép `orders` với `geography` qua `zip`.
- Dòng 65-66:
  - Tạo cột cờ cho từng `region`.
  - Nếu đơn thuộc `East` thì `East_orders = 1`, ngược lại là 0.
- Dòng 67-75:
  - Gom theo ngày và cộng lên.
  - Ta được số đơn ở East, Central, West mỗi ngày.
- Dòng 76:
  - Tính tổng số đơn vùng trong ngày.
- Dòng 77-79:
  - Đổi sang tỷ lệ vùng.
  - Ví dụ `east_region_share` = tỷ lệ đơn từ East.

#### Nhóm payments

- Dòng 81-86:
  - Ghép payment với orders để biết payment thuộc ngày nào.
  - Gom theo ngày.
  - Tính:
    - tổng `payment_value`
    - trung bình `installments`

#### Nhóm shipments

- Dòng 88:
  - `shipments.copy()` để tránh sửa thẳng bảng gốc.
- Dòng 89:
  - Tính số ngày giao hàng = `delivery_date - ship_date`.
- Dòng 90-95:
  - Gom shipment theo ngày order.
  - Tính tổng phí ship và thời gian giao trung bình.

#### Nhóm returns

- Dòng 97-107:
  - Gom returns theo `return_date`.
  - Tính:
    - số lượt trả
    - tổng số lượng trả
    - tổng tiền refund
    - số case `late_delivery`
    - số case `wrong_size`
- Dòng 108-109:
  - Tính tỷ lệ từng lý do trả hàng.

#### Nhóm reviews

- Dòng 111-119:
  - Gom review theo ngày review.
  - Tính:
    - số review
    - rating trung bình
    - số review rating thấp (`<= 2`)
- Dòng 120:
  - Tính tỷ lệ review xấu.

#### Nhóm customers signup

- Dòng 122-130:
  - Gom khách hàng theo ngày signup.
  - Tính:
    - số signup
    - số signup nữ
    - số signup đến từ social media
- Dòng 131-132:
  - Chuyển thành tỷ lệ.

#### Nhóm promotions

- Dòng 134:
  - Tạo list rỗng `promo_frames`.
- Dòng 135:
  - Lặp qua từng chương trình khuyến mãi.
- Dòng 136:
  - Tạo bảng gồm tất cả ngày mà promo đó đang hoạt động.
- Dòng 137-147:
  - Với mỗi ngày trong promo, gắn các cột mô tả promo:
    - có bao nhiêu promo active
    - discount bao nhiêu
    - là promo phần trăm hay fixed
    - có stackable không
    - online/email/social/all_channels không
    - áp dụng cho Streetwear hay Outdoor không
- Dòng 148:
  - Thêm bảng promo này vào `promo_frames`.
- Dòng 149:
  - Nối tất cả promo lại rồi gom theo ngày để ra tổng tác động promo mỗi ngày.

#### Nhóm inventory

- Dòng 151:
  - Tạo lịch ngày đầy đủ từ ngày đầu đến ngày cuối của sales.
- Dòng 152-167:
  - Gom inventory theo `snapshot_date`.
  - Tính tổng hoặc trung bình các chỉ số tồn kho.
- Dòng 168:
  - Ghép với lịch đầy đủ để ngày nào cũng có dòng.
- Dòng 169:
  - `ffill()` nghĩa là điền giá trị gần nhất phía trước xuống cho các ngày trống.
  - Vì inventory thường là snapshot theo kỳ, không phải ngày nào cũng có.

#### Nhóm web traffic

- Dòng 171-180:
  - Gom web traffic theo ngày.
  - Tính:
    - tổng sessions
    - unique visitors
    - page views
    - bounce rate trung bình
    - thời lượng phiên trung bình
- Dòng 181:
  - Tạo bảng pivot theo `traffic_source`.
  - Mục tiêu: biết từng nguồn đem bao nhiêu sessions.
- Dòng 182:
  - Đặt lại tên cột cho dễ hiểu.
- Dòng 183:
  - Ghép bảng tổng với bảng nguồn traffic.
- Dòng 184-187:
  - Với mỗi nguồn chính, tính tỷ lệ sessions của nguồn đó trên tổng sessions.

#### Gộp tất cả tín hiệu phụ

- Dòng 189:
  - Bắt đầu từ `calendar.copy()`.
- Dòng 190-191:
  - Lần lượt ghép tất cả bảng daily vào một bảng lớn tên `aux`.
- Dòng 192:
  - Sắp xếp theo ngày và điền `NaN` bằng 0.
- Dòng 193-195:
  - Tạo thêm cột tháng, ngày trong tháng, thứ trong tuần.

#### Tạo profile theo lịch

- Dòng 197:
  - Chọn các cột dùng làm profile, bỏ `date`, `month`, `day`, `dow`.
- Dòng 198:
  - Gom theo `(month, day)` và lấy trung bình.
  - Prefix `md_` nghĩa là month-day.
- Dòng 199:
  - Gom theo `(month, dow)` và lấy trung bình.
  - Prefix `mwd_` nghĩa là month-weekday.

#### Tạo profile từ sales

- Dòng 201:
  - Copy sales.
- Dòng 202-204:
  - Tạo `month`, `day`, `dow`.
- Dòng 205-207:
  - Tính doanh thu/COGS trung bình theo `(month, day)`.
- Dòng 208-210:
  - Tính doanh thu/COGS trung bình theo `(month, dow)`.

- Dòng 212:
  - Trả ra 4 bảng profile:
    - `md_profile`
    - `mwd_profile`
    - `sales_md`
    - `sales_mwd`

### Phần 3: `build_model_frame`

- Dòng 215:
  - Định nghĩa hàm dựng bảng feature cuối cùng.
- Dòng 216:
  - Lấy sales.
- Dòng 217:
  - Lấy danh sách ngày tương lai từ test.

- Dòng 219:
  - Nối ngày train và ngày future lại thành một bảng duy nhất tên `full`.
  - Loại trùng, sắp xếp, reset index.
- Dòng 220-228:
  - Tạo feature lịch:
    - tháng
    - ngày
    - thứ
    - ngày thứ mấy trong năm
    - tuần thứ mấy trong năm
    - quý
    - cuối tuần hay không
    - đầu tháng/cuối tháng hay không
- Dòng 229-231:
  - Tạo feature sin/cos theo chu kỳ.
  - Đây là cách giúp máy hiểu tính tuần hoàn.
  - Ví dụ ngày 31/12 và 1/1 thật ra gần nhau theo mùa, dù số ngày khác nhau rất nhiều.

- Dòng 233-239:
  - Ghép tất cả profile vào `full`.
  - Cuối cùng ghép thêm sales thật ở giai đoạn train.

- Dòng 241:
  - Lặp cho cả hai target: `Revenue`, `COGS`.
- Dòng 242:
  - Đổi tên target sang chữ thường để dùng làm prefix.
- Dòng 243:
  - Ép cột target thành float.
- Dòng 244-245:
  - Tạo các lag.
- Dòng 246-248:
  - Tạo rolling mean và rolling std.
  - `shift(1)` nghĩa là chỉ nhìn quá khứ, không được nhìn chính ngày hiện tại.

- Dòng 250:
  - Tạo `margin_ratio = (Revenue - COGS) / Revenue`.
  - Đây là tỷ suất lợi nhuận.
- Dòng 251-252:
  - Tạo lag của tỷ suất lợi nhuận.

- Dòng 254-259:
  - Bắt đầu list `feature_cols` bằng các feature lịch và profile sales.
- Dòng 260:
  - Thêm tất cả feature bắt đầu bằng `md_` hoặc `mwd_`.
- Dòng 261-264:
  - Thêm lag, rolling mean, rolling std cho revenue và cogs.
- Dòng 265:
  - Thêm lag của margin ratio.
- Dòng 267:
  - Trả ra bảng `full` và danh sách cột feature.

### Phần 4: `fit_ridge`

- Dòng 270:
  - Định nghĩa hàm train ridge regression.
- Dòng 271:
  - Tính trung bình của từng cột feature.
- Dòng 272:
  - Tính độ lệch chuẩn từng cột.
- Dòng 273:
  - Nếu độ lệch chuẩn quá nhỏ thì đặt bằng 1 để tránh chia cho 0.
- Dòng 274:
  - Chuẩn hóa feature: trừ trung bình, chia độ lệch chuẩn.
  - Đây gọi là standardization.
- Dòng 275:
  - Thêm cột toàn số 1 vào đầu ma trận.
  - Cột này giúp mô hình học “bias” hay “intercept”.
- Dòng 276:
  - Tạo ma trận phạt đơn vị.
- Dòng 277:
  - Không phạt intercept.
- Dòng 278:
  - Giải công thức ridge regression bằng đại số tuyến tính.
- Dòng 279:
  - Trả về tham số mô hình:
    - `mu`
    - `sigma`
    - `beta`

### Phần 5: `predict_ridge`

- Dòng 282:
  - Định nghĩa hàm dự đoán.
- Dòng 283:
  - Chuẩn hóa dữ liệu mới bằng `mu`, `sigma` học từ train.
- Dòng 284:
  - Thêm cột 1.
- Dòng 285:
  - Nhân ma trận để ra dự đoán.

### Phần 6: `prepare_train_matrix`

- Dòng 288:
  - Định nghĩa hàm chuẩn bị tập train.
- Dòng 289:
  - Chỉ lấy dữ liệu đến `train_end`.
  - Đồng thời bỏ các dòng chưa có `lag 364`.
  - Vì nếu chưa có đủ 364 ngày lịch sử, feature sẽ bị thiếu.
- Dòng 290:
  - Thay vô cực bằng `NaN`, rồi điền `NaN` bằng 0.
- Dòng 291:
  - Trả dataframe train.

### Phần 7: `recursive_forecast`

- Dòng 294:
  - Định nghĩa hàm forecast từng ngày một.
- Dòng 295:
  - Copy frame và lấy `date` làm index để truy cập theo ngày dễ hơn.
- Dòng 296:
  - Lặp từng ngày từ `start_date` đến `end_date`.
- Dòng 297:
  - Lặp qua 2 target: Revenue và COGS.
- Dòng 298:
  - Tạo prefix chữ thường.
- Dòng 299-300:
  - Cập nhật các lag của ngày hiện tại từ dữ liệu quá khứ.
- Dòng 301:
  - Lấy toàn bộ lịch sử trước ngày hiện tại.
- Dòng 302-305:
  - Tính rolling mean/std tại ngày hiện tại từ lịch sử trước đó.

- Dòng 307-310:
  - Tính lịch sử `margin_ratio`.
- Dòng 311-312:
  - Gán lag của margin ratio cho ngày hiện tại.

- Dòng 314:
  - Lấy đúng một dòng feature của ngày hiện tại.
  - Làm sạch `inf`, `NaN`, rồi đổi sang ma trận số.
- Dòng 315:
  - Dự đoán Revenue.
  - `predict_ridge(...)` ra giá trị trên thang log.
  - `np.expm1(...)` đổi ngược từ log về giá trị gốc.
  - `max(..., 0.0)` để tránh số âm.
- Dòng 316:
  - Làm tương tự cho COGS.
- Dòng 318:
  - Trả bảng forecast.

### Phần 8: `compute_metrics`

- Dòng 321:
  - Định nghĩa hàm tính chỉ số đánh giá.
- Dòng 322-323:
  - Ép `y_true`, `y_pred` thành mảng số.
- Dòng 324:
  - Tính `MAE`.
- Dòng 325:
  - Tính `RMSE`.
- Dòng 326:
  - Tính `R²`.
- Dòng 327:
  - Trả kết quả.

### Phần 9: `extract_top_coefficients`

- Dòng 330:
  - Định nghĩa hàm lấy feature quan trọng nhất.
- Dòng 331:
  - Lấy trị tuyệt đối của hệ số `beta`, bỏ intercept ở đầu.
- Dòng 332:
  - Sắp xếp giảm dần và lấy top `n`.

## Cell 8

Markdown.

- Dòng 1: tiêu đề phần build feature frame.
- Dòng 3: nhắc rằng tới đây chưa train mô hình, mới chỉ tạo feature.

## Cell 9

- Dòng 1:
  - Tạo các profile phụ trợ từ dữ liệu raw.
- Dòng 2:
  - Dùng các profile đó để dựng bảng feature cuối cùng.
- Dòng 4:
  - In kích thước bảng.
- Dòng 5:
  - In số lượng feature.
- Dòng 6:
  - Hiển thị vài cột đầu để kiểm tra.

## Cell 10

Markdown.

- Dòng 3-6: giải thích cách validate đúng cho time series.
- Train tới 2021, dự đoán 2022, so với thật.

## Cell 11

- Dòng 1:
  - Chuẩn bị tập train tới hết 2021.
- Dòng 3-6:
  - Train model doanh thu.
  - `train_2021[feature_cols].to_numpy(float)` là ma trận đầu vào `X`.
  - `np.log1p(train_2021['Revenue'])` là nhãn `y` trên thang log.
- Dòng 7-10:
  - Train model COGS tương tự.

- Dòng 12-19:
  - Chạy forecast recursive cho năm 2022.
- Dòng 21:
  - Lấy dữ liệu thật 2022 từ `frame`.
- Dòng 22:
  - Lấy dự đoán 2022 từ `valid_forecast`.
- Dòng 23:
  - Ghép 2 bảng theo ngày để so sánh.

- Dòng 25-28:
  - Tính metrics cho Revenue và COGS.
  - `**compute_metrics(...)` nghĩa là bung dictionary metrics ra thành các cột.
- Dòng 29:
  - Hiển thị bảng metrics.

## Cell 12

Cell này vẽ biểu đồ.

- Dòng 1:
  - Tạo hình gồm 2 đồ thị xếp dọc.
- Dòng 3:
  - Vẽ đường Revenue thật.
- Dòng 4:
  - Vẽ đường Revenue dự đoán.
- Dòng 5:
  - Đặt tiêu đề.
- Dòng 6:
  - Hiện chú thích.

- Dòng 8:
  - Vẽ COGS thật.
- Dòng 9:
  - Vẽ COGS dự đoán.
- Dòng 10:
  - Đặt tiêu đề.
- Dòng 11:
  - Hiện chú thích.

- Dòng 13:
  - Sắp xếp layout cho gọn.
- Dòng 14:
  - Hiển thị biểu đồ.

## Cell 13

Markdown.

- Dòng 1: tiêu đề phần feature importance.
- Dòng 3: vì ridge regression đã chuẩn hóa feature, độ lớn hệ số có thể dùng để xem mô hình đang dựa vào tín hiệu nào nhiều hơn.

## Cell 14

- Dòng 1:
  - Lấy top 15 feature quan trọng nhất cho Revenue.
- Dòng 2:
  - Lấy top 15 feature quan trọng nhất cho COGS.

- Dòng 4:
  - Tạo hình gồm 2 đồ thị cạnh nhau.
- Dòng 6:
  - Vẽ bar chart ngang cho Revenue.
- Dòng 7:
  - Đặt tiêu đề.
- Dòng 8:
  - Đặt tên trục x.

- Dòng 10:
  - Vẽ bar chart ngang cho COGS.
- Dòng 11:
  - Đặt tiêu đề.
- Dòng 12:
  - Đặt tên trục x.

- Dòng 14-15:
  - Sắp xếp layout và hiển thị.

- Dòng 17:
  - Hiển thị bảng top feature Revenue.
- Dòng 18:
  - Hiển thị bảng top feature COGS.

## Cell 15

Markdown.

- Dòng 1: tiêu đề phần train cuối và forecast test.
- Dòng 3: sau khi validate xong, train lại bằng toàn bộ lịch sử 2012-2022 để dự đoán giai đoạn cần nộp.

## Cell 16

- Dòng 1:
  - Chuẩn bị tập train đầy đủ tới cuối 2022.
- Dòng 3-6:
  - Train model Revenue trên full history.
- Dòng 7-10:
  - Train model COGS trên full history.

- Dòng 12-19:
  - Dự đoán recursive cho giai đoạn 2023-01-01 đến 2024-07-01.
- Dòng 21-25:
  - Ghép kết quả dự đoán vào danh sách ngày của test.
  - `how='left'` nghĩa là giữ toàn bộ ngày từ test.
- Dòng 26:
  - Đổi `date` về `Date` cho đúng format nộp bài.
- Dòng 27:
  - Ghi ra file CSV.
- Dòng 29:
  - In nơi lưu file.
- Dòng 30:
  - In kích thước file submission.
- Dòng 31:
  - Hiển thị 10 dòng đầu.

## Cell 17

- Dòng 1:
  - Hiển thị 10 dòng cuối của submission.
  - Mục đích: kiểm tra cuối file có đủ dữ liệu không.

## Cell 18

Markdown tổng kết.

- Dòng 3:
  - Khẳng định notebook không dùng `Revenue/COGS` của test để tạo feature.
  - Đây là chống leakage.
- Dòng 4:
  - Liệt kê các bảng phụ đã được tận dụng.
  - Nhưng không dùng trực tiếp “giá trị tương lai thật”, mà biến thành profile lịch sử.
- Dòng 5-7:
  - Gợi ý hướng cải thiện:
    - thay ridge bằng mô hình mạnh hơn như boosting
    - hoặc trộn nhiều mô hình lại với nhau

## Hiểu các từ khóa chính trong notebook

- `feature`: một tín hiệu đầu vào cho mô hình. Ví dụ: hôm nay là thứ mấy, promo có active không, doanh thu 7 ngày trước là bao nhiêu.
- `target`: thứ mình muốn dự đoán. Ở đây là `Revenue` và `COGS`.
- `lag`: giá trị quá khứ. Ví dụ `Revenue` của 7 ngày trước.
- `rolling mean`: trung bình trượt. Ví dụ trung bình doanh thu 28 ngày gần nhất.
- `profile theo lịch`: “mẫu hành vi trung bình” theo ngày/thứ/tháng từ lịch sử.
- `ridge regression`: mô hình hồi quy tuyến tính có phạt để tránh học quá đà.
- `log1p`: biến đổi log của `1 + x`, giúp dữ liệu dễ học hơn khi giá trị chênh nhau lớn.
- `walk-forward`: dự đoán từng bước theo thời gian, không nhảy cóc.
- `leakage`: nhìn lén thông tin tương lai khi train/predict. Đây là điều phải tránh.

Nếu bạn muốn, ở tin nhắn tiếp theo mình có thể làm tiếp một bản **“dịch notebook này sang tiếng Việt đời thường”**, tức là viết lại toàn bộ luồng bằng ngôn ngữ cực dễ hiểu kiểu:

- “đoạn này giống mở Excel”
- “đoạn này giống cộng doanh thu từng ngày”
- “đoạn này giống nhìn 1 tuần trước để đoán hôm nay”

để bạn hiểu sâu hơn mà không cần biết nhiều Python.