# Predicting Station-Level Bike Availability in Vienna: A Layered Profile Model

Course project for *Applications of Data Analytics* (SS 2026, TU Dresden).
Authors: **Rui Dai & Shuimei Jin**.

We forecast the number of available bikes at 235 stations in Vienna's Nextbike
system at 30-minute resolution (Kaggle competition, metric = RMSLE). Training
covers 2024-09-03 → 2025-03-13; the hidden test set is the following ~7 weeks
(2025-03-13 → 2025-04-29), split into a near **private** half and a warmer
**public** half.

**Final public-leaderboard score: 1.10944.**

---

## Repository contents

| File | Description |
|---|---|
| `final-version.ipynb` | Main report notebook (R kernel). Full workflow: EDA → XGBoost attempt → final layered model → evaluation. Running it top-to-bottom writes `submission.csv`. |
| `model_b5r_a309.R` | Standalone R script of the final model (same pipeline as the notebook). |
| `model_b5r_tune_standalone.R` | Two-regime hyperparameter search for the base layers (L1–L3). Optional. |
| `weather_vienna.rds` | **Frozen** hourly weather cache (open-meteo), shipped for reproducibility. |
| `gtfs_transit_features.rds`| **Frozen** hourly weather cache (open-meteo), shipped for reproducibility. |
| `station_elevation.rds`| **Frozen** hourly weather cache (open-meteo), shipped for reproducibility. |
| `submission.csv` | The graded prediction file. |

`train.csv` / `test.csv` are the competition data files (from Kaggle).

---

## How to run

The notebook uses an **R kernel** (`ir`). It runs on Kaggle or locally.

**On Kaggle**
1. Set the notebook language to **R**, and turn **Internet OFF** (see the weather note below).
2. Attach the competition data and the `weather_vienna.rds` dataset as inputs
   (they appear under `/kaggle/input/...`).
3. **Run All.** The final cell writes `submission.csv` to `/kaggle/working/`.

**Locally**
1. Put `train.csv`, `test.csv`, and `weather_vienna.rds` in the working directory
   (or `setwd()` to their folder).
2. Open the notebook in Jupyter with an R kernel (or run `model_b5r_a309.R`).
3. Run all cells → `submission.csv` is written to the working directory.

### R package dependencies

```r
install.packages(c("data.table", "xgboost", "timeDate", "ggplot2",
                   "jsonlite", "suncalc"))
```

(`suncalc` and `jsonlite` are only used in the EDA section.)

---

## Reproducibility note — the weather cache

The public-half prediction (layers L4/L5) depends on **smoothed temperature**.
Open-meteo periodically revises its historical reanalysis, so **fetching fresh
weather gives a slightly different score**. To reproduce 1.10921 exactly, the
notebook reads the **frozen** `weather_vienna.rds` shipped in this repo and does
**not** re-fetch. Do not delete or overwrite this file, and keep Internet off on
Kaggle so no fresh fetch occurs.

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

The final integer prediction is `max(0, round(exp(prediction) - 1))`.
