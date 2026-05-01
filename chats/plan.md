Bài toán của bạn thuộc loại gì?

Bạn có dữ liệu theo ngày từ quá khứ, và cần đoán tương lai. Trong data science, đây gọi là Time Series Forecasting (dự báo chuỗi thời gian).
Giống như nhìn biểu đồ nhiệt độ 10 năm qua rồi đoán nhiệt độ năm tới.
---
3 cách tiếp cận phổ biến
Cách 1 — Phương pháp thống kê (baseline đang dùng)
Ý tưởng: "Nhìn quy luật cũ, áp dụng cho tương lai"
- Tháng 12 năm nào cũng bán nhiều → tháng 12 tới cũng vậy
- Mỗi năm tăng 5% → năm tới cũng tăng 5%
Ưu điểm: Đơn giản, dễ hiểu
Nhược điểm: Quá đơn giản, không xét được nhiều yếu tố
Cách 2 — Prophet (do Facebook/Meta tạo ra)
Ý tưởng: Giống cách 1 nhưng thông minh hơn. Nó tự động tìm ra:
- Xu hướng tăng/giảm dài hạn
- Quy luật theo mùa (mùa hè bán nhiều, mùa đông bán ít)
- Quy luật theo tuần (cuối tuần bán nhiều hơn ngày thường)
Ưu điểm: Rất dễ dùng, chỉ cần 3 dòng code
Nhược điểm: Không tận dụng được dữ liệu khác (web_traffic, promotions...)
Cách 3 — Machine Learning (XGBoost / LightGBM)
Ý tưởng: Thay vì chỉ nhìn ngày tháng, cho máy tính học từ nhiều yếu tố cùng lúc:
- Hôm nay là thứ mấy? Tháng mấy? Có khuyến mãi không?
- Tuần trước bán được bao nhiêu?
- Lượng truy cập web hôm nay thế nào?
Máy tính sẽ tự tìm ra "khi nào có khuyến mãi + cuối tuần + tháng 12 thì doanh thu tăng gấp 3".
Ưu điểm: Mạnh nhất, tận dụng được tất cả 15 file CSV
Nhược điểm: Phức tạp hơn, cần tạo features
---
Tôi khuyên bạn nên dùng gì?
Kết hợp cách 2 + cách 3, cụ thể:
1. Dùng Prophet làm mô hình đầu tiên — dễ, nhanh, cho kết quả khá tốt
2. Dùng XGBoost làm mô hình chính — tận dụng thêm dữ liệu từ web_traffic.csv, promotions.csv... để cải thiện điểm
3. Trộn kết quả 2 mô hình lại (ensemble) — thường cho điểm cao nhất
Lý do: Cuộc thi cho bạn 15 file CSV, không phải tự nhiên. Ban tổ chức muốn bạn tận dụng các dữ liệu khác để dự báo tốt hơn. Đội nào chỉ dùng mỗi sales.csv sẽ khó đạt điểm cao.
---
Bạn muốn tôi bắt đầu xây dựng mô hình nào trước? Tôi khuyên bắt đầu với Prophet vì nó đơn giản nhất — chạy được kết quả nộp Kaggle trước, rồi cải thiện sau.