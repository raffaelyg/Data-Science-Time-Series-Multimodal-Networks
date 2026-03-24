# Sales & Demand Forecasting — Hybrid SARIMA-LSTM Ensemble

> **Built a production-grade parallel hybrid SARIMA-LSTM forecasting engine for retail book sales using Nielsen BookScan data (2012–2024). Best model achieved 18.94% MAPE — beating standalone LSTM (24.9%) and XGBoost (29.6%) on volatile weekly demand signals.**

![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python&logoColor=white)
![TensorFlow](https://img.shields.io/badge/TensorFlow-Keras-orange?logo=tensorflow&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3-blue?logo=scikit-learn&logoColor=white)
![pmdarima](https://img.shields.io/badge/pmdarima-AutoARIMA-green)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## Business Problem

**Nielsen BookScan** — the world's largest book sales tracking service — wants to offer small/medium publishers a forecasting tool for data-driven procurement and inventory management. The challenge: predict weekly sales 32 weeks ahead for titles with strong seasonality but volatile demand spikes, enabling optimised stock control and reprint planning.

Two enduring titles were selected as representative case studies:
- **The Alchemist** — volatile, event-driven demand with irregular spikes
- **The Very Hungry Caterpillar** — stable, highly seasonal (Christmas-driven) with predictable annual patterns

---

## Architecture & Approach

```text
                    ┌─────────────┐
                    │  Raw Weekly  │
                    │  Sales Data  │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   Preprocess │  Resample to regular weekly frequency
                    │   & Engineer │  Fill zero-sale weeks, lag features
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼──────┐ ┌──────▼──────┐ ┌───────▼──────┐
   │   SARIMA    │ │   XGBoost   │ │    LSTM      │
   │  (m=52)     │ │  (lag grid) │ │ (KerasTuner) │
   └──────┬──────┘ └──────┬──────┘ └───────┬──────┘
          │                │                │
          └───────┬────────┘                │
                  │                         │
          ┌───────▼─────────────────────────▼──┐
          │         Hybrid Ensembles            │
          │  Sequential: SARIMA + LSTM residual │
          │  Parallel:   Weighted average       │
          └─────────────────────────────────────┘
```

### Models Implemented
1. **Auto ARIMA (SARIMA)** — seasonal period m=52, ADF-confirmed stationarity  
2. **XGBoost Regressor** — grid-searched lag window (4-week & 52-week optimal)  
3. **LSTM Neural Network** — 12-week lookback, KerasTuner hyperparameter optimisation (units, dropout, learning rate)  
4. **Sequential Hybrid** — SARIMA baseline + LSTM trained on SARIMA residuals  
5. **Parallel Hybrid** — weighted ensemble of standalone SARIMA & LSTM forecasts (grid-searched optimal weights)  
6. **Monthly Aggregation** — SARIMA & XGBoost on monthly-resampled data (8-month horizon)  

---

## Results

### Weekly Forecasting (32-week horizon)

| Model | The Alchemist (MAPE) | The Very Hungry Caterpillar (MAPE) |
|---|---|---|
| **Parallel Hybrid (SARIMA+LSTM)** | **23.24%** | **18.94%** ✅ |
| Sequential Hybrid | 23.29% | 21.47% |
| XGBoost | 23.38% | 29.56% |
| SARIMA (monthly) | 32.87% | 19.33% |
| XGBoost (monthly) | 48.07% | 30.23% |
| LSTM (standalone) | 53.31% | 24.94% |

### Optimal Parallel Weights
- The Alchemist: **0.8 SARIMA / 0.2 LSTM**
- The Very Hungry Caterpillar: **0.7 SARIMA / 0.3 LSTM**

---

## Key Findings

1. **Linear patterns dominate book sales.** SARIMA weights of 0.7–0.8 in the best model confirm that annual seasonality and trend are the primary drivers. Deep learning serves as a complementary adjustment, not the forecasting engine.

2. **Standalone LSTM fails on sparse, seasonal data.** The 12-week lookback creates "tunnel vision" — missing the 52-week annual cycle entirely. With only ~600 weekly observations, LSTM overfits short-term noise.

3. **Monthly aggregation smooths volatile titles effectively.** For The Very Hungry Caterpillar, monthly SARIMA (19.33% MAPE) nearly matches the best weekly model — with far less computational overhead.

4. **Volatile titles need exogenous variables to break the error floor.** The Alchemist's ~23% MAPE floor across all models suggests that external factors (media mentions, price promotions) must be integrated for further improvement.

---

## Project Structure

```text
├── nielsen_timeseries_forecasting.ipynb   # Full analysis notebook
├── GAO_YULIANG_CAM_C301_9-10_Report.pdf  # Written report (800-1000 words)
├── requirements.txt                        # Dependencies
├── README.md                               # This file
└── data/                                   # Data loaded from Google Drive at runtime
```

---

## Quick Start

```bash
pip install -r requirements.txt
jupyter notebook nielsen_timeseries_forecasting.ipynb
```

### requirements.txt

```text
pandas>=2.0
numpy>=1.24
matplotlib>=3.7
seaborn>=0.12
statsmodels>=0.14
pmdarima>=2.0
scikit-learn>=1.3
xgboost>=2.0
tensorflow>=2.15
keras-tuner>=1.4
gdown>=5.0
```

---

## Technical Details

- **Data:** Nielsen BookScan UK weekly sales data (2000–2024), filtered to post-2012 for model training  
- **Preprocessing:** ISBN string conversion, datetime indexing, weekly resampling with zero-fill for missing weeks  
- **Stationarity:** Confirmed via Augmented Dickey-Fuller test for both titles  
- **Decomposition:** Additive decomposition revealing trend, 52-week seasonal, and residual components  
- **Evaluation Metrics:** Mean Absolute Error (MAE) and Mean Absolute Percentage Error (MAPE) on held-out final 32 weeks  
- **Hyperparameter Tuning:** KerasTuner (LSTM), GridSearchCV (XGBoost), Auto ARIMA parameter bounds  

---

## About

This project was completed as part of the **University of Cambridge Certificate in Data Science with Machine Learning** (Module C301, Topic 9.1). Analysis and modelling by [Yuliang Raffael Gao](https://linkedin.com/in/raffaelyg/).
