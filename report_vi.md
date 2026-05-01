# Báo Cáo Kỹ Thuật Task 3: Dự Báo Doanh Thu

## 1. Tóm Tắt Tổng Quan

Báo cáo này mô tả pipeline dự báo cho Task 3, trong đó mục tiêu là dự đoán `Revenue` và `COGS` theo ngày cho giai đoạn test ẩn từ `2023-01-01` đến `2024-07-01`. Lời giải cuối cùng được triển khai trong `feature_ml.ipynb` và tạo ra file `dataset/submission.csv` theo đúng định dạng yêu cầu: `Date`, `Revenue`, và `COGS`.

Thay vì chỉ dựa vào bảng doanh thu lịch sử theo ngày, chúng tôi xây dựng một mô hình chuỗi thời gian dựa trên feature, tận dụng bối cảnh kinh doanh nội bộ có sẵn trong bộ dữ liệu. Mô hình kết hợp các tín hiệu về lịch sử bán hàng, mùa vụ theo lịch, hành vi đơn hàng, cơ cấu sản phẩm, khuyến mãi, lưu lượng truy cập web, tồn kho, trả hàng, đánh giá, thanh toán, giao hàng, kênh thu hút khách hàng và khu vực địa lý. Các tín hiệu này được chuyển thành những profile lịch sử an toàn về mặt leakage để có thể sử dụng cho các ngày tương lai, khi dữ liệu giao dịch thực tế chưa tồn tại.

Mô hình được đánh giá bằng phương pháp walk-forward: train trên dữ liệu đến `2021-12-31`, dự báo từng ngày trong năm 2022 theo kiểu recursive, rồi so sánh với giá trị thực tế của năm 2022. Cách đánh giá này mô phỏng sát hơn bài toán dự báo thực tế so với việc chia train/test ngẫu nhiên.

## 2. Tổng Quan Pipeline

Pipeline được thiết kế theo một chuỗi bước có thể tái lập:

1. Đọc dữ liệu nội bộ của cuộc thi từ thư mục `dataset/`.
2. Tổng hợp các bảng giao dịch, vận hành, khách hàng, khuyến mãi, traffic và địa lý thành tín hiệu ở cấp ngày.
3. Chuyển các tín hiệu không có sẵn trong tương lai thành profile lịch sử theo lịch.
4. Tạo lag features và rolling-window features từ lịch sử `Revenue` và `COGS`.
5. Train hai mô hình Ridge Regression riêng biệt: một mô hình cho `Revenue`, một mô hình cho `COGS`.
6. Đánh giá mô hình bằng walk-forward forecasting trên năm 2022.
7. Train lại mô hình trên toàn bộ lịch sử 2012-2022.
8. Dự báo giai đoạn test ẩn theo kiểu recursive và xuất `submission.csv`.

Thiết kế này tách riêng bước tạo feature và bước train mô hình. Điều đó giúp pipeline dễ kiểm tra, dễ chạy lại và giảm rủi ro vô tình gây leakage từ tập test.

## 3. Nguồn Dữ Liệu Được Sử Dụng

Mô hình sử dụng tất cả các file nội bộ có liên quan, ngoại trừ đáp án ẩn của tập test:

| Nguồn dữ liệu | Vai trò trong mô hình |
|---|---|
| `sales.csv` | Lịch sử target cho `Revenue` và `COGS`; nguồn để tạo lag và rolling features |
| `orders.csv` | Số lượng đơn hàng, cơ cấu trạng thái đơn, thiết bị, phương thức thanh toán, nguồn đơn hàng |
| `order_items.csv` | Số lượng sản phẩm bán ra, discount, cơ cấu sản phẩm, proxy doanh thu ở cấp giao dịch |
| `products.csv` | Danh mục sản phẩm và thông tin COGS ở cấp sản phẩm |
| `payments.csv` | Giá trị thanh toán và hành vi trả góp |
| `shipments.csv` | Phí vận chuyển và thời gian giao hàng |
| `returns.csv` | Số lượng trả hàng, refund amount và cơ cấu lý do trả hàng |
| `reviews.csv` | Số lượng review, rating trung bình và tỷ lệ rating thấp |
| `customers.csv` | Số lượng khách đăng ký và profile kênh thu hút khách hàng |
| `promotions.csv` | Số lượng khuyến mãi đang hoạt động, loại khuyến mãi, giá trị giảm giá, kênh khuyến mãi |
| `inventory.csv` | Tồn kho, stockout, reorder, fill rate, sell-through |
| `web_traffic.csv` | Sessions, visitors, page views, bounce rate, cơ cấu nguồn traffic |
| `geography.csv` | Phân bổ đơn hàng theo khu vực `East`, `Central`, và `West` |
| `sample_submission.csv` | Danh sách ngày test và thứ tự dòng bắt buộc khi nộp bài |

Không sử dụng bất kỳ dữ liệu ngoài nào.

## 4. Feature Engineering

Feature engineering là phần quan trọng nhất của lời giải này. Mô hình không chỉ hỏi: "Doanh thu năm ngoái là bao nhiêu?" Mô hình còn khai thác các câu hỏi có ý nghĩa kinh doanh như:

- Ngày này thường là mùa cao điểm hay thấp điểm?
- Ngày này có gần với một mẫu hình theo tuần hoặc theo năm không?
- Xu hướng nhu cầu gần đây đang như thế nào?
- Những ngày tương tự trong lịch sử có gắn với traffic web cao hơn không?
- Những ngày tương tự có thường có nhiều khuyến mãi không?
- Tồn kho và stockout trong lịch sử có ảnh hưởng đến doanh số không?
- Trả hàng, refund, rating và giao hàng có cho thấy vấn đề trong trải nghiệm khách hàng không?
- Nhu cầu trong lịch sử có khác nhau theo khu vực hoặc cơ cấu danh mục sản phẩm không?

Các nhóm feature quan trọng nhất gồm:

| Nhóm feature | Ví dụ feature | Ý nghĩa kinh doanh |
|---|---|---|
| Calendar features | `month`, `dow`, `is_weekend`, `year_sin`, `year_cos` | Bắt mùa vụ, cuối tuần và chu kỳ mua sắm trong năm |
| Lag features | `revenue_lag_7`, `revenue_lag_364`, `cogs_lag_364` | Bắt hành vi lặp lại theo tuần và nhu cầu cùng mùa năm trước |
| Rolling features | `revenue_roll_mean_28`, `cogs_roll_mean_364`, `revenue_roll_std_364` | Bắt xu hướng gần đây và độ biến động nhu cầu |
| Sales calendar profiles | `sales_md_revenue_mean`, `sales_mwd_cogs_mean` | Bắt hành vi trung bình lịch sử theo cùng ngày trong năm hoặc cùng mẫu thứ trong tháng |
| Order profiles | `md_order_count`, `mwd_customer_count`, `md_mobile_share` | Bắt nhu cầu và hành vi khách hàng thông thường ở các ngày tương tự |
| Promotion profiles | `md_active_promo_count`, `md_promo_discount_value` | Bắt mức độ khuyến mãi kỳ vọng ở các ngày tương tự trong lịch sử |
| Inventory profiles | `md_stock_on_hand`, `mwd_stockout_days`, `md_fill_rate` | Bắt việc tồn kho có hỗ trợ hay giới hạn khả năng bán hàng không |
| Web traffic profiles | `md_sessions`, `mwd_page_views`, `md_bounce_rate` | Bắt nhu cầu online kỳ vọng và chất lượng traffic |
| Return and review profiles | `md_refund_amount`, `md_return_count`, `md_rating_mean` | Bắt mức độ hài lòng và vấn đề sau mua hàng |
| Geography profiles | `md_east_region_share`, `md_west_region_share` | Bắt cơ cấu nhu cầu theo khu vực |

Với các ngày tương lai, ta không biết trước đơn hàng, traffic, returns hay inventory thực tế. Vì vậy, mô hình không dùng trực tiếp các giá trị quan sát trong tương lai từ những bảng này. Thay vào đó, mô hình sử dụng các profile lịch sử như:

- hành vi trung bình cho cùng `month-day`, ví dụ mẫu lịch sử của ngày 1 tháng 1;
- hành vi trung bình cho cùng `month-weekday`, ví dụ các ngày thứ Hai trong tháng 1.

Cách này giúp mô hình có thêm bối cảnh kinh doanh mà không làm rò rỉ thông tin tương lai.

## 5. Lựa Chọn Mô Hình

Notebook sử dụng Ridge Regression trên target đã được biến đổi log:

```text
target used by model = log(1 + Revenue)
target used by model = log(1 + COGS)
```

Ridge Regression được chọn vì ba lý do:

1. Ổn định khi có nhiều feature tương quan với nhau, ví dụ nhiều biến lag và rolling-window.
2. Có thể diễn giải thông qua hệ số đã chuẩn hóa.
3. Nhẹ, dễ tái lập và chỉ cần `numpy`, `pandas`, `matplotlib`.

Biến đổi log hữu ích vì giá trị doanh thu có thể chênh lệch lớn giữa ngày thường, giai đoạn nhu cầu cao và giai đoạn nhiều khuyến mãi. Việc học trên log target giúp giảm ảnh hưởng của các spike quá lớn và làm bài toán regression ổn định hơn. Sau khi dự đoán, kết quả được chuyển ngược về thang giá trị gốc bằng `expm1`.

Hai mô hình được train độc lập:

- một mô hình cho `Revenue`;
- một mô hình cho `COGS`.

Cách này hợp lý vì doanh thu và giá vốn có liên quan nhưng không hoàn toàn giống nhau. `Revenue` chịu ảnh hưởng bởi giá bán, discount, nhu cầu và traffic; trong khi `COGS` chịu ảnh hưởng mạnh bởi cơ cấu sản phẩm và số lượng bán ra.

## 6. Chiến Lược Validation

Chiến lược validation là walk-forward forecasting:

```text
Train period: data available up to 2021-12-31
Validation period: 2022-01-01 to 2022-12-31
Forecast style: recursive, one day at a time
```

Cách đánh giá này phù hợp hơn chia ngẫu nhiên vì bài toán thật cũng là dự báo tương lai. Nếu chia ngẫu nhiên, mô hình có thể được train trên các ngày sau và validate trên các ngày trước, điều đó không phản ánh đúng bài toán Kaggle.

Kết quả validation cục bộ trên năm 2022:

| Target | MAE | RMSE | R2 |
|---|---:|---:|---:|
| `Revenue` | 603,521 | 808,695 | 0.7666 |
| `COGS` | 572,348 | 756,149 | 0.7312 |

Diễn giải:

- MAE là sai số tuyệt đối trung bình theo ngày.
- RMSE phạt nặng hơn các lỗi dự đoán lớn so với MAE.
- R2 đo mức độ mô hình giải thích được biến động dữ liệu so với cách dự đoán đơn giản bằng giá trị trung bình.

Giá trị R2 cho thấy mô hình nắm bắt được một phần đáng kể biến động hằng ngày của cả `Revenue` và `COGS`. Vì validation được thực hiện trên cả một năm tương lai, kết quả này là một ước lượng hợp lý cho khả năng dự báo ngoài mẫu.

## 7. Phòng Tránh Leakage

Leakage là một trong những rủi ro quan trọng nhất của Task 3. Leakage xảy ra khi mô hình vô tình nhìn thấy thông tin mà tại thời điểm dự báo thực tế chưa thể biết.

Pipeline phòng tránh leakage bằng các cách sau:

1. Không bao giờ dùng `Revenue` và `COGS` của tập test làm feature.
2. Tất cả lag và rolling features tạo từ target đều được shift để mỗi dự báo chỉ nhìn thấy dữ liệu quá khứ.
3. Rolling statistics sử dụng `shift(1)` trước khi tính rolling mean hoặc standard deviation.
4. Không dùng trực tiếp dữ liệu giao dịch, traffic, promotion, inventory, return và review thực tế của tương lai.
5. Các nguồn dữ liệu không có sẵn trong tương lai được chuyển thành profile lịch sử theo lịch.
6. Validation được thực hiện theo thứ tự thời gian, không chia ngẫu nhiên.
7. Forecast cuối cùng là recursive: khi mô hình bước vào giai đoạn tương lai, nó chỉ dùng giá trị quá khứ thật hoặc các dự đoán đã sinh ra trước đó.

Ví dụ, một rolling feature đúng là:

```text
mean Revenue from the previous 28 days
```

không phải:

```text
mean Revenue from a window that includes the current day
```

Điểm khác biệt này rất quan trọng vì nếu cửa sổ rolling chứa cả ngày hiện tại, mô hình đã gián tiếp nhìn thấy đáp án mà nó cần dự đoán.

## 8. Feature Importance Và Diễn Giải Kinh Doanh

Vì mô hình là Ridge Regression với feature đã được chuẩn hóa, feature importance được diễn giải bằng trị tuyệt đối của standardized coefficients. Hệ số càng lớn thì mô hình càng phụ thuộc nhiều vào feature đó.

Các tín hiệu quan trọng nhất trong validation cục bộ gồm:

| Feature quan trọng | Diễn giải kinh doanh |
|---|---|
| `cogs_roll_mean_364` | Mẫu giá vốn dài hạn theo năm có sức dự báo cao, cho thấy nhu cầu sản phẩm và cơ cấu sản phẩm có tính mùa vụ mạnh |
| `revenue_roll_mean_364` | Doanh thu có mùa vụ theo năm rõ rệt; các giai đoạn tương tự trong năm trước rất hữu ích |
| `day` và yearly sine/cosine features | Vị trí trong lịch có ý nghĩa, phù hợp với nhu cầu thời trang theo mùa |
| `revenue_lag_1` và `cogs_lag_1` | Kết quả ngày hôm qua chứa thông tin quan trọng về momentum ngắn hạn |
| `cogs_roll_mean_182` và `revenue_roll_mean_91` | Xu hướng doanh thu và giá vốn trung hạn giúp ước lượng mức nhu cầu tương lai |
| `md_outdoor_units` | Cơ cấu danh mục sản phẩm trong lịch sử ảnh hưởng đến revenue và COGS kỳ vọng |
| `md_refund_amount` | Những giai đoạn có refund cao có thể phản ánh vấn đề trải nghiệm khách hàng hoặc vận hành |
| `md_payment_value_proxy` và `md_payment_value` | Hành vi thanh toán gắn chặt với mức nhu cầu |
| `md_stock_on_hand`, `mwd_stock_on_hand`, `md_fill_rate` | Mức độ sẵn có của tồn kho ảnh hưởng đến khả năng chuyển nhu cầu thành doanh thu |
| `margin_ratio_lag_1` | Biên lợi nhuận gần đây giúp ước lượng quan hệ giữa revenue và cost |

Ở góc nhìn kinh doanh, mô hình cho thấy nhu cầu thương mại điện tử thời trang được thúc đẩy bởi bốn nhóm yếu tố chính:

1. Seasonality: doanh số lặp lại theo tuần, tháng và năm.
2. Momentum: nhu cầu gần đây ảnh hưởng mạnh đến nhu cầu trong ngắn hạn.
3. Merchandising and inventory: mức độ sẵn có của hàng hóa và cơ cấu danh mục ảnh hưởng đến cả `Revenue` và `COGS`.
4. Customer and operational friction: refund, return, review và giao hàng cung cấp tín hiệu về chất lượng nhu cầu và trải nghiệm khách hàng.

Phần diễn giải này hữu ích vì nó không coi mô hình là một hộp đen. Nó liên kết hành vi của mô hình với các quyết định kinh doanh cụ thể:

- chuẩn bị tồn kho sớm hơn cho các giai đoạn có nhu cầu cao trong lịch sử;
- theo dõi các giai đoạn refund cao vì chúng có thể làm giảm chất lượng doanh thu thực tế;
- phối hợp khuyến mãi và chiến dịch traffic với các đỉnh mùa vụ;
- theo dõi cơ cấu danh mục sản phẩm vì `Revenue` và `COGS` thay đổi khác nhau tùy loại sản phẩm được bán.

## 9. Khả Năng Tái Lập

Lời giải có thể được tái lập từ repository:

```text
Notebook: feature_ml.ipynb
Input folder: dataset/
Output file: dataset/submission.csv
```

Notebook sử dụng các phép tính xác định từ `numpy` và `pandas`. Ridge Regression được triển khai trực tiếp bằng đại số tuyến tính, nên không có bước khởi tạo ngẫu nhiên của mô hình. Điều này giúp giảm khác biệt giữa các lần chạy.

Để tái tạo file submission:

1. Mở `feature_ml.ipynb`.
2. Chạy toàn bộ cell từ trên xuống dưới.
3. Kiểm tra rằng `dataset/submission.csv` được tạo ra.
4. Nộp `dataset/submission.csv` lên Kaggle.

File submission giữ nguyên thứ tự dòng từ `sample_submission.csv` bằng cách merge dự đoán trở lại bảng ngày test gốc.

## 10. Hạn Chế Và Hướng Cải Thiện

Mô hình hiện tại ưu tiên sự ổn định và khả năng diễn giải. Tuy nhiên, vẫn có một số hướng cải thiện hiệu suất:

1. Thêm mô hình tree-based boosting như LightGBM, XGBoost hoặc CatBoost nếu được phép thêm dependency.
2. Ensemble dự đoán của Ridge với seasonal baseline để giảm phương sai.
3. Thêm SHAP values để giải thích chi tiết hơn cho từng ngày dự báo cụ thể.
4. Tinh chỉnh độ mạnh regularization của Ridge bằng nhiều chronological validation folds.
5. Train các mô hình chuyên biệt riêng cho giai đoạn high-season và normal-season.

Phiên bản hiện tại ưu tiên an toàn leakage, khả năng tái lập và khả năng giải thích, phù hợp trực tiếp với tiêu chí báo cáo kỹ thuật của Task 3.

