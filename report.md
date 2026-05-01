# Task 3 Technical Report: Sales Forecasting

## 1. Executive Summary

This report describes our forecasting pipeline for Task 3, where the goal is to predict daily `Revenue` and `COGS` for the hidden test period from `2023-01-01` to `2024-07-01`. The final solution is implemented in `feature_ml.ipynb` and generates `dataset/submission.csv` in the required format: `Date`, `Revenue`, and `COGS`.

Instead of relying only on the historical daily sales table, we built a feature-based time-series model that uses the full internal business context available in the dataset. The model combines historical sales patterns, calendar seasonality, order behavior, product mix, promotions, web traffic, inventory signals, returns, reviews, payments, shipments, customer acquisition, and geography. These signals are transformed into leakage-safe historical profiles so they can be used for future dates where actual transaction-level data is not available.

The model is validated using a walk-forward setup: train on data up to `2021-12-31`, forecast each day in 2022 recursively, and compare predictions with actual 2022 values. This mirrors the real forecasting situation more closely than random train/test splitting.

## 2. Pipeline Overview

The pipeline follows a reproducible sequence:

1. Load internal competition data from the `dataset/` folder.
2. Aggregate transaction, operational, customer, promotion, traffic, and geography tables to daily-level signals.
3. Convert unavailable future signals into calendar-based historical profiles.
4. Build lag and rolling-window features from historical `Revenue` and `COGS`.
5. Train two separate Ridge Regression models: one for `Revenue`, one for `COGS`.
6. Validate the model with walk-forward forecasting on 2022.
7. Retrain on the full 2012-2022 history.
8. Forecast the hidden test period recursively and export `submission.csv`.

The design separates feature construction from model training. This makes the pipeline easier to audit, easier to rerun, and safer against accidental test leakage.

## 3. Data Sources Used

The model uses all relevant internal files except the hidden test answers:

| Data source | How it contributes to the model |
|---|---|
| `sales.csv` | Target history for `Revenue` and `COGS`; source of lag and rolling features |
| `orders.csv` | Order volume, order status mix, device mix, payment method mix, acquisition source mix |
| `order_items.csv` | Units sold, discount amount, product mix, transaction-level revenue proxy |
| `products.csv` | Product category and product-level COGS information |
| `payments.csv` | Payment value and installment behavior |
| `shipments.csv` | Shipping fee and delivery duration signals |
| `returns.csv` | Return volume, refund amount, and return reason mix |
| `reviews.csv` | Review volume, average rating, and low-rating share |
| `customers.csv` | Signup volume and acquisition-channel profile |
| `promotions.csv` | Active promotion count, promotion type, discount value, promotion channel |
| `inventory.csv` | Stock availability, stockout, reorder, fill-rate, sell-through signals |
| `web_traffic.csv` | Sessions, visitors, page views, bounce rate, traffic-source mix |
| `geography.csv` | Regional order distribution by `East`, `Central`, and `West` |
| `sample_submission.csv` | Test dates and required submission row order |

No external data is used.

## 4. Feature Engineering

Feature engineering is the main strength of this solution. The model does not only ask, "What was sales last year?" It also asks business-relevant questions such as:

- Is this date usually high or low season?
- Is this day close to a weekly or yearly sales pattern?
- What was the recent demand trend?
- Were similar historical dates associated with higher web traffic?
- Were similar dates promotion-heavy?
- Did inventory availability or stockout behavior historically affect sales?
- Did returns, refunds, ratings, and shipping behavior indicate customer friction?
- Did demand historically differ by region or product category mix?

The most important feature groups are:

| Feature group | Example features | Business meaning |
|---|---|---|
| Calendar features | `month`, `dow`, `is_weekend`, `year_sin`, `year_cos` | Captures seasonality, weekends, and yearly shopping cycles |
| Lag features | `revenue_lag_7`, `revenue_lag_364`, `cogs_lag_364` | Captures weekly repetition and same-season last-year demand |
| Rolling features | `revenue_roll_mean_28`, `cogs_roll_mean_364`, `revenue_roll_std_364` | Captures recent trend and demand volatility |
| Sales calendar profiles | `sales_md_revenue_mean`, `sales_mwd_cogs_mean` | Captures average historical behavior for the same calendar day or weekday pattern |
| Order profiles | `md_order_count`, `mwd_customer_count`, `md_mobile_share` | Captures normal demand and customer behavior for comparable dates |
| Promotion profiles | `md_active_promo_count`, `md_promo_discount_value` | Captures the expected promotional intensity for similar historical dates |
| Inventory profiles | `md_stock_on_hand`, `mwd_stockout_days`, `md_fill_rate` | Captures whether product availability historically supported or constrained sales |
| Web traffic profiles | `md_sessions`, `mwd_page_views`, `md_bounce_rate` | Captures expected online demand and traffic quality |
| Return and review profiles | `md_refund_amount`, `md_return_count`, `md_rating_mean` | Captures customer satisfaction and post-purchase friction |
| Geography profiles | `md_east_region_share`, `md_west_region_share` | Captures regional demand composition |

For future dates, actual future orders, traffic, returns, and inventory are unknown. Therefore, the model does not use future observed values from these tables. Instead, it uses historical profiles such as:

- average behavior for the same `month-day`, for example historical January 1 patterns;
- average behavior for the same `month-weekday`, for example Mondays in January.

This gives the model business context without leaking hidden future information.

## 5. Model Choice

The notebook uses Ridge Regression on log-transformed targets:

```text
target used by model = log(1 + Revenue)
target used by model = log(1 + COGS)
```

Ridge Regression was selected for three reasons:

1. It is stable with many correlated features, such as multiple lag and rolling-window variables.
2. It is interpretable through standardized coefficients.
3. It is lightweight and reproducible with only `numpy`, `pandas`, and `matplotlib`.

The log transformation helps because sales values can vary widely across normal days, high-demand periods, and promotion-heavy periods. Modeling the log target reduces the effect of extreme spikes and makes the regression problem more stable. Predictions are transformed back to the original scale with `expm1`.

Two models are trained independently:

- one model for `Revenue`;
- one model for `COGS`.

This is appropriate because revenue and cost are related but not identical. Revenue is affected by price, discount, demand, and traffic; COGS is affected by product mix and units sold.

## 6. Validation Strategy

The validation strategy is walk-forward forecasting:

```text
Train period: data available up to 2021-12-31
Validation period: 2022-01-01 to 2022-12-31
Forecast style: recursive, one day at a time
```

This setup is stronger than random splitting because the real task is also a future forecasting problem. In a random split, the model might train on later dates and validate on earlier dates, which would not reflect the actual Kaggle task.

Local validation results on 2022:

| Target | MAE | RMSE | R2 |
|---|---:|---:|---:|
| `Revenue` | 603,521 | 808,695 | 0.7666 |
| `COGS` | 572,348 | 756,149 | 0.7312 |

Interpretation:

- MAE means the average absolute daily error.
- RMSE penalizes large misses more heavily than MAE.
- R2 measures how much variation the model explains compared with a naive mean prediction.

The R2 values show that the model captures a meaningful share of daily variation in both `Revenue` and `COGS`. Since the validation period is a full future year, this gives a reasonable estimate of out-of-sample forecasting performance.

## 7. Leakage Prevention

Leakage is one of the most important risks in this task. Leakage means the model accidentally sees information that would not be available at prediction time.

The pipeline prevents leakage in several ways:

1. Test `Revenue` and `COGS` are never used as features.
2. All target-derived lag and rolling features are shifted so that each prediction only uses past values.
3. Rolling statistics use `shift(1)` before calculating the rolling mean or standard deviation.
4. Future transaction-level, traffic, promotion, inventory, return, and review values are not used directly.
5. Data sources that are unavailable in the future are converted into historical calendar profiles.
6. Validation is chronological, not random.
7. The final forecast is recursive: once the model enters the future period, it only uses past actual values or earlier predictions.

For example, a correct rolling feature is:

```text
mean Revenue from the previous 28 days
```

not:

```text
mean Revenue from a window that includes the current day
```

This distinction matters because including the current day would allow the model to indirectly see the answer it is supposed to predict.

## 8. Feature Importance and Business Interpretation

Because the model is Ridge Regression with standardized features, feature importance is interpreted using the absolute value of standardized coefficients. Larger coefficients mean the model relies more heavily on that feature.

The most important signals in local validation include:

| Important feature | Business interpretation |
|---|---|
| `cogs_roll_mean_364` | Long-term yearly cost pattern is highly predictive, suggesting strong annual seasonality in product demand and product mix |
| `revenue_roll_mean_364` | Revenue has strong year-level seasonality; similar periods in previous years are informative |
| `day` and yearly sine/cosine features | Calendar position matters, consistent with seasonal fashion demand |
| `revenue_lag_1` and `cogs_lag_1` | Yesterday's performance contains useful short-term momentum |
| `cogs_roll_mean_182` and `revenue_roll_mean_91` | Medium-term sales and cost trends help estimate future demand level |
| `md_outdoor_units` | Historical product category mix affects expected revenue and cost |
| `md_refund_amount` | High-refund historical periods may indicate customer friction or operational issues |
| `md_payment_value_proxy` and `md_payment_value` | Payment behavior is closely tied to demand level |
| `md_stock_on_hand`, `mwd_stock_on_hand`, `md_fill_rate` | Inventory availability influences whether demand can convert into sales |
| `margin_ratio_lag_1` | Recent gross margin behavior helps estimate the relationship between revenue and cost |

In business terms, the model suggests that fashion e-commerce demand is driven by four major forces:

1. Seasonality: sales patterns repeat across weeks, months, and years.
2. Momentum: recent demand strongly affects near-future demand.
3. Merchandising and inventory: product availability and category mix shape both revenue and COGS.
4. Customer and operational friction: refunds, returns, reviews, and delivery behavior provide signals about demand quality.

This interpretation is useful because it does not treat the model as a black box. It connects model behavior back to decisions a business team can understand:

- prepare inventory earlier for historically high-demand periods;
- monitor refund-heavy periods because they may reduce effective revenue quality;
- align promotions and traffic campaigns with seasonal peaks;
- track product-category mix because revenue and COGS move differently depending on what is sold.

## 9. Reproducibility

The solution is reproducible from the repository:

```text
Notebook: feature_ml.ipynb
Input folder: dataset/
Output file: dataset/submission.csv
```

The notebook uses deterministic numerical operations from `numpy` and `pandas`. Ridge Regression is implemented directly with matrix algebra, so there is no random model initialization. This reduces run-to-run variation.

To reproduce the submission:

1. Open `feature_ml.ipynb`.
2. Run all cells from top to bottom.
3. Confirm that `dataset/submission.csv` is generated.
4. Submit `dataset/submission.csv` to Kaggle.

The submission preserves the row order from `sample_submission.csv` by merging predictions back onto the original test date table.

## 10. Limitations and Future Improvements

The current model is intentionally stable and interpretable. However, there are several ways to improve performance:

1. Add a tree-based boosting model such as LightGBM, XGBoost, or CatBoost if extra dependencies are allowed.
2. Ensemble Ridge predictions with a seasonal baseline to reduce variance.
3. Add SHAP values for a richer local explanation of individual forecast dates.
4. Tune Ridge regularization strength using multiple chronological validation folds.
5. Train separate specialized models for high-season and normal-season periods.

The current version prioritizes leakage safety, reproducibility, and explainability, which are directly aligned with the technical report criteria for Task 3.

