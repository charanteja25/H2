# Green Hydrogen Demand Forecasting — Databricks ML Pipeline

An end-to-end data engineering and ML pipeline built on **Azure Databricks** to forecast green hydrogen (H₂) demand, integrating real energy market data, ERA5 climate records, and H₂ station data from the Netherlands.

---

## Overview

Green hydrogen production cost and demand are tightly coupled to electricity prices, grid load, and renewable generation mix. This project builds a **medallion architecture pipeline** (Bronze → Silver → Gold) that ingests multi-source energy data, engineers features relevant to H₂ economics, and prepares a training-ready dataset for demand forecasting models.

**Data sources:**
- **ENTSO-E** — hourly electricity prices, grid load, and generation by fuel type (2020–2021)
- **ERA5 (ECMWF)** — hourly weather data (temperature, wind, solar irradiance)
- **H₂ Stations NL** — Netherlands hydrogen refuelling station status and throughput

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        BRONZE LAYER                             │
│   Raw CSV ingestion → Delta format → Schema validation          │
│   ENTSO-E prices · ENTSO-E load · ENTSO-E generation · ERA5    │
│   weather · H2 station records                                  │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────────┐
│                        SILVER LAYER                             │
│   Deduplication · Type standardisation · Null handling          │
│   Timezone normalisation (Europe/Amsterdam)                     │
│   Joined training base: prices + load + generation + weather    │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────────┐
│                         GOLD LAYER                              │
│   generation_dependency_hourly — renewable vs fossil mix        │
│   market_weather_hourly — price × weather feature mart          │
│   h2_station_status_summary — station demand aggregates         │
└─────────────────────────────────────────────────────────────────┘
```

---

## Notebooks

| Notebook | Purpose |
|---|---|
| `00_config.ipynb` | Catalog, schema, and volume configuration |
| `01_uc_bootstrap.ipynb` | Unity Catalog setup and RBAC |
| `02_bronze_ingest.ipynb` | Raw data ingestion to Delta |
| `03_silver_clean.ipynb` | Cleaning, validation, and joins |
| `04_gold_marts.ipynb` | Feature marts for analytics and ML |
| `Feature Engineering.ipynb` | ML feature derivation (lags, rolling stats, renewable share) |
| `06-Stream_Simulation.ipynb` | Streaming simulation for real-time inference testing |
| `99_ops_log.ipynb` | Pipeline run logging and monitoring |

---

## Tech Stack

| Layer | Tools |
|---|---|
| Processing | PySpark, Azure Databricks |
| Storage | Delta Lake, Unity Catalog |
| Data sources | ENTSO-E Transparency Platform API, ERA5 / ECMWF, Open H₂ station data |
| ML prep | Feature engineering, lag features, rolling aggregations |
| Orchestration | Databricks Workflows |

---

## Key Features Engineered

- Hourly renewable generation share (wind + solar as % of total)
- Electricity price lag features (1h, 6h, 24h, 168h)
- Rolling mean and volatility windows for price and load
- Weather-price interaction terms
- H₂ station demand aligned to grid conditions

---

## Results

Pipeline produces a `h2_training_base` Silver table and three Gold analytical marts, ready for time-series forecasting (ARIMA, LightGBM, LSTM) of hourly H₂ demand across Netherlands stations.

---

## Setup

Designed to run on **Azure Databricks** with Unity Catalog enabled. Run notebooks in order: `00 → 01 → 02 → 03 → 04 → Feature Engineering`.

```bash
# Dependencies
pip install -r requirements.txt  # if running locally for exploration
```
