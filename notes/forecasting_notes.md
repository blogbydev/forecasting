# Time Series Forecasting — Course Notes

---

# 1. What Is Forecasting

Forecasting uses **historical patterns to predict the future**. The output is a **range of plausible values**, not a single point — uncertainty grows with the horizon.

Two sources of repetition in a series:

| Source | Mechanism | Example |
|---|---|---|
| **External drivers** | Independent events cause the outcome | Weather causes umbrella sales |
| **Internal momentum** | Past values influence future values | Momentum in stock prices |

---

# 2. The Fundamental Question

> **Before picking any algorithm: does the series have internal memory, external drivers, or both?**

This question drives every modeling decision downstream.

```mermaid
flowchart TD
    A[Observe your series] --> B{What drives it?}
    B --> C[Internal memory\npast values predict future]
    B --> D[External drivers\noutside events predict future]
    B --> E[Both]
    C --> F[AR / ARIMA / ETS]
    D --> G[Prophet / Regression\nwith time features]
    E --> H[ARIMAX / SARIMA\nML hybrids]
```

---

# 3. Regression Recap

- **Regression** = finding weights that best explain an output from inputs
- **The act of regressing** = iteratively adjusting weights until error is minimized
- **Autoregression** = regression where the inputs are **past values of the same series**
- "Auto" comes from Greek *autos* = self — same root as *autobiography*

---

# 4. Autoregression (AR)

Past values of a series predict its future values. Key ideas:

- **Lag structure** — how many past values (lags) to include as predictors
- **Influence fades with distance** — recent past matters more than distant past
- A **sliding window** of fixed size moves forward one period at a time

```mermaid
flowchart LR
    subgraph Window["Sliding Window  (size = 3)"]
        direction LR
        T1["t-3\nweight: small"] --> T2["t-2\nweight: medium"]
        T2 --> T3["t-1\nweight: large"]
    end
    T3 -->|predict| T4["t\n(forecast)"]

    subgraph Next["Window shifts forward"]
        direction LR
        T2b["t-2\nweight: small"] --> T3b["t-1\nweight: medium"]
        T3b --> T4b["t\nweight: large"]
    end
    T4b -->|predict| T5["t+1\n(forecast)"]
```

Each step: drop the oldest observation, add the newest, reuse the same weights.

---

# 5. Stationarity

## Definition
A series is **stationary** if its statistical properties — mean and variance — do not drift over time.

## Why It Matters
AR and ARIMA models assume a **stable underlying process**. If mean or variance shifts, the model's learned parameters become unreliable.

## Non-Stationarity Types

| Type | Description | Fix |
|---|---|---|
| **Drifting mean** (trend) | Mean increases or decreases over time | Differencing |
| **Growing variance** | Spread of values expands over time | Log transformation |
| **Structural break** | Process fundamentally changes | Cannot fix by transformation |

## Achieving Stationarity — Iterative Process

```mermaid
flowchart TD
    A[Raw series] --> B[Run ADF test]
    B --> C{Stationary?}
    C -->|Yes| D[Ready to model]
    C -->|No — trend| E[Apply differencing]
    C -->|No — variance| F[Apply log transform]
    E --> B
    F --> B
    D --> G[Check residuals after fitting]
    G --> H{Residuals white noise?}
    H -->|Yes| I[Model is valid]
    H -->|No| J[Re-examine lag structure\nor transform again]
    J --> B
```

## ADF Test (Augmented Dickey-Fuller)
- Null hypothesis: series has a unit root (non-stationary)
- Reject null → evidence of stationarity
- Low p-value (< 0.05) → stationary

> Stationarity is a time series concept — not a requirement in all statistical problems.

**Structural breaks** (policy changes, recessions) cannot be removed by transformation — the process itself changed.

---

# 6. ARIMA(p, d, q)

Three components, each capturing a different source of predictability:

```mermaid
flowchart TD
    ARIMA["ARIMA(p, d, q)"]

    ARIMA --> AR["AR — p\nAutoregression\nMemory of past VALUES\nHow many lags to include"]
    ARIMA --> I["I — d\nIntegrated\nTimes differenced to reach stationarity\nDifferencing ≈ differentiation in calculus\nRecovering predictions ≈ integration"]
    ARIMA --> MA["MA — q\nMoving Average\nMemory of past ERRORS\nShocks that haven't fully faded"]

    AR --> AR2["Recent values predict\nnext value directly"]
    I --> I2["Removes trend so\nprocess is stable"]
    MA --> MA2["Past forecast errors\nreveal lingering shocks"]

    MA2 --> shock["Whether shock lingers\ndepends on its cause:\none-time event vs\nstructural change"]
```

## Component Summary

| Component | Parameter | What it captures |
|---|---|---|
| **AR** | p | How many past values influence today |
| **I** | d | How many differences needed for stationarity |
| **MA** | q | How many past forecast errors influence today |

The "Integrated" terminology comes from calculus: differencing a series is analogous to differentiation; reconstructing the original level from differences is analogous to integration.

---

# 7. ARIMA Limitations

| Limitation | Detail |
|---|---|
| Single long series | No concept of learning across multiple entities |
| Data requirements | Needs 50+ observations for stable parameter estimates |
| Linearity | Assumes linear relationships; non-linear requires manual transformation |
| Structural breaks | Cannot model processes that fundamentally change |
| Panel data | Not designed for many short series with cross-sectional patterns |

**Not suitable for:** short series per entity, retail with many SKUs, externally-driven processes, any setting where cross-series information matters.

---

# 8. Forecasting Algorithm Landscape

```mermaid
flowchart TD
    Root["Forecasting Algorithms"]

    Root --> Mem["Internal Memory Family"]
    Root --> Hyb["Hybrid Family"]
    Root --> Ext["External Drivers Family"]
    Root --> ML["Modern ML Family"]

    Mem --> ARIMA["ARIMA p,d,q\nAR + differencing + MA errors"]
    Mem --> ETS["ETS\nExponential smoothing\nweighted past values"]

    Hyb --> SARIMA["SARIMA\nARIMA extended for\nseasonal cycles"]
    Hyb --> ARIMAX["ARIMAX\nARIMA with external\ninput variables"]

    Ext --> Prophet["Prophet\nTrend + seasonality +\nholiday effects decomposed"]
    Ext --> RegTime["Regression with\ntime features\nTime itself as predictor"]

    ML --> LSTM["LSTM / RNN\nNeural network with\nexplicit memory cells"]
    ML --> XGB["XGBoost\nLag features engineered manually\ngradient boosting"]
    ML --> Trans["Transformer\nAttention mechanism\nover time steps"]
    ML --> NBeats["N-BEATS / NHiTS\nDeep decomposition\nno structural assumptions"]
```

---

# 9. Algorithm–Dataset Pairings

| Algorithm | Natural Dataset | Why It Fits |
|---|---|---|
| **ETS** | Airline passengers (monthly 1949–1960) | Smooth trend + growing seasonality; ETS handles multiplicative seasonality cleanly |
| **ARIMA** | Stock price returns (daily) | Stationary after differencing; internal momentum signal |
| **SARIMA** | Electricity consumption (hourly) | Strong multi-frequency seasonal cycles (daily + weekly) |
| **Prophet** | Wikipedia page views (daily) | Holiday spikes, trend changes, missing data — all handled natively |
| **XGBoost** | Retail sales (daily, many stores) | Many series, external features (promotions, holidays), no long-memory assumption |
| **LSTM** | Weather temperature (hourly) | Long sequences, non-linear patterns, sequential dependencies |
| **Transformers** | Energy demand (multi-site) | Long-range dependencies, attention across many series simultaneously |

---

# 10. Algorithm Selection Decision Flow

```mermaid
flowchart TD
    Start["New forecasting problem"] --> Q1{How many series?}

    Q1 -->|One long series| Q2{External drivers\navailable?}
    Q1 -->|Many series| Q3{Series length\nper entity?}

    Q2 -->|No| Q4{Strong seasonality?}
    Q2 -->|Yes| ARIMAX2["ARIMAX or Prophet"]

    Q4 -->|No| ARIMA2["ARIMA\ncheck stationarity first"]
    Q4 -->|Yes| SARIMA2["SARIMA"]

    Q3 -->|Short < 50 obs| Q5{External features\navailable?}
    Q3 -->|Long| Q6{Non-linear\npatterns?}

    Q5 -->|No| ETS2["ETS or\nnaive baseline"]
    Q5 -->|Yes| XGB2["XGBoost or\nProphet"]

    Q6 -->|No| ARIMA3["ARIMA family"]
    Q6 -->|Yes| ML2["LSTM / Transformer /\nN-BEATS"]
```

---

# 11. Key Principles

1. **Data structure determines algorithm choice** — the series itself tells you which model class applies
2. **Stationarity is the price of admission** for models that exploit temporal order
3. **Achieving stationarity is iterative and messy** — transform, test, transform again
4. **"Personalized forecasting" does not always require separate models per entity** — cross-series learning can outperform entity-specific models on short series
5. **Always ask first:** does my series have internal memory, external drivers, or both?

---

## Quick Reference: Algorithm Properties

| Algorithm | Handles trend | Handles seasonality | External features | Multiple series | Min obs |
|---|---|---|---|---|---|
| ARIMA | via d | No | No | No | 50+ |
| ETS | Yes | Yes | No | No | 20+ |
| SARIMA | via d | Yes | No | No | 50+ |
| ARIMAX | via d | No | Yes | No | 50+ |
| Prophet | Yes | Yes | Yes (holidays) | No | Any |
| XGBoost | via features | via features | Yes | Yes | Any |
| LSTM | Yes | Yes | Yes | Yes | Large |
| Transformer | Yes | Yes | Yes | Yes | Large |
