# Predictability of Monthly Burned Area Across Southeast Asian Fire Regimes (2003–2020)

This repository contains the modelling code and dataset documentation accompanying the study of monthly burned-area predictability across Southeast Asia ($10^\circ\text{S}\text{–}30^\circ\text{N},\ 90^\circ\text{E}\text{–}150^\circ\text{E}$) at $0.25^\circ \times 0.25^\circ$ spatial resolution (7,612 active fire pixels over 2003–2020). Two machine-learning pipelines—a concurrent reconstruction model (`emulator_*.ipynb`) and a one-month-ahead forecaster (`forecaster_*.ipynb`)—are evaluated across 14 sub-regions using a walk-forward expanding-window cross-validation scheme for 2012–2020 against Global Fire Emissions Database version 4 (GFED4s) observations and four benchmark baselines.

## Repository Contents

- [`Code/README.md`](Code/README.md) — Documentation of the two-stage hurdle ensemble architecture, predictor groups, walk-forward evaluation protocol, and exported outputs across the four regional pipelines (eight notebooks):
  - **Mainland Southeast Asia** (Myanmar, Northern Lower Mekong, Southern Lower Mekong): [`emulator_mainland.ipynb`](Code/emulator_mainland.ipynb), [`forecaster_mainland.ipynb`](Code/forecaster_mainland.ipynb)
  - **Maritime West** (Sumatra and Kalimantan Peat, Non-Peat Monsoonal, Non-Peat Savanna): [`emulator_maritime_west.ipynb`](Code/emulator_maritime_west.ipynb), [`forecaster_maritime_west.ipynb`](Code/forecaster_maritime_west.ipynb)
  - **Peninsular Malaysia** (Peat, Non-Peat): [`emulator_pm.ipynb`](Code/emulator_pm.ipynb), [`forecaster_pm.ipynb`](Code/forecaster_pm.ipynb)
  - **Maritime East** (Philippines: Luzon, Visayas, Mindanao; Papua New Guinea: West Savanna, Highlands, Islands East): [`emulator_maritime_east.ipynb`](Code/emulator_maritime_east.ipynb), [`forecaster_maritime_east.ipynb`](Code/forecaster_maritime_east.ipynb)
- [`Data/README.md`](Data/README.md) — Inventory of raw observational and reanalysis datasets (GFED4s, ERA5, MODIS MOD13A3, SoilGrids, and GRACE/GRACE-FO), schema of the compiled $0.25^\circ$ `.npz` archives, and runtime ingestion workflow.

## Data Availability and Execution

Because the compiled `.npz` archives (`~1.88 GB`) and raw source datasets (445 files, `~9.16 GB`) exceed GitHub file-size limits, binary data archives are excluded from version control via `.gitignore`:

1. Download the compiled $0.25^\circ$ archives—`bbur_enhanced_training_ready_seasia_025.npz` (`1,918.78 MB`) and `gfed4s_targets_seasia_025.npz` (`3.28 MB`)—together with the NASA/JPL GRACE/GRACE-FO RL06.3Mv04 CRI mascon NetCDF file (`GRCTellus.JPL.200204_202603.GLO.RL06.3M.MSCNv04CRI.nc`, `43.19 MB`) from the [Google Drive Repository](https://drive.google.com/drive/folders/1JdrqukD05QXRoW2JGfpWtoDG2hE8l3Tv?usp=sharing).
2. Keep all three files in the same directory so that the notebooks ingest the GRACE/GRACE-FO dataset automatically via `GRACE_PATH`.
3. Update `DATASET_PATH` (and `GFED_PATH` or `GFED_TARGET_PATH` where applicable) in the data-loading cell of each notebook to point to the local data directory.

## Software Requirements

Python 3 with `numpy`, `pandas`, `scipy`, `xarray`, `netCDF4`, `cftime`, `scikit-learn`, `catboost`, `xgboost`, `optuna`, `shapely`, `cartopy`, and `matplotlib`.
