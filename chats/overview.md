Tất cả data đã được import vào dự án trong thư mục dataset, đừng cố gắng đọc nó vì data rất lớn, ngoài ra có có thêm @baseline.ipynb nữa, lưu ý rằng là tôi là một người mới, hoàn     
toàn không biết gì về các khái niệm trong AI và ML cũng như là python nên khi hướng dẫn, hãy làm như đang dạy cho một người không biết gì về lập trình

Tôi đã hiểu toàn bộ bức tranh rồi. Giờ để tôi giải thích mọi thứ cho bạn thật đơn giản.

---

## Bạn đang cần làm gì? (Giải thích như cho người chưa biết gì)

Hãy tưởng tượng thế này:

Bạn có một **cửa hàng thời trang online**. Mỗi ngày cửa hàng ghi lại 2 con số:
- **Revenue** = tổng tiền khách trả (doanh thu)
- **COGS** = tổng tiền bạn bỏ ra mua hàng (giá vốn)

Bạn đã có sổ ghi chép từ **2012 đến 2022** (3833 ngày) — đây là file `sales.csv`.

Bài thi yêu cầu bạn **đoán** Revenue và COGS cho **548 ngày tiếp theo** (2023-2024) — rồi ghi vào file `submission.csv` nộp lên Kaggle.

Giống như bạn nhìn doanh thu 10 năm qua rồi đoán "năm tới mỗi ngày bán được bao nhiêu?".

---

## File baseline.ipynb đã có sẵn làm gì?

File này dùng cách đơn giản nhất: **"Nhìn mấy năm trước, tháng này ngày này bán được bao nhiêu, thì năm sau chắc cũng tương tự, cộng thêm tí tăng trưởng"**. Tuy nhiên file này có 1 lỗi nhỏ (thiếu định nghĩa `TEST_FILE`).

---

## Bước tiếp theo

Tôi sẽ giúp bạn theo thứ tự:

1. **Sửa baseline cho chạy được** — để bạn có file submission nộp lên Kaggle trước
2. **Cải thiện mô hình** — dùng kỹ thuật tốt hơn để điểm cao hơn
3. **Hướng dẫn viết báo cáo** — 8 điểm cho phần report

Bạn muốn tôi bắt đầu sửa baseline và tạo notebook hoàn chỉnh chạy được luôn không?