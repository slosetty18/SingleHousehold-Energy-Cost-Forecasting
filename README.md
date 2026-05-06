# Single Household Energy Cost Forecasting ⚡

Next-day electricity cost forecasting using 3 years of personal SRP utility data, SARIMAX time series modeling, and domain-engineered features based on Arizona TOU pricing.

---

## Results

| Metric | Value |
|--------|-------|
| MAE | **$0.485** |
| RMSE | **$0.584** |
| Validation window | 90 days out-of-sample (Feb–May 2026) |
| Training period | 3 years (Feb 2023 – Jan 2026) |
| Model | SARIMAX(1,1,1)(1,0,1,7) |

**MAE by month:**

| Month | MAE |
|-------|-----|
| February | $0.425 |
| March | $0.564 |
| April | $0.469 |
| May (1 day) | $0.663 |

---

## Problem

SRP's EZ-3 Time-of-Use plan charges significantly different rates depending on the season and time of day:

| Season | Off-peak Rate | On-peak Rate | On-peak Hours |
|--------|--------------|-------------|---------------|
| Winter (Nov–Apr) | 9.70¢/kWh | 12.94¢/kWh | 3–6pm weekdays |
| Summer (May, Jun, Sep, Oct) | 10.30¢/kWh | 30.96¢/kWh | 3–6pm weekdays |
| Summer Peak (Jul–Aug) | 10.69¢/kWh | 36.61¢/kWh | 3–6pm weekdays |

Knowing tomorrow's expected cost enables smarter energy decisions — pre-cooling before peak hours, scheduling appliances overnight, and reducing on-peak usage during expensive summer months.

---

## Features

| Feature | Description | Correlation with Cost |
|---------|-------------|----------------------|
| `lag1` | Yesterday's actual total cost | 0.85 |
| `temp_avg` | Average of daily high and low temperature | 0.74 |
| `srp_rate` | Blended SRP rate weighted by actual kWh usage | 0.75 |
| `is_weekend` | 1 if Saturday or Sunday — SRP charges no on-peak rates on weekends, all hours are off-peak | -0.15 |
| `is_holiday` | 1 if one of SRP's 6 off-peak holidays — treated same as weekends, all hours off-peak | -0.01 |

**Weekends and the 6 holidays below are always off-peak — no on-peak charges apply regardless of time of day.**

**SRP's 6 off-peak holidays:** New Year's Day, Memorial Day, Independence Day, Labor Day, Thanksgiving Day, Christmas Day

---

## Model

**SARIMAX(1,1,1)(1,0,1,7)** — Seasonal AutoRegressive Integrated Moving Average with eXogenous variables

| Parameter | Value | Meaning |
|-----------|-------|---------|
| p=1, d=1, q=1 | Non-seasonal order | AR, differencing, MA terms |
| P=1, D=0, Q=1, s=7 | Seasonal order | Weekly seasonal pattern |

**Model comparison vs original:**

| Model | AIC | Features |
|-------|-----|---------|
| Original (no srp_rate) | 2958 | temp_avg, is_weekend, is_holiday, lag1 |
| Current (with srp_rate) | **2429** | temp_avg, is_weekend, is_holiday, srp_rate, lag1 |

Lower AIC = better fit. Adding `srp_rate` improved AIC by **529 points**.

---

## Key Findings

- `srp_rate` is the strongest engineered feature (coef=0.41, z=34) — captures seasonal pricing directly
- `lag1` correlation of 0.85 confirms strong day-to-day cost persistence
- Model has upward bias (mean residual = -$0.33) — 2026 winter costs ran lower than 3-year training average
- Errors reduce through April as lag1 self-corrects on real validation costs
- Residual kurtosis = 7.36 — extreme summer heatwave days create fat tails

---

## Project Structure

```
├── Energy_cost_forecasting.ipynb    # Full pipeline — EDA, training, validation
├── cost_forecast_model_latest.pkl   # Trained SARIMAX model
├── model_config.json                # Model configuration and metadata
└── README.md
```

---

## How to Run

1. Upload your SRP CSV files to Colab:
   - `dailyUsage_<date_range>.csv`
   - `dailyCost_<date_range>.csv`

2. Open `Energy_cost_forecasting.ipynb` in Google Colab
3. Run all cells in order

---

## Data

- **Source:** Personal SRP utility account (myaccount.srpnet.com)
- **Plan:** SRP EZ-3 3–6pm Time-of-Use pricing
- **Training:** Feb 2023 – Jan 2026 (1,095 days)
- **Validation:** Feb – May 2026 (90 days out-of-sample)
- **Features per day:** Off-peak kWh, On-peak kWh, High/Low temperature, Off-peak cost, On-peak cost

*Note: Raw data files not included — personal utility data. Download your own from myaccount.srpnet.com.*

---

## Stack

Python · statsmodels · pandas · NumPy · scikit-learn · Matplotlib · Seaborn · Google Colab

---

## Author

**Suhasini** · [LinkedIn](https://linkedin.com/in/suhasinilosetty) · [Medium](https://medium.com/@slosetty18)
