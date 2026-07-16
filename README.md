# Predicting Station-Level Bike Availability in Vienna: A Layered Profile Model

Course project for *Applications of Data Analytics* (SS 2026, TU Dresden).
Authors: **Rui Dai & Shuimei Jin**.

We forecast the number of available bikes at 235 stations in Vienna's Nextbike
system at 30-minute resolution (Kaggle competition, metric = RMSLE). Training
covers 2024-09-03 → 2025-03-13; the hidden test set is the following ~7 weeks
(2025-03-13 → 2025-04-29), split into a near **private** half and a warmer
**public** half.

**Final public-leaderboard score: 1.10944 and 1.10946.**

---

## Repository contents

| File | Description |
|---|---|
| `bda_report_project2_ruidai_shuimeijin_ss26.ipynb` | Main report notebook (R kernel). Full workflow: EDA → XGBoost attempt → final layered model → evaluation. Running it top-to-bottom writes `submission.csv`. |
| `weather_vienna.rds` | **Frozen** hourly weather cache (open-meteo), shipped for reproducibility. |
| `gtfs_transit_features.rds`| **Frozen** GTFS-derived transit features (Austrian public-transport timetable), shipped for reproducibility. |
| `station_elevation.rds`| **Frozen** per-station elevation cache (open-meteo elevation API), shipped for reproducibility.  |
| `submission.csv` | The graded prediction file. |
| `submission_2.csv` | The graded prediction file with a different shape parameter. |

`train.csv` / `test.csv` are the competition data files (from Kaggle).

---


### R package dependencies

```r
install.packages(c("data.table", "xgboost", "timeDate", "ggplot2",
                   "jsonlite", "suncalc"))
```

(`suncalc` and `jsonlite` are only used in the EDA section.)

---

## Model in one line

A decomposable additive stack in log space:

- **L1–L3 (shared base):** a zero-inflated, recency-weighted, hierarchically
  shrunk per-station × time-of-week profile, blended across the forecast horizon
  with far-side mean reversion. *(No machine learning — this base alone scores
  0.577 / 0.671 locally.)*
- **E3 (private half):** a small, damped XGBoost residual correction, applied
  only to the near, in-range horizon.
- **L4/L5 + E1/E2 (public half):** for the warm, out-of-range April window, an
  extrapolatable linear temperature law with a leaderboard-anchored level shift,
  plus a temperature-driven intraday shape and calendar (holiday / school-break)
  extensions.
