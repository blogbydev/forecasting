# ARIMA Notes

## 1. What is a Time Series?

A sequence of data points where **order matters** — each value is temporally dependent on past values.
ARIMA is **univariate**: one variable predicting itself using its own history.

```
Regular data:   [10, 7, 3, 15, 9]   ← order doesn't matter
Time series:    [Mon=10, Tue=7, Wed=3, ...]  ← order matters, values depend on each other
```

---

## 2. Stationarity

A stationary series has **constant mean, variance, and autocovariance** over time.

| Property     | Stationary         | Non-Stationary              |
|--------------|--------------------|-----------------------------|
| Mean         | Constant           | Drifts (trend)              |
| Variance     | Constant           | Changes over time           |
| Pattern      | No trend/seasonality | Has trend and/or seasonality |

### Sources of Non-Stationarity
- **Trend**: Revenue of a growing business (upward drift)
- **Seasonality**: Cracker sales in India (spikes every Diwali)

```
Non-stationary (trend):         Stationary:
  |        /                      |  ~ ~ ~ ~ ~ ~
  |      /                        | ~ ~ ~ ~ ~ ~
  |    /                          |~ ~ ~ ~ ~ ~
  |  /                            |
  |/                              |
  +--------                       +--------
```

---

## 3. The "I" — Integrated (Differencing)

**Differencing** removes trend: `Δy_t = y_t - y_{t-1}`

For a linear trend `y_t = mt + c`:
```
Δy_t = (mt + c) - (m(t-1) + c) = m   ← constant, trend removed
```

| d value | Meaning                        | Use when               |
|---------|--------------------------------|------------------------|
| d = 0   | Already stationary             | No differencing needed |
| d = 1   | Linear trend removed           | Series has linear trend|
| d = 2   | Quadratic trend removed        | Series has curved trend|

**How to find d**: Use the **ADF Test (Augmented Dickey-Fuller)**.
- ADF tests if φ₁ = 1 (unit root / random walk → non-stationary)
- Difference and re-test until ADF confirms stationarity
- Number of times you differenced = d

---

## 4. The "AR" — AutoRegressive (p)

Uses past **actual values** to predict the current value.

```
AR(1):  y_t = φ₁·y_{t-1} + ε_t
AR(2):  y_t = φ₁·y_{t-1} + φ₂·y_{t-2} + ε_t
```

φ (phi) = learned weight on each past value.

### Stability and φ₁

```
φ₁ = 0.1  →  |████      |  fast decay    → stationary
              |█         |
              |          |

φ₁ = 0.9  →  |██████████|  slow decay    → stationary
              |█████████ |
              |████████  |
              ...eventually reaches 0

φ₁ = 1.0  →  |██████████|  no decay      → NON-STATIONARY (random walk)
              |██████████|  shocks accumulate forever
              |██████████|

φ₁ = 1.1  →  |██████████ |  explodes     → NON-STATIONARY
              |████████████|
              |████████████████|
```

**Rule**: |φ₁| < 1 → stationary. This is why we difference first.

**p** = how many past actual values to include (AR order).

---

## 5. The "MA" — Moving Average (q)

Uses past **errors/shocks** to predict the current value.

```
MA(1):  y_t = μ + θ·ε_{t-1} + ε_t
MA(2):  y_t = μ + θ₁·ε_{t-1} + θ₂·ε_{t-2} + ε_t
```

ε (epsilon) = error = actual − predicted at that time step.
θ (theta) = learned weight on each past error.

### Intuition
A shock at t-1 may still reverberate at time t. MA captures how long a surprise lingers.

| Example                        | MA order |
|-------------------------------|----------|
| 1-day flash sale               | MA(1)    |
| Amazon 2-day sale              | MA(2)    |

**q** = how many past errors to include (MA order).

---

## 6. AR vs MA — Key Difference

| | AR | MA |
|--|----|----|
| Uses | Past actual values | Past errors/shocks |
| Captures | Long-term dependencies | Short-term shock propagation |
| Parameter | p | q |

---

## 7. ACF and PACF — Finding p and q

**Autocorrelation**: correlation of a series with a lagged version of itself.
- ACF at lag k = correlation between `y_t` and `y_{t-k}`

**Partial Autocorrelation (PACF)**: correlation between `y_t` and `y_{t-k}` after removing the indirect influence of all lags in between.

### Why the difference matters
```
Chain: y_{t-3} → y_{t-2} → y_{t-1} → y_t

ACF at lag 3:   HIGH  (picks up indirect chain effect)
PACF at lag 3:  ~0    (no direct link from y_{t-3} to y_t)
```

### How to use them

| Plot | Cuts off after lag | Tells you |
|------|--------------------|-----------|
| PACF | p                  | AR order  |
| ACF  | q                  | MA order  |

```
PACF plot:                    ACF plot:
  1.0 |█                       1.0 |█
  0.5 |█ █                     0.5 |█ █ █
  0.0 |      ~ ~ ~ ~           0.0 |          ~ ~ ~
     lag: 1 2 3 4 5               lag: 1 2 3 4 5
          ↑                                ↑
     cuts off at 2 → p=2            cuts off at 3 → q=3
```

---

## 8. Full ARIMA Pipeline

```
Raw Time Series
      │
      ▼
 ADF Test → non-stationary?
      │              │
     YES             NO → d = 0
      │
 Difference (d=1)
      │
 ADF Test again → still non-stationary?
      │              │
     YES             NO → d = 1
      │
 Difference again → d = 2
      │
 Now stationary
      │
      ▼
 Plot PACF → find p (AR order)
 Plot ACF  → find q (MA order)
      │
      ▼
 Fit ARIMA(p, d, q)
      │
      ▼
 Evaluate:
   - RMSE / MAE on test set
   - Residuals should be white noise (no autocorrelation left)
   - If residuals show pattern → increase p, d, or q
```

---

## 9. ARIMA(p, d, q) — Summary

| Parameter | Name       | Meaning                                      |
|-----------|------------|----------------------------------------------|
| p         | AR order   | How many past actual values to use           |
| d         | Difference | How many times to difference for stationarity|
| q         | MA order   | How many past errors/shocks to use           |

**Example: ARIMA(2, 1, 2)**
- Difference once to achieve stationarity
- Use last 2 actual values (AR)
- Use last 2 errors (MA)

---

## 10. Model Evaluation

| Check               | What to look for                              |
|--------------------|-----------------------------------------------|
| RMSE / MAE          | Lower is better                               |
| Residual plot       | Should look like white noise (random, flat)   |
| Residual ACF        | No significant spikes — no structure left     |

```
Good residuals:          Bad residuals (pattern left):
  |  .  .  .  .           |    /\    /\
  |. . . . . .            |   /  \  /  \
  |  .  .  .              |  /    \/    \
  +----------             +-------------
  → white noise           → model missed something
```
