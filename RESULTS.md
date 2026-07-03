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