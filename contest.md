# THÔNG TIN TỔNG QUAN (META-INFORMATION)
- **Tên cuộc thi:** DATATHON 2026 - THE GRIDBREAKER [1].
- **Chủ đề:** Breaking Business Boundaries - Biến Dữ liệu thành Giải pháp cho Doanh nghiệp [1].
- **Đơn vị tổ chức:** VinTelligence (VinUniversity Data Science & AI Club) [1].
- **Bối cảnh (Domain):** Doanh nghiệp thương mại điện tử chuyên ngành thời trang tại Việt Nam [2, 3].
- **Thời gian chung kết:** Ngày 23/05/2026, tổ chức trực tiếp tại Đại học VinUni, Hà Nội [4, 5].

---

# PHẦN 1: TỪ ĐIỂN DỮ LIỆU (DATA DICTIONARY)
Bộ dữ liệu gồm 15 file CSV, mô phỏng giai đoạn từ 04/07/2012 đến 31/12/2022 (Train) và 01/01/2023 đến 01/07/2024 (Test) [2, 6]. Phân loại thành 4 nhóm chính:

## 1. Dữ liệu tham chiếu (Master Data)
Dữ liệu lưu trữ các thuộc tính cố định.
*   **products.csv:** `product_id` (PK), `product_name`, `category`, `segment`, `size`, `color`, `price`, `cogs`. Ràng buộc: `cogs < price` [7].
*   **customers.csv:** `customer_id` (PK), `zip` (FK -> geography.zip), `city`, `signup_date`, `gender`, `age_group`, `acquisition_channel` [7].
*   **promotions.csv:** `promo_id` (PK), `promo_name`, `promo_type` (percentage/fixed), `discount_value`, `start_date`, `end_date`, `applicable_category`, `promo_channel`, `stackable_flag`, `min_order_value` [8].
    *   *Công thức discount (percentage):* `quantity * unit_price * (discount_value/100)` [9].
    *   *Công thức discount (fixed):* `quantity * discount_value` [9].
*   **geography.csv:** `zip` (PK), `city`, `region`, `district` [9].

## 2. Dữ liệu giao dịch (Transaction Data)
*   **orders.csv:** `order_id` (PK), `order_date`, `customer_id` (FK), `zip` (FK), `order_status`, `payment_method`, `device_type`, `order_source` [9].
*   **order_items.csv:** `order_id` (FK), `product_id` (FK), `quantity`, `unit_price`, `discount_amount`, `promo_id` (FK), `promo_id_2` (FK) [10].
*   **payments.csv:** `order_id` (FK), `payment_method`, `payment_value`, `installments` [11].
*   **shipments.csv:** `order_id` (FK), `ship_date`, `delivery_date`, `shipping_fee` [11].
*   **returns.csv:** `return_id` (PK), `order_id` (FK), `product_id` (FK), `return_date`, `return_reason`, `return_quantity`, `refund_amount` [12].
*   **reviews.csv:** `review_id` (PK), `order_id` (FK), `product_id` (FK), `customer_id` (FK), `review_date`, `rating` (1-5), `review_title` [12].

## 3. Dữ liệu phân tích (Analytical Data)
*   **sales.csv (Train):** `Date`, `Revenue`, `COGS`. Từ 04/07/2012 – 31/12/2022 [13].
*   **sales_test.csv (Test - Ẩn):** Chỉ số đo lường mô hình, cấu trúc giống `sample_submission.csv` (Date, Revenue, COGS), giai đoạn 01/01/2023 – 01/07/2024 [6, 13-15].

## 4. Dữ liệu vận hành (Operational Data)
*   **inventory.csv:** `snapshot_date`, `product_id` (FK), `stock_on_hand`, `units_received`, `units_sold`, `stockout_days`, `days_of_supply`, `fill_rate`, `stockout_flag`, `overstock_flag`, `reorder_flag`, `sell_through_rate`, `product_name`, `category`, `segment`, `year`, `month` [16].
*   **web_traffic.csv:** `date`, `sessions`, `unique_visitors`, `page_views`, `bounce_rate`, `avg_session_duration_sec`, `traffic_source` [17].

## 5. Quy tắc quan hệ (Entity Relationships - Cardinality)
*   `orders` <-> `payments`: 1:1 [18].
*   `orders` <-> `shipments`: 1:0 hoặc 1 (áp dụng cho shipped/delivered/returned) [18].
*   `orders` <-> `returns`: 1:0 hoặc nhiều [18].
*   `orders` <-> `reviews`: 1:0 hoặc nhiều [18].
*   `order_items` <-> `promotions`: Nhiều:0 hoặc 1 [18].
*   `products` <-> `inventory`: 1:nhiều (theo tháng) [18].

---

# PHẦN 2: CẤU TRÚC ĐỀ THI & NHIỆM VỤ (COMPETITION TASKS)
Tổng điểm: 100 điểm, chia làm 3 phần [19].

## Task 1: Câu hỏi Trắc nghiệm (20 điểm - 2 điểm/câu) [19, 20]
Mục tiêu: Đánh giá khả năng truy vấn (SQL/Pandas) trực tiếp. 10 câu hỏi bao gồm:
1.  Tính trung vị inter-order gap của khách hàng mua > 1 đơn [21].
2.  Tìm `segment` có tỷ suất lợi nhuận gộp `(price - cogs)/price` cao nhất [21].
3.  Tìm `return_reason` phổ biến nhất của danh mục `Streetwear` [21, 22].
4.  Tìm `traffic_source` có `bounce_rate` trung bình thấp nhất [22].
5.  Tính % dòng trong `order_items` có `promo_id` != null [22].
6.  Tìm `age_group` có (tổng số đơn / số khách) cao nhất [23].
7.  Tìm `region` tạo ra `Revenue` cao nhất trong `sales_train` [23].
8.  Tìm `payment_method` phổ biến nhất với đơn `cancelled` [23].
9.  Tìm `size` (S, M, L, XL) có tỷ lệ trả hàng (returns/order_items) cao nhất [24].
10. Tìm số kỳ `installments` có trung bình `payment_value` cao nhất [24].

## Task 2: Phân tích và Trực quan hoá - EDA (60 Điểm) [19, 25]
Mục tiêu: Data Storytelling. Khám phá insight kinh doanh, yêu cầu gồm:
- **Trực quan hoá (Visualizations):** Đầy đủ tiêu đề, trục, chú thích [26].
- **Phân tích (Analysis):** Đánh giá theo 4 cấp độ tư duy:
    1.  *Descriptive (What happened?):* Thống kê đúng, biểu đồ chuẩn [27].
    2.  *Diagnostic (Why did it happen?):* Nhân quả, phân khúc, bất thường [27].
    3.  *Predictive (What is likely to happen?):* Xu hướng, tính mùa vụ [27].
    4.  *Prescriptive (What should we do?):* Đề xuất hành động kinh doanh định lượng [3]. (Làm được mức này nhất quán sẽ đạt điểm tối đa).

*Tiêu chí chấm Task 2:*
- Chất lượng trực quan hoá (15đ) [28, 29].
- Chiều sâu phân tích (25đ) [28, 29].
- Insight kinh doanh (15đ) [28, 30].
- Tính sáng tạo & kể chuyện (5đ) [28, 30].

## Task 3: Mô hình Dự báo Doanh thu - Sales Forecasting (20 Điểm) [3, 19]
Mục tiêu: Dự báo `Revenue` trong `sales_test.csv` (01/01/2023 – 01/07/2024) ở mức độ chi tiết [31].
- **Chỉ số đánh giá (Metrics):**
    - $MAE$: Sai số tuyệt đối trung bình (Càng thấp càng tốt) [14, 31].
    - $RMSE$: Căn bậc hai sai số toàn phương trung bình (Càng thấp càng tốt) [14, 31].
    - $R^2$: Hệ số xác định (Càng gần 1 càng tốt) [14, 31].
- **Tiêu chí chấm Task 3:**
    - Hiệu suất mô hình trên Kaggle Leaderboard (12đ) [32, 33].
    - Báo cáo kỹ thuật (8đ): Đánh giá pipeline, feature engineering, xử lý leakage, giải thích SHAP/feature importance bằng ngôn ngữ kinh doanh [32, 33].
- **Ràng buộc (Constraints) & Điều kiện loại trực tiếp:**
    - Không sử dụng bất kỳ dữ liệu ngoài [15, 34].
    - Không dùng Revenue/COGS từ tập test để làm features (Leakage) [34].
    - Phải có khả năng tái lập (Reproducibility), cung cấp mã nguồn, thiết lập random seeds [15, 34].

---

# PHẦN 3: HƯỚNG DẪN & CHECKLIST NỘP BÀI (SUBMISSION GUIDELINES)
Đội thi phải nộp 4 thành phần sau để được tính là hợp lệ:

1.  **Hệ thống Kaggle:**
    - Nộp file `submission.csv` chứa các cột `Date`, `Revenue`, `COGS` [14, 35].
    - Ràng buộc: Bắt buộc giữ đúng thứ tự dòng của `sample_submission.csv` (không xáo trộn) [14].
2.  **Báo cáo kỹ thuật (Report):**
    - Định dạng: PDF, sử dụng template LaTeX của NeurIPS 2025 [35].
    - Độ dài: Tối đa 4 trang (chưa tính Phụ lục và References) [35].
    - Nội dung: Tóm tắt phân tích EDA (Phần 2) và Pipeline/Kết quả mô hình (Phần 3) [35].
3.  **GitHub Repository:**
    - Trạng thái: Public hoặc chia sẻ quyền với Ban tổ chức trước Deadline [4].
    - Nội dung bắt buộc: Source code, Notebooks, `submission.csv`, file `README.md` hướng dẫn chạy code lại từ đầu [4].
    - Gắn link repo vào Báo cáo kỹ thuật [4].
4.  **Form Nộp bài chính thức:**
    - Điền 10 đáp án trắc nghiệm [4].
    - Cung cấp Link Kaggle, Link GitHub và File PDF Báo cáo [4].
    - Upload ảnh thẻ sinh viên của toàn bộ thành viên [4].
    - **TICKBOX QUAN TRỌNG:** Cam kết có ít nhất 1 thành viên tham dự trực tiếp Chung kết (23/05/2026 tại VinUni). Nếu không cam kết sẽ bị loại ngay lập tức [4, 5].