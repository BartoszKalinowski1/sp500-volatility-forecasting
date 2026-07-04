# Results Log — S&P 500 Volatility Forecasting

Running log of statistical test results and model outputs.

## Data & Stationarity Tests

| Test | Statistic | p-value | Conclusion |
|---|---|---|---|
| Jarque-Bera (normality) | 29405.91 | < 0.001 | Reject H₀ — log returns are not normally distributed |
| ADF (price) | 0.8266 | 0.992 | Fail to reject H₀ — price series is non-stationary (has a trend) |
| ADF (log returns) | -16.9525 | < 0.001 | Reject H₀ — returns are stationary |
| Ljung-Box (squared returns, lag 10) | 2982.49 | < 0.001 | Reject H₀ — volatility clustering confirmed |

**Interpretation:** S&P 500 daily log returns violate the normality assumption underlying
Black-Scholes (fat tails, leptokurtosis). Returns (not raw prices) are the correct series to
model, since they are stationary. Volatility clustering is statistically confirmed, justifying
an attempt to forecast future volatility from past volatility.

## Black-Scholes Volatility Sensitivity

| Volatility assumption | σ (annualized) | Call option price |
|---|---|---|
| Historical (full sample) | 0.1796 | $154.80 |
| Recent (21-day realized) | 0.0884 | $84.04 |

**Interpretation:** Option prices computed via Black-Scholes are highly sensitive to the
volatility input (Vega effect). Using a static historical volatility estimate versus a recent
realized estimate produces a 45.7% difference in the theoretical option price for the same
contract. This is a direct practical consequence of the volatility clustering confirmed
statistically in 01_data_and_stationarity — motivating the need for a model that forecasts volatility dynamically
rather than assuming it is constant.

## Feature Engineering & Correlation Analysis

Target: realized volatility over the next 21 trading days (forward-looking, no leakage).

| Feature | Correlation with future 21-day volatility |
|---|---|
| realized_volatility_5 | 0.535 |
| realized_volatility_18 | 0.505 |
| realized_volatility_21 | 0.490 |
| volume_ma_5 | 0.410 |
| volume_change | 0.019 |
| day_of_week | 0.002 |
| return_lag_5 | -0.091 |
| return_lag_3 | -0.110 |
| return_lag_2 | -0.118 |
| return_lag_1 | -0.129 |
| month | -0.162 |

**Interpretation:** Recent realized volatility (shortest 5-day window) is the strongest linear
predictor of future volatility — consistent with the volatility clustering confirmed in 01_data_and_stationarity.
Volume is meaningfully predictive; calendar and single-day lagged return features carry little
to no linear signal individually. Selected features for modeling: realized volatility (5/18/21-day),
volume_ma_5, and month.

## Baseline & Linear Regression

Chronological 80/20 train-test split (test set: 2023-09-29 to 2025-12-01), 545 test observations.

| Model | RMSE | MAE | R² |
|---|---|---|---|
| Naive baseline | 0.0906 | 0.0568 | — |
| Linear Regression | 0.0730 | 0.0501 | 0.0924 |

**Interpretation:** Linear regression outperforms the naive baseline (RMSE -19%, MAE -12%),
confirming the engineered features add real predictive value. However, R² = 0.092 indicates
the model explains only ~9% of the variance in future volatility, consistent with markets
being inherently difficult to forecast (see 01_data_and_stationarity fat-tail findings). Model coefficients show
multicollinearity across the three realized-volatility windows (18-day and 5-day positive,
21-day negative), and `volume_ma_5` contributes essentially nothing once combined with
volatility features despite its standalone correlation — motivating tree-based models in next part of project.

## Tree-Based Models (Random Forest, XGBoost)

| Model | RMSE | MAE | R² |
|---|---|---|---|
| Naive Baseline | 0.0906 | 0.0568 | — |
| Linear Regression | 0.0730 | 0.0501 | 0.0924 |
| Random Forest | 0.0915 | 0.0517 | -0.4236 |
| XGBoost | 0.0951 | 0.0544 | -0.5397 |

5-fold TimeSeriesSplit cross-validation: Random Forest CV RMSE = 0.1005 (±0.0482),
XGBoost CV RMSE = 0.1118 (±0.0496) — confirming the poor test-set performance is not a
one-off artifact of the specific train/test split.

**Interpretation:** Both tree-based models underperform Linear Regression and even the naive
baseline, with negative R² indicating worse-than-mean predictions on the test period. This is
most likely driven by a regime shift between training (2015–2023, including the 2020 shock)
and test (2023–2025) periods — the trees overfit patterns from training data that don't
generalize to a different volatility regime, while linear regression's simpler form is more
robust to this shift. Linear Regression remains the best model so far.