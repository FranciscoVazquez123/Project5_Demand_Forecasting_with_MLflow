# Project 5 — Demand Forecasting with MLflow

## Objective

Build a demand forecasting pipeline using a medallion architecture in Databricks, train two models (Prophet and XGBoost), track experiments with MLflow, and compare results to identify the best performing model.

## Dataset

Store Sales Time Series Forecasting — Corporacion Favorita (Ecuador)
Source: Kaggle

| File | Description |
|---|---|
| train.csv | Daily sales by store and product family (3,000,888 records) |
| holidays_events.csv | National and regional holidays and events |
| oil.csv | Daily oil price (WTI) |

## Architecture

```
Source Tables                Bronze              Silver                Gold
workspace.default.train  →  p5_bronze_sales  →                   →  p5_gold_forecast
workspace.default.holidays_events  →  p5_bronze_holidays  →  p5_silver_sales
workspace.default.oil  →  p5_bronze_oil  →
```

## Notebooks

| Notebook | Description |
|---|---|
| P5_00_Config | Global parameters, table references, run ID, model parameters |
| P5_01_Ingest | Bronze layer ingestion with audit columns |
| P5_02_Features | Silver layer: national aggregation, joins, feature engineering |
| P5_03_Prophet | Prophet model training and MLflow logging |
| P5_04_XGBoost | XGBoost model training and MLflow logging |
| P5_05_Compare | Model comparison and visualization |

## Feature Engineering

Applied in the Silver layer:

- National-level aggregation: sales summed across 54 stores by date and product family
- Holiday flag: binary column derived from holidays_events table
- Oil price forward-fill: missing values on weekends filled with last available price
- Calendar features: year, month, dayofweek
- Lag features (XGBoost only): lag_7, lag_14, lag_28
- Rolling means (XGBoost only): rolling_mean_7, rolling_mean_28

## Models

### Prophet

Time series model developed by Meta. Captures trend and seasonality automatically without manual feature engineering. Data prepared in ds/y format as required by the library.

### XGBoost

Supervised ML model trained on engineered features. Requires explicit lag and rolling mean features to capture temporal patterns.

## Results

Target: AUTOMOTIVE product family

| Model | MAE |
|---|---|
| Prophet | 40.32 |
| XGBoost | 15.07 |

Average daily sales for AUTOMOTIVE: 329.47 units. XGBoost achieved a mean absolute error of 15.07, representing approximately 4.6% of average sales. Prophet achieved 40.32, representing approximately 12.2%.

## Experiment Tracking

All runs logged to MLflow with the following parameters and metrics:

| Parameter | Prophet | XGBoost |
|---|---|---|
| family | AUTOMOTIVE | AUTOMOTIVE |
| horizon | 30 | — |
| n_estimators | — | 100 |
| learning_rate | — | 0.1 |

| Metric | Prophet | XGBoost |
|---|---|---|
| MAE | 40.32 | 15.07 |

## Tools

- Databricks Free Edition (Serverless compute)
- PySpark / Delta Lake
- Prophet 1.3.0
- XGBoost
- MLflow
