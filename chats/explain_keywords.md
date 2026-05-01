

## COGS (Cost of Goods Sold) — Giá vốn hàng bán

Bạn mua 1 chiếc áo từ xưởng giá **80 nghìn**, rồi bán cho khách giá **100 nghìn**.

- 100 nghìn = **Revenue** (doanh thu — tiền khách trả)
- 80 nghìn = **COGS** (giá vốn — tiền bạn bỏ ra mua hàng)
- 20 nghìn = lợi nhuận

COGS là "tiền gốc" bạn phải bỏ ra để có hàng bán.

---

## MAE, RMSE, R² — Ba cách chấm điểm "đoán giỏi hay dở"

Giả sử 3 ngày liên tiếp, doanh thu thật là **100, 200, 300** (triệu). Bạn đoán **110, 180, 290**.

### MAE (Mean Absolute Error) — Sai trung bình

Đếm mỗi lần bạn đoán sai bao nhiêu, rồi tính trung bình:

```
Ngày 1: đoán 110, thật 100 → sai 10
Ngày 2: đoán 180, thật 200 → sai 20
Ngày 3: đoán 290, thật 300 → sai 10

MAE = (10 + 20 + 10) / 3 = 13.3
```

**MAE càng nhỏ = đoán càng giỏi.** MAE = 0 nghĩa là đoán đúng 100%.

---

### RMSE (Root Mean Squared Error) — Sai trung bình nhưng phạt nặng lỗi lớn

Giống MAE nhưng **khắt khe hơn**. Nếu bạn đoán sai 1 ngày rất nhiều, RMSE sẽ phạt nặng hơn.

```
Sai: 10, 20, 10
Bình phương: 100, 400, 100       ← nhân mỗi số với chính nó
Trung bình: (100 + 400 + 100) / 3 = 200
Căn bậc 2: √200 ≈ 14.1
```

**RMSE càng nhỏ = càng tốt.** RMSE luôn ≥ MAE. Nếu RMSE lớn hơn MAE nhiều, nghĩa là có vài ngày bạn đoán sai rất xa.

Tưởng tượng: MAE giống cô giáo hiền — sai bao nhiêu trừ bấy nhiêu. RMSE giống cô giáo nghiêm — sai nhiều thì bị phạt gấp đôi.

---

### R² (R-squared) — Đoán giỏi hơn "đoán bừa" bao nhiêu?

Cách "đoán bừa" đơn giản nhất là luôn đoán **số trung bình**. Trung bình 3 ngày = 200, vậy đoán bừa = luôn nói "200".

R² so sánh bạn với cách đoán bừa đó:

- **R² = 1.0** → bạn đoán đúng hoàn hảo
- **R² = 0.5** → bạn đoán giỏi hơn đoán bừa kha khá
- **R² = 0** → bạn đoán cũng tệ như đoán bừa
- **R² < 0** → bạn đoán còn tệ hơn đoán bừa (rất dở)

**R² càng gần 1 = càng tốt.**

---

## Tóm lại

| Chỉ số | Ý nghĩa | Tốt khi |
|--------|----------|---------|
| MAE | Sai trung bình bao nhiêu | Càng nhỏ càng tốt |
| RMSE | Sai trung bình, phạt nặng lỗi lớn | Càng nhỏ càng tốt |
| R² | Giỏi hơn đoán bừa bao nhiêu | Càng gần 1 càng tốt |

Kaggle sẽ dùng 3 chỉ số này để chấm điểm file submission của bạn.