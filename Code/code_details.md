# Southeast Asia Burned-Area Regime Modelling (2003–2020)

Regional hurdle-ensemble pipelines for reconstructing (**Emulator**, concurrent) and predicting (**Forecaster**, 1-month lead) monthly GFED4s burned area ($\text{km}^2$) across Southeast Asia ($10^\circ\text{S}\text{–}30^\circ\text{N},\ 90^\circ\text{E}\text{–}150^\circ\text{E}$).

## Overview

- **Target:** Monthly burned area from GFED4s (2003–2020), evaluated via expanding-window walk-forward validation over 2012–2020 (9 annual folds) restricted to regional fire-season windows (Feb–Apr Mainland, Aug–Nov Maritime West & Papua New Guinea, Jan–Mar Peninsular Malaysia, Mar–May Philippines) plus non-fire-season false-positive rates and 4 benchmark baselines (climatology, persistence, previous-year, and 2003–2011 OLS combination).
- **Predictors:** Shared base predictors and strictly scoped region-specific features from ERA5 surface, 3-layer soil (`swvl1–3`, `stl1–3`), and pressure-level meteorology, climate teleconnections (ENSO/ONI, IOD/DMI), MODIS vegetation indices (NDVI/EVI), GRACE/GRACE-FO terrestrial water storage (`ffill`/`bfill` gap-filled, lags 0, 1, 3, 6, 12), SoilGrids organic carbon & peatland masks, and lagged GFED4s deforestation/cropland fire history. All anomalies and z-scores are standardized on the pre-test training window (2003–2011), followed by two-stage pre-test feature selection (Spearman $|r| > 0.70$ complete-linkage clustering and walk-forward permutation importance $> 0$).
- **Architecture:** Pooled sub-regional two-stage hurdle models:
  1. **Hurdle Classifier** (Class-weighted CatBoost fire/no-fire gate with threshold selected over $0.05\text{–}0.60$, precision $\ge 0.30$, maximizing validation $R^2$ per fold)
  2. **Magnitude Regressor** (Optuna-tuned on a 15% quintile-stratified pixel sample; CatBoost + XGBoost with asymmetric quantile loss $\alpha \in [0.55, 0.65]$ and ExtraTrees with MSE, $\sqrt{\text{BA}}$ sample weights, inverse-validation-RMSE ensemble weights, walk-forward accumulated OOF isotonic calibration with min 200 samples, Maritime West La Niña & previous-month fuel-depletion rules, and leak-free training-OOF Jackknife+ 80% conformal intervals)

## Notebooks

The study domain is partitioned into 4 regional pipelines (covering 14 sub-regions total), each with an **Emulator** and a lag-1 antecedent **1-Month-Lead Forecaster**:

| Region | Sub-regions Covered | Emulator | Lead-1 Forecaster |
|---|---|---|---|
| **Mainland SE Asia** | Myanmar, Northern Lower Mekong, Southern Lower Mekong | `emulator_mainland.ipynb` | `forecaster_mainland.ipynb` |
| **Maritime West** | Sumatra & Kalimantan Peat, Non-Peat Monsoonal (Java/Bali/Sumatra/Kalimantan), Non-Peat Savanna (Sulawesi/Nusa Tenggara/Maluku) | `emulator_maritime_west.ipynb` | `forecaster_maritime_west.ipynb` |
| **Peninsular Malaysia** | Peninsular Malaysia Peat, Peninsular Malaysia Non-Peat | `emulator_pm.ipynb` | `forecaster_pm.ipynb` |
| **Maritime East** | Philippines (Luzon, Visayas, Mindanao), Papua New Guinea (West Savanna, Highlands, Islands East) | `emulator_maritime_east.ipynb` | `forecaster_maritime_east.ipynb` |

## Dataset Ingestion (`.npz` Archives)

All notebooks load a unified $0.25^\circ \times 0.25^\circ$ compiled NumPy archive produced from the raw multi-source satellite/reanalysis datasets (`process_all_kaggle_data_025.py`):
- **`bbur_enhanced_training_ready_seasia_025.npz` (1,918.78 MB / ~1.92 GB on disk, ~4.05 GB in memory):** Loaded via `np.load(..., allow_pickle=True)` to provide `x_flat` (`9,725,184 × 104`, `float32` covering `216 months × 168 lat × 268 lon`), `time_idx`, `lat_flat`, `lon_flat`, `feature_names`, `years_pm`, `lat`, and `lon`. The notebook auto-detects `GRID_RES_DEG` ($0.25^\circ$) from the coordinate spacing, builds explicit 6- and 12-month deep-lag precursors and lag-1 antecedent rolling dryness/NDVI features, and filters to active fire pixels (`BA > 1e-8`, variance `> 1e-10`).
- **`gfed4s_targets_seasia_025.npz` (3.28 MB on disk):** Loaded via `np.load(...)` to extract `y_total_flat` (ground-truth GFED4s burned-area fraction) and overwrite `burned_area_pred` in `x_flat` prior to cosine-latitude $\text{km}^2$ area conversion.

> [!NOTE]
> Because `bbur_enhanced_training_ready_seasia_025.npz` is **1.92 GB** (and the raw NetCDF/GeoTIFF archives exceed **10 GB**), the `.npz` datasets exceed GitHub's 100 MB file-size limit and are excluded via `.gitignore`. See [`Data/data_details.md`](../Data/data_details.md) for the full feature schema and path configuration (`_DATASET_CANDIDATES`).

## Pipeline Outputs

Each notebook exports standardized artifacts (`bbur_{mode}_{region}_*`):
1. **Final Comprehensive Export (`bbur_{mode}_{region}_final_comprehensive.pkl` & `_checkpoint.pkl`):** Per-row predictions, hurdle probabilities/gates, global & sub-regional fire-season evaluation metrics, non-fire-season FPR, 4 benchmark baseline comparisons (RMSE, Spearman $\rho$, skill score), per-pixel metric grids, and monthly time series with leak-free 80% conformal bands.
2. **Grouped Feature Importance (`bbur_{mode}_{region}_grouped_feature_importance_{summary,folds}.csv` / `.pkl`):** Walk-forward grouped permutation importance across the 6 manuscript predictor categories and top-5 individual predictors per sub-region.
3. **Hyperparameters (`bbur_{mode}_{region}_tuned_hyperparameters.csv` / `.pkl`):** Final Optuna-tuned CatBoost/XGBoost parameters, fixed ExtraTrees configuration, quantile $\alpha$, and tuned regional hurdle gates.
