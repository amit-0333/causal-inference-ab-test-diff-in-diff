# Project Journey: Forecasting UPI Transaction Volume with Uncertainty

## Notebook Intro Cell

The block below is written to be pasted as the very first markdown cell in the notebook, before any code — so anyone opening it cold understands the project in under a minute.

> ### Forecasting UPI Transaction Volume, with Honest Uncertainty
>
> **Goal:** Forecast India's monthly UPI transaction volume and measure how accurate that forecast actually is — not just draw a trend line, but show a genuine range of plausible outcomes and test it against real holdout data the model never saw during training.
>
> **Dataset:** UPI Transaction Monthly Data (India, 2016–2025), Kaggle — monthly transaction volume/value styled on NPCI/RBI-style reporting.
>
> **Approach:** Two forecasting methods compared — ARIMA and Prophet — each evaluated with 95% confidence intervals on a genuine 12-month holdout (Sep 2024–Aug 2025), followed by an unverified 6-month future forecast (Sep 2025–Feb 2026).
>
> **Headline result:** Prophet slightly outperformed ARIMA on holdout accuracy (MAPE 11.25% vs 11.44%) and tracked the real growth trend far more closely, but both models' confidence intervals became too wide to be practically useful by month 12 — a genuine limitation, not hidden in this write-up. See `PROJECT_JOURNEY.md` for the full story, including a data-leakage mistake caught and fixed mid-project.

## What this project is

A monthly time-series forecast of India's UPI (Unified Payments Interface) transaction volume, comparing two different forecasting approaches (ARIMA and Prophet), with genuine prediction intervals and honest accuracy measurement on a real holdout set. The goal wasn't just to draw a trend line into the future — it was to show a *range* of plausible outcomes and to test that range against real data the models never saw during training.

**Dataset:** UPI Transaction Monthly Data (India, 2016–2025), Kaggle — monthly transaction volume and value, styled on NPCI/RBI-style reporting.

**Tools:** Python, pandas, statsmodels (ARIMA), Prophet, scikit-learn (metrics).

---

## Phase 1: Scoping and Dataset Validation

Before writing any code, the dataset needed to be checked, not assumed trustworthy.

**What was checked:**
- Row count and date range — confirmed 113 monthly rows, April 2016 to August 2025, no gaps, no duplicate months.
- Null values — 3 nulls in derived columns (`Avg_Txn_Value_INR`, `MoM_Growth_Volume_%`, `MoM_Growth_Value_%`), traced to the start of the series where no prior month existed for growth calculations.

**Problem found:** The first three months (April–June 2016) had `Volume = 0` and `Value = 0` exactly. Initially this looked like it could be a data error, but the explanation was simple and correct: UPI genuinely launched as a near-zero-adoption pilot in April 2016. This wasn't missing data — it was a real "pre-launch" period.

**Why it mattered:** Keeping these rows would have broken the log-transform planned for handling UPI's exponential growth (`log(0)` is undefined), and made month-over-month growth calculations meaningless (0 → 0, or 0 → any number, is undefined growth).

**Resolution:** Trimmed the series to start from July 2016 — the first month with genuine non-zero activity. This wasn't cherry-picking for better results; it excluded a period no forecasting model could reasonably explain anyway (a market that didn't functionally exist yet). Left 110 months of real data.

---

## Phase 2: Data Cleaning

Beyond the pre-launch trim, three concrete fixes were needed:

1. **Date formatting** — `Month` was stored as text (`"Aug-25"`), and rows were ordered newest-first. Parsed into a real `datetime` and sorted chronologically, since time-series models need to know actual chronological spacing between points.

2. **Leftover `inf` values** — After trimming, the first remaining row (July 2016) still showed `inf` for both growth-rate columns, calculated against June 2016 (which was zero). Going from zero to any positive number is mathematically "infinite" growth — not a usable number. Replaced with `NaN` rather than imputing a fake value like 0%, which would have misrepresented "undefined" as "no growth."

3. **What was left alone** — `volume_mn` and `value_cr`, the actual forecasting targets, had zero missing values after the trim. No imputation was applied to them.

**Result:** 110 clean, contiguous months (July 2016 – August 2025), zero missing values in the forecast targets.

---

## Phase 3: EDA — Stationarity and Seasonality

This phase existed to justify model configuration choices with evidence, not guesswork.

**Stationarity (ADF test):**

| Series | p-value | Stationary? |
|---|---|---|
| Raw volume | 0.96 | No |
| log(volume) | 0.75 | No |
| log(volume), differenced once | <0.001 | Yes |

This confirmed ARIMA needed to run on log-transformed, once-differenced data (`d=1`).

**Seasonality (STL decomposition):**

A more important and less obvious finding came from decomposing the series. The seasonal component swung wildly (±50%) during 2016–2019, then stabilized into a much smaller, consistent pattern (±5–15%) from around 2021 onward. The residual component told the same story — large errors early, near-zero later.

**Interpretation:** The early "seasonality" wasn't a real recurring calendar pattern — it was noise from UPI's explosive early growth off a tiny base being misread as seasonal swings. Real, stable seasonality only became visible once the platform matured.

**Decision made:** Because early-period "seasonality" was unreliable, a SARIMA model trained on the full noisy history risked conflating early growth chaos with genuine seasonal signal. Chose **plain ARIMA (no seasonal term)** for the trend/level forecast, and let **Prophet carry the seasonality analysis**, since Prophet's changepoint detection is more robust to this kind of early-regime instability. This difference in how each model handles the noisy early period became a real comparison point later.

---

## Phase 4: Train/Holdout Split

Held out the **last 12 months (September 2024 – August 2025)** as a genuine test set — 98 months for training, 12 for evaluation. A full year of holdout, not just a few months, so the evaluation could catch seasonal misses, not just short-term luck. Notably, the holdout period's volume range sat entirely above the training data's typical range — the models were being tested on genuine extrapolation, not interpolation.

---

## Phase 5: Building ARIMA

**Order selection:** ACF/PACF plots on the log-differenced training data showed a sharp cutoff dominated by lag 1, pointing to a low-order model. Comparing candidate orders by AIC confirmed **ARIMA(1,1,1)** as the best fit — agreeing with the visual read.

**Residual diagnostics:** Ljung-Box test (p = 0.089) showed no significant leftover autocorrelation — good sign. But the Jarque-Bera test showed residuals were far from normally distributed (skew = 2.30, kurtosis = 12.63), driven by extreme volatility during UPI's chaotic 2016–2018 launch phase. This meant ARIMA's confidence intervals — which assume roughly normal residuals — were likely somewhat miscalibrated (too narrow). Decision: keep the full training history rather than truncate the noisy years, and flag this honestly rather than hide it.

**A rejected experiment, and why it mattered:** Tried adding an explicit trend term to let ARIMA extrapolate growth forward. It improved the in-sample fit (AIC -18.97 → -23.37) but **catastrophically worsened the holdout forecast** (MAPE 11.44% → 86.24%) — the drift term, extrapolated on a log scale, compounded into wild overshooting. This was a genuinely important lesson: a model that looks better by training-data criteria can perform far worse on real unseen data. It's a concrete demonstration of why holdout evaluation — not in-sample fit — is what actually matters.

**Final ARIMA(1,1,1) holdout results:**

**MAE: 2,105 Mn | RMSE: 2,536 Mn | MAPE: 11.44% | 95% CI coverage: 12/12**

The 100% coverage looked good on paper but was misleading: the point forecast was nearly flat (~15,400) for the full 12 months while actual volume climbed to over 20,000. The interval only "worked" because it grew enormous (13,000-wide in month 1 to 436,000-wide by month 12) — wide enough to catch nearly anything, which is technically correct but not practically useful by the end of the horizon.

---

## Phase 6: Building Prophet

**First attempt failed badly** (MAPE 36.60%) using Prophet's default settings. Investigating why revealed the same early-noise problem flagged in EDA: Prophet's default yearly seasonality term jumped from -0.076 in November to +0.38 in December (a ~47% seasonal boost) — learned from UPI's volatile 2016–2018 launch period, not a real recent pattern.

**Testing seasonality fixes helped only marginally** (36.6% → ~34%), even with seasonality turned off entirely — showing seasonality wasn't the main driver.

**The real cause: trend rigidity.** UPI's growth isn't purely exponential — it's an S-curve, decelerating as adoption matures. Prophet's default trend, fit rigidly across the entire history (including the steep early ramp), extrapolated that early steepness forward and overshot badly. Testing showed that increasing `changepoint_prior_scale` (letting the trend flex to match recent, decelerating growth) dramatically improved accuracy — the mirror image of the ARIMA-trend lesson from Phase 5.

**A real mistake, caught mid-project:** While tuning `changepoint_prior_scale` and `seasonality_prior_scale`, the search was initially run by checking performance directly against the real 12-month holdout — picking whichever setting scored best on it. This is a subtle form of leakage: tuning against the "unseen" test set means it's no longer a genuine test of unseen performance, even though no code was technically "wrong."

**Fix:** Carved out a separate internal validation window (Sep 2023–Aug 2024) *within* the training data — never touching the real holdout — tuned hyperparameters honestly against that, then evaluated the final chosen model on the untouched holdout exactly once.

**Final tuned Prophet (`changepoint_prior_scale=1.5`, `seasonality_prior_scale=0.1`) holdout results:**

**MAE: 2,012 Mn | RMSE: 2,423 Mn | MAPE: 11.25% | 95% CI coverage: 12/12**

Slightly better point-forecast accuracy than ARIMA, and visually it tracked the actual data's shape (including catching a real December uptick) far more closely than ARIMA's flat line.

**A new trade-off, not a clean win:** The same trend flexibility that improved point accuracy also made Prophet's uncertainty intervals compound explosively over the 12-month horizon — the upper bound reached ~10.5 million by month 12, even wider than ARIMA's already-too-generous interval. Better central estimate, worse-calibrated long-horizon uncertainty.

---

## Phase 7: Final Comparison and Verdict

Neither model "wins" cleanly — the honest answer depends on what's being optimized for.

- **Point-forecast accuracy:** Prophet slightly ahead (MAPE 11.25% vs 11.44%), and it visibly tracks the real trend shape much better than ARIMA's near-flat line.
- **Interval calibration:** Both models achieved 12/12 coverage, but only because both intervals grew wide enough to catch almost anything by month 12. Perfect coverage here is a symptom of overly wide intervals, not a sign of a well-calibrated model.
- **Practical takeaway:** ARIMA is the safer, more conservative choice for short-term forecasts (1–3 months), where its flat-ish bias does less damage. Prophet is noticeably better at capturing a maturing, decelerating growth trend over a longer horizon, but its flexibility makes its long-horizon uncertainty estimates less trustworthy.

---

## Phase 8: Future Forecast (Unverified)

Both models were refit on the **full 110-month dataset** and used to forecast 6 months beyond the last known data point (August 2025) — September 2025 through February 2026. Unlike the holdout evaluation, these predictions cannot be checked against reality yet.

ARIMA predicted a flat ~20,200 for every future month, consistent with its earlier weakness of not projecting the growth trend forward. Prophet predicted continued growth, peaking around 27,000 in December before easing — more consistent with UPI's real, ongoing growth story. Both intervals widened substantially further out. Prophet's forecast was treated with more confidence given its stronger holdout performance, but neither was read as a precise, guaranteed number.

---

## Future Work: Verifying the Forecast

The Phase 8 forecast (Sep 2025 – Feb 2026) is genuinely unverified — as of writing, real UPI data for those months either doesn't exist yet or hasn't been published by NPCI. This isn't a gap in the project; it's the honest nature of forecasting the future rather than backtesting the past.

**Once real data is available** (NPCI publishes monthly UPI statistics, the same source this dataset was built from), the forecast can be properly checked:
1. Pull the actual Sep 2025–Feb 2026 volume figures once published.
2. Re-run the same MAE/RMSE/MAPE calculations used for the original holdout evaluation, comparing predictions to the real values.
3. Check how many actual values fall inside the predicted 95% confidence intervals.
4. Report the result honestly — if Prophet's growth-tracking forecast proves closer to reality than ARIMA's flat one, that would confirm the holdout-based conclusion; if not, that's worth stating plainly too.

This step can't be completed today, but it's the natural way this project would continue to be validated as real-world data catches up to the forecast window.

---

## What This Project Demonstrates

- Validating a dataset's integrity before trusting it (the pre-launch zero-volume discovery)
- Justifying model configuration with statistical evidence (ADF tests, ACF/PACF, STL decomposition) rather than guesswork
- Recognizing when a model's in-sample fit and its real-world forecast quality diverge (the ARIMA-trend overshoot)
- Catching and correcting a genuine data leakage mistake mid-project, rather than only being taught about leakage in the abstract
- Being honest about a model's limitations even when the topline metric looks good (the "12/12 coverage" that only worked because the interval was too wide to be useful)
- Distinguishing between backtesting (measuring accuracy on known data) and true forecasting (predicting genuinely unknown future data) — and being explicit about which is which
