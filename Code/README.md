# Modelling Code: Monthly Burned-Area Reconstruction and Forecasting Across Southeast Asia (2003–2020)

Regional two-stage hurdle-ensemble pipelines for reconstructing (concurrent **reconstruction model**, `emulator_*.ipynb`) and predicting (one-month-ahead **forecaster**, `forecaster_*.ipynb`) monthly GFED4s burned area ($\text{km}^2$) at $0.25^\circ \times 0.25^\circ$ spatial resolution across Southeast Asia ($10^\circ\text{S}\text{–}30^\circ\text{N},\ 90^\circ\text{E}\text{–}150^\circ\text{E}$; 7,612 active fire pixels).

## Overview

- **Target and Evaluation Protocol:** Monthly burned area ($\text{km}^2$) from GFED4s (2003–2020), evaluated via walk-forward expanding-window cross-validation across nine annual folds (2012–2020, with the training window expanding from 108 to 204 months). Evaluation is restricted to sub-regional fire-season windows (February–April for Mainland Southeast Asia, August–November for Maritime West and Papua New Guinea, January–March for Peninsular Malaysia, and March–May for the Philippines), supplemented by non-fire-season false-positive rates and benchmarked against four reference baselines (pixel-month climatology, persistence, previous-year recurrence, and a 2003–2011 ordinary least squares combination).
- **Predictors and Feature Selection:** Comprises 86 shared base predictors plus region-specific geographic, seasonal-phase, and physical features derived from ERA5 surface, three-layer soil (`swvl1–3`, `stl1–3`), and pressure-level (850 hPa and 500 hPa) meteorology; climate teleconnections (Oceanic Niño Index and Indian Ocean Dipole / Dipole Mode Index); MODIS vegetation indices (NDVI and EVI); GRACE/GRACE-FO terrestrial water storage anomalies (gap-filled via forward-filling followed by backward-filling, at lags 0, 1, 3, 6, and 12 months plus 6-month trend); SoilGrids organic carbon stock and peatland masks; and lagged GFED4s deforestation and cropland fire history. Pre-computed `.npz` anomalies and GRACE terrestrial water storage are standardized over the 2003–2011 training period, and two-stage feature selection (complete-linkage Spearman correlation clustering with $|r| > 0.70$ and walk-forward permutation importance $> 0$) is executed strictly within the 2003–2011 pre-test training period (using 2009–2011 held-out folds for permutation importance).
- **Model Architecture:** Pooled sub-regional two-stage hurdle models:
  1. **Fire-Occurrence Classifier:** Class-weighted CatBoost binary classifier with a sub-region-specific probability threshold selected per fold over $0.05\text{–}0.60$ (subject to precision $\ge 0.30$) to maximize validation $R^2$ on de-normalized targets.
  2. **Conditional Magnitude Regressor:** Ensemble of CatBoost, XGBoost (asymmetric quantile loss, $\alpha \in [0.55, 0.65]$), and ExtraTrees (mean-squared-error loss), tuned via Optuna on a 15% quintile-stratified pixel sample with sample weights proportional to the square root of observed burned area and inverse-validation-RMSE ensemble weights. Predictions undergo walk-forward out-of-fold isotonic recalibration (minimum 200 calibration samples), post-processing rules for La Niña suppression and prior-month fuel depletion in Maritime West, and leak-free Jackknife+ 80% conformal prediction intervals computed from pre-test training residuals.

## Regional Notebooks

The study domain is partitioned into four regional pipelines encompassing 14 sub-regions, each implemented as a concurrent reconstruction model (**Emulator**) and a one-month-ahead **Forecaster**:

| Region | Sub-Regions Covered | Reconstruction Model | One-Month-Ahead Forecaster |
|---|---|---|---|
| **Mainland Southeast Asia** | Myanmar, Northern Lower Mekong, Southern Lower Mekong | `emulator_mainland.ipynb` | `forecaster_mainland.ipynb` |
| **Maritime West** | Sumatra and Kalimantan Peat, Non-Peat Monsoonal (Java, Bali, and non-peat Kalimantan/Sumatra), Non-Peat Savanna (Sulawesi, Nusa Tenggara, Maluku) | `emulator_maritime_west.ipynb` | `forecaster_maritime_west.ipynb` |
| **Peninsular Malaysia** | Peninsular Malaysia Peat, Peninsular Malaysia Non-Peat | `emulator_pm.ipynb` | `forecaster_pm.ipynb` |
| **Maritime East** | Philippines (Luzon, Visayas, Mindanao), Papua New Guinea (West Savanna, Highlands, Islands East) | `emulator_maritime_east.ipynb` | `forecaster_maritime_east.ipynb` |

## Data Ingestion

All notebooks ingest the compiled $0.25^\circ \times 0.25^\circ$ NumPy archives alongside the GRACE/GRACE-FO mascon NetCDF file from the same base directory (`_BASE`):
- **`bbur_enhanced_training_ready_seasia_025.npz` (1,918.78 MB / ~1.87 GB on disk; ~4.05 GB in memory):** Loaded via `np.load(..., allow_pickle=True)` to extract `x_flat` (`9,725,184 × 104`, `float32`, spanning `216 months × 168 latitude × 268 longitude`), `time_idx`, `lat_flat`, `lon_flat`, `feature_names`, `years_pm`, `lat`, and `lon`. The notebooks construct 6- and 12-month deep-lag precursors and one-month antecedent rolling dryness/vegetation-stress predictors, then filter to active fire pixels (maximum burned area $> 10^{-8}$, temporal variance $> 10^{-10}$).
- **`gfed4s_targets_seasia_025.npz` (3.28 MB on disk):** Loaded via `np.load(...)` to extract `y_total_flat` (observed GFED4s burned-area fraction) and overwrite `burned_area_pred` in `x_flat` prior to cosine-latitude conversion to physical area ($\text{km}^2$).
- **`GRCTellus.JPL.200204_202603.GLO.RL06.3M.MSCNv04CRI.nc` (43.19 MB on disk, located in `Raw_Data`):** Loaded via `xarray` (`lwe_thickness` multiplied by `scale_factor`), gap-filled across inter-mission gaps via temporal forward-filling followed by backward-filling, spatially interpolated to the $0.25^\circ$ grid, standardized on 2003–2011, and appended as seven `grace_tws_*` predictors (alongside region-specific drawdown and peat-interaction terms in Maritime West and Peninsular Malaysia). This file must reside in the same directory as the `.npz` archives during execution.

> [!NOTE]
> Because `bbur_enhanced_training_ready_seasia_025.npz` (~1.87 GB) and the 445 raw source files (~9.16 GB total) exceed GitHub file-size limits, they are excluded from version control. Both compiled `.npz` archives and `GRCTellus.JPL.200204_202603.GLO.RL06.3M.MSCNv04CRI.nc` are available via the [Google Drive Repository](https://drive.google.com/drive/folders/1JdrqukD05QXRoW2JGfpWtoDG2hE8l3Tv?usp=sharing) (and locally in `Raw_Data`). Refer to [`Data/README.md`](../Data/README.md) for the complete raw data inventory and 104-feature schema.

## Pipeline Outputs

Each notebook exports standardized output files (`bbur_{mode}_{region}_*`):
1. **Comprehensive Evaluation Archive (`bbur_{mode}_{region}_final_comprehensive.pkl` and `_checkpoint.pkl`):** Row-level predictions, hurdle probabilities and selected gates, domain-wide and sub-regional fire-season evaluation metrics, non-fire-season false-positive rates, four benchmark baseline comparisons (RMSE, Spearman $\rho$, and skill score), per-pixel metric grids, and monthly time series with leak-free Jackknife+ conformal prediction intervals.
2. **Grouped Feature Importance (`bbur_{mode}_{region}_grouped_feature_importance_{summary,folds}.csv` / `.pkl`):** Walk-forward grouped permutation importance across the six predictor categories and top five individual predictors per sub-region.
3. **Selected Hyperparameters (`bbur_{mode}_{region}_tuned_hyperparameters.csv` / `.pkl`):** Optuna-selected CatBoost and XGBoost parameters, ExtraTrees configuration, regional quantile $\alpha$, and calibrated hurdle probability thresholds.
