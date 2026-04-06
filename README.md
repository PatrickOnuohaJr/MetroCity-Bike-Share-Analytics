# MetroCity Bike Share Analytics

An end-to-end data engineering pipeline and operational analytics project analyzing 700K+ bike share trips across the UT Austin MetroBike network. Built on a Databricks medallion architecture (Bronze → Silver → Gold), with Snowflake integration and AWS SageMaker demand forecasting in progress.

Originally developed as a DSCI 4700 capstone at the University of North Texas in partnership with CapMetro. Rebuilt as a modern DE/DS portfolio project using enterprise-grade tooling.

---

## The Problem

In 2023, UT Austin campus kiosks experienced **8.75% unavailability** — meaning 9 out of every 100 trips failed because students arrived at a kiosk that was either completely empty or completely full. This resulted in approximately **25,860 interrupted trips annually**.

The root cause: kiosks were rebalanced on a rigid schedule (midnight and noon) at 50% dock capacity — optimizing for no one.

**Goal:** Reduce unavailability below 3% without excessive capital expenditure.

---

## The Solution

Using a manual grid search A/B testing framework across three optimization levers — reset timing, reset capacity, and dock expansion — two solutions were identified:

| | Baseline | Solution 1 | Solution 2 |
|---|---|---|---|
| **Reset Schedule** | 12am / 12pm | **10am / 3pm** | 10am / 3pm |
| **Reset Capacity** | 50% | **60%** | 60% |
| **Dock Expansion** | — | None | +25 docks (5 high-risk kiosks) |
| **Unavailability** | 8.75% | **3.38%** | **2.18%** |
| **Trips Recovered** | — | ~16,400/year | ~19,850/year |
| **CapEx Required** | — | $0 | ~$42K |

**Solution 1 is the recommended approach** — it captures 85% of the available improvement at zero capital cost by simply changing when dispatch crews perform resets.

---

## Key Findings

- **Timing beats infrastructure.** Changing reset hours from midnight/noon to 10am/3pm reduced unavailability from 8.75% to 3.65% — a 60% reduction — with zero investment.
- **Capacity adjustments alone are insufficient.** Even at optimal 60% capacity, unavailability only dropped from 8.75% to 8.31% — less than half a percentage point.
- **Dock expansion is a secondary lever.** Adding 25 docks improved unavailability by 1.33 points at significant cost — far less impactful than timing changes.
- **Combined Solution 1** (10am/3pm + 60% capacity) achieves **3.38% unavailability**, recovering ~16,400 trips and projecting $44K+ annual profit by Year 2.

---

## Pipeline Architecture

```
Raw CSV Data (269,680 trip records, 2022–2024)
        │
        ▼
┌─────────────────────────────────────┐
│           BRONZE LAYER              │
│  Raw ingestion — no transformations │
│  bronze_trips | bronze_kiosks       │
└─────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────┐
│           SILVER LAYER              │
│  Cleaned, typed, enriched           │
│  silver_trips — parsed timestamps,  │
│  duration validation, school season │
│  silver_kiosks — lat/lon parsed,    │
│  kiosk dimension table              │
└─────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────┐
│            GOLD LAYER               │
│  Aggregated, analysis-ready         │
│  gold_station_balance_hourly —      │
│  hourly checkouts, returns,         │
│  net_flow, imbalance_ratio,         │
│  is_peak, is_weekend per kiosk      │
└─────────────────────────────────────┘
        │
        ▼
┌─────────────────┐    ┌──────────────────────┐
│    Snowflake    │    │    AWS SageMaker      │
│  Cloud export   │    │  Demand forecasting   │
│  (connector     │    │  on Gold layer data   │
│   documented)   │    │  (in progress)        │
└─────────────────┘    └──────────────────────┘
```

---

## Notebook Structure

| Notebook | Type | Description |
|---|---|---|
| `00_setup_database` | SQL | Creates `metrobike` database in Databricks |
| `01_bronze_ingestion` | SQL | Raw data inspection — kiosks and trips |
| `02_bronze_schema_check` | SQL | Schema validation — confirms 14-column structure |
| `03_silver_trips` | SQL | Cleans timestamps, duration, builds silver trips layer |
| `04_silver_kiosks` | SQL | Kiosk dimension table with lat/lon parsing |
| `05_gold_station_balance_hourly` | SQL | Main Gold table — hourly imbalance metrics per kiosk |
| `06_gold_validation` | SQL | Gold layer validation and imbalance analysis |
| `07_snowflake_export` | Python | Snowflake connector + Gold layer CSV export |

---

## Dataset

- **Source:** City of Austin MetroBike (via CapMetro)
- **Scope:** 14 campus kiosks within 1-mile radius of UT Austin
- **Time Period:** January 2022 – July 2024
- **Records:** 269,680 trip records (567,930 checkout/return events)
- **Focus:** Weekday peak hours (8am–6pm), active school sessions, trips 3–60 minutes

---

## Tech Stack

| Layer | Technology |
|---|---|
| Pipeline & Transformation | Databricks (SQL + PySpark) |
| Cloud Data Warehouse | Snowflake (connector implemented) |
| Demand Forecasting | AWS SageMaker (in progress) |
| Languages | SQL, Python |
| Libraries | PySpark, Pandas, Snowflake Connector |
| Version Control | GitHub |

---

## Financial Impact (Solution 1)

| Metric | Value |
|---|---|
| 2023 Baseline Revenue | $57,620 |
| 2023 Projected Revenue (Solution 1) | $61,130 |
| 2024 Predicted Revenue | $96,302 |
| Year 2+ Annual Profit | ~$44,100 |
| 5-Year Net Return | ~$137,000 |
| Payback Period | ~2 years |

---

## Status

- [x] Bronze layer — raw ingestion and inspection
- [x] Silver layer — cleaned trips and kiosk dimension
- [x] Gold layer — hourly station balance metrics
- [x] Snowflake connector — documented and implemented
- [ ] AWS SageMaker demand forecasting — in progress
- [ ] Visualization dashboard

---

## Author

**Patrick Onuoha Jr.**  
B.S. Data Science | University of North Texas  
[GitHub](https://github.com/PatrickOnuohaJr) | [Churn Prediction Project](https://github.com/PatrickOnuohaJr/Churn-Prediction)
