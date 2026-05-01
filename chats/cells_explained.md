Đây là giải thích từng cell:

---

## Cell 0 — Giới thiệu (Markdown)

Đây chỉ là đoạn văn mô tả mục tiêu. Không phải code, không chạy gì cả. Nó nói:
- Mục tiêu: Dự đoán Revenue và COGS cho giai đoạn 2023–2024
- Cách làm: Dùng "trung bình mùa vụ + xu hướng tăng trưởng" — cách đơn giản nhất

---

## Cell 2 — Nhập thư viện & cài đặt đường dẫn

```python
import pandas as pd          # Thư viện đọc và xử lý bảng dữ liệu (giống Excel)
import numpy as np            # Thư viện tính toán số học
import matplotlib.pyplot as plt  # Thư viện vẽ biểu đồ
import warnings
warnings.filterwarnings('ignore')  # Tắt cảnh báo cho gọn

DATA_DIR   = 'dataset/'
TRAIN_FILE = DATA_DIR + 'sales.csv'       # File dữ liệu lịch sử (2012-2022)
OUT_FILE   = DATA_DIR + 'submission.csv'  # File kết quả sẽ nộp
```

Tưởng tượng: Bạn mở Excel lên, rồi chỉ cho máy tính "file dữ liệu nằm ở đâu" và "kết quả lưu ở đâu".

**Lỗi ở đây:** Thiếu dòng `TEST_FILE = DATA_DIR + 'sample_submission.csv'`. Cell sau dùng `TEST_FILE` nhưng chưa được định nghĩa, nên sẽ bị lỗi khi chạy.

---

## Cell 4 — Đọc dữ liệu và xem thử

```python
train = pd.read_csv(TRAIN_FILE, parse_dates=['Date'])  # Đọc file sales.csv vào biến "train"
test  = pd.read_csv(TEST_FILE,  parse_dates=['Date'])   # Đọc file submission mẫu vào biến "test"
```

Tưởng tượng: Mở 2 file Excel lên. `train` là sổ ghi chép 10 năm. `test` là bảng trống 548 dòng mà bạn cần điền số vào.

`parse_dates=['Date']` nghĩa là "cột Date là ngày tháng, hãy hiểu nó là ngày chứ không phải chữ bình thường" — để sau này có thể lấy ra tháng, năm, ngày trong tuần...

Phần `print(...)` và `train.tail()` chỉ để in ra xem dữ liệu trông như thế nào.

---

## Cell 5 — Vẽ biểu đồ Revenue và COGS theo thời gian

```python
fig, axes = plt.subplots(2, 1, ...)   # Tạo 2 biểu đồ xếp chồng lên nhau
axes[0].plot(train['Date'], train['Revenue'], ...)  # Biểu đồ trên: Revenue
axes[1].plot(train['Date'], train['COGS'], ...)     # Biểu đồ dưới: COGS
```

Tưởng tượng: Vẽ 2 đường biểu đồ — trục ngang là ngày, trục dọc là số tiền. Giống bạn vẽ biểu đồ đường trong Excel. Mục đích là **nhìn bằng mắt** xem doanh thu có tăng theo năm không, có lên xuống theo mùa không.

---

## Cell 7 — Tạo thêm cột thời gian & tính tổng theo năm

```python
train['year']        = train['Date'].dt.year        # Lấy ra năm (2012, 2013, ...)
train['day_of_year'] = train['Date'].dt.dayofyear   # Ngày thứ mấy trong năm (1-365)
train['month']       = train['Date'].dt.month       # Tháng (1-12)
train['day']         = train['Date'].dt.day          # Ngày trong tháng (1-31)

annual = train.groupby('year')[['Revenue', 'COGS']].sum()  # Tính tổng Revenue mỗi năm
```

Tưởng tượng: Từ cột ngày "2022-03-15", bạn tách ra thành: năm = 2022, tháng = 3, ngày = 15. Rồi cộng tổng doanh thu từng năm để xem năm nào bán nhiều nhất.

---

## Cell 8 — Tính tốc độ tăng trưởng trung bình mỗi năm

```python
yoy_rev = full_years['Revenue'].pct_change()  # So sánh năm nay với năm trước: tăng/giảm bao nhiêu %
growth_rev = (1 + yoy_rev).prod() ** (1 / len(yoy_rev))  # Tính trung bình tăng trưởng
```

Tưởng tượng: Năm 2013 bán 100 triệu, năm 2014 bán 110 triệu → tăng 10%. Năm 2015 bán 120 triệu → tăng ~9%. Làm tương tự cho tất cả các năm, rồi tính **trung bình** mỗi năm tăng bao nhiêu %. Con số này dùng để đoán "năm 2023 sẽ tăng thêm bao nhiêu so với 2022".

---

## Cell 10 — Tạo "hồ sơ mùa vụ" (Seasonal Profile)

```python
# Chia doanh thu mỗi ngày cho trung bình năm đó → được tỷ lệ
train['rev_norm'] = train['Revenue'] / annual_means['Revenue']

# Tính trung bình tỷ lệ đó cho mỗi ngày (tháng, ngày) qua tất cả các năm
seasonal = train.groupby(['month', 'day'])[['rev_norm', 'cogs_norm']].mean()
```

Đây là phần quan trọng nhất. Tưởng tượng:

- Ngày 14/2 (Valentine) mỗi năm đều bán **gấp đôi** ngày thường → tỷ lệ = 2.0
- Ngày 3/1 mỗi năm đều bán **ít hơn** bình thường → tỷ lệ = 0.7
- Ngày bình thường → tỷ lệ ≈ 1.0

Bảng `seasonal` lưu tỷ lệ này cho tất cả 365 ngày trong năm. Nó cho biết **ngày nào trong năm bán nhiều, ngày nào bán ít**.

---

## Cell 12 — Dự đoán cho giai đoạn test

```python
test['Revenue_pred'] = base_rev * growth_rev**years_ahead * test['rev_norm']
```

Công thức dự đoán = **Mức cơ sở × Tăng trưởng × Mùa vụ**

Tưởng tượng: Bạn muốn đoán doanh thu ngày 14/2/2023:
1. **Mức cơ sở**: Trung bình mỗi ngày năm 2022 bán được 1 tỷ
2. **Tăng trưởng**: Mỗi năm tăng 5% → 2023 sẽ là 1 tỷ × 1.05 = 1.05 tỷ
3. **Mùa vụ**: Ngày Valentine bán gấp đôi → 1.05 tỷ × 2.0 = **2.1 tỷ**

---

## Cell 14 — Kiểm tra xem dự đoán có tốt không

Lấy dữ liệu 2021-2022 (mà mình đã biết đáp án), áp dụng cùng cách dự đoán, rồi so sánh "đoán" vs "thực tế". Tính MAPE (sai lệch trung bình bao nhiêu %) và vẽ biểu đồ so sánh.

Giống như bạn tự kiểm tra: "Nếu tôi dùng cách này đoán năm 2021, thì đoán sai bao nhiêu?"

---

## Cell 16 — Xuất file nộp bài

```python
submission.to_csv(OUT_FILE, index=False)  # Lưu kết quả ra file CSV
```

Lưu bảng dự đoán thành file `submission.csv` để nộp lên Kaggle. File này có 548 dòng, mỗi dòng là 1 ngày với Revenue và COGS mà bạn đã đoán.

---

## Tóm lại cách baseline hoạt động

Nó giống như bạn nói: *"Mỗi năm cửa hàng tăng trưởng 5%, và ngày Valentine luôn bán gấp đôi, ngày thường bán bình thường. Vậy năm tới cũng thế thôi."*

Đây là cách đơn giản nhất. Muốn điểm cao hơn, cần dùng các mô hình thông minh hơn — tôi sẽ giúp bạn ở bước tiếp theo. Bạn muốn tiếp tục chứ?