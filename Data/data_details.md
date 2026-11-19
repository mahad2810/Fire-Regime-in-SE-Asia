# Southeast Asia Burned Area Dataset

This document describes the raw observational and reanalysis datasets (**444 files totaling 9,333.40 MB / ~9.33 GB**) and the compiled **0.25° × 0.25° `.npz` training archives** covering Southeast Asia from **January 2003 to December 2020** (216 months).

> [!IMPORTANT]
> **Why the Compiled `.npz` and Raw Datasets Are Not Uploaded to GitHub**
> The multi-source raw archives (~9.33 GB total across 444 NetCDF, GeoTIFF, CSV, and metadata files) were preprocessed and consolidated into two compressed NumPy (`.npz`) archives:
> - `bbur_enhanced_training_ready_seasia_025.npz`: **1,918.78 MB (~1.92 GB)** on disk (~4.05 GB `float32` feature matrix in memory)
> - `gfed4s_targets_seasia_025.npz`: **3.28 MB** on disk
>
> Because the compiled `.npz` dataset (`1.92 GB`) and the raw spatial archives (`9.33 GB`) exceed **GitHub's 100 MB per-file upload limit**, the raw binaries and `.npz` archives could not be uploaded to GitHub and are excluded from version control.
>
> **Download the compiled `.npz` files:** [Google Drive Folder](https://drive.google.com/drive/folders/1JdrqukD05QXRoW2JGfpWtoDG2hE8l3Tv?usp=sharing)

---

## 1. Raw Data Inventory (444 Files, ~9.33 GB Total)

### 1.1 GFED4s — Ground-Truth Burned Area (296 files, 342.89 MB)
Global Fire Emissions Database v4 with small fires (0.25° monthly NetCDF archives):
- **Monthly Global Burned-Area Grids (`BA199701.nc` – `BA202012.nc`, 288 files)**: Monthly files spanning January 1997 to December 2020 containing total burned area fraction and fire-type partitioning (`Peat`, `Defo` [deforestation], `Crop` [agricultural waste burning], `Sava` [savanna], `BorF`, and `TemF`).
- **Regional Subset Aggregates (7 NetCDF files + 1 helper script)**: Pre-extracted regional subsets (`gfed4s_se_asia.nc`, `gfed4s_africa.nc`, `gfed4s_china.nc`, `gfed4s_europe.nc`, `gfed4s_india.nc`, `gfed4s_north_america.nc`, `gfed4s_south_america.nc`) and the deforestation variable standardization utility (`renamedefo.py`).
- **Role in Pipeline**: Provides the ground-truth burned-area target ($y$, fractional burned area) and lagged deforestation/cropland fire memory predictors (`defo_ba_lag3`–`24`, `crop_ba_lag1`–`24`, `years_since_first_defo`, `defo_ba_cumulative_13_24mo`, and 9-/12-month cumulative sums using the 2001–2002 lookback buffer).

### 1.2 ERA5 & ERA5-Land Reanalysis (139 files total: 43 full-domain files [798.32 MB] + 96 Southeast Asia clipped files [120.09 MB])
ECMWF 5th Generation Atmospheric and Land Reanalysis (0.25° monthly NetCDF archives, 2000–2020):
- **Full-Domain Annual Archives (43 files, 798.32 MB)**:
  - 21 annual single-level surface files (`era5_single_level_2000.nc` – `era5_single_level_2020.nc`, 2000–2020)
  - 21 annual pressure-level files (`era5_pressure_levels_2000.nc` – `era5_pressure_levels_2020.nc`, 2000–2020) and 1 verification file (`test_connection.nc`)
- **Southeast Asia Clipped Annual Archives (96 files, 120.09 MB)**:
  - **Single-Level Surface (`era5_single_level_2000_seasia.nc` – `era5_single_level_2020_seasia.nc`, 21 files)**: 2m temperature (`t2m`), 2m dewpoint (`d2m`), surface pressure (`sp`), boundary layer height (`blh`), and 10m wind components (`u10`, `v10`). Also used to derive Vapour Pressure Deficit (`era5_vpd`) and the 3-/6-month cumulative drought stress index (`drought_stress_kbdi_proxy_3mo/6mo`).
  - **Pressure Levels (`era5_pressure_levels_2000_seasia.nc` – `era5_pressure_levels_2020_seasia.nc`, 21 files)**: Lower-tropospheric temperature and horizontal winds at 850 hPa (`t_850hPa`, `u_850hPa`, `v_850hPa`), plus mid-tropospheric vertical velocity and geopotential height at 500 hPa (`w_500hPa`, `z_500hPa`).
  - **Total Precipitation (`era5_precip_2003_seasia.nc` – `era5_precip_2020_seasia.nc`, 18 files)**: Monthly total precipitation (`tp`) evaluated at lags 0, 1, and 3 months.
  - **Topsoil Layer 1 (`era5_soil_2003_seasia.nc` – `era5_soil_2020_seasia.nc`, 18 files)**: Layer-1 (0–7 cm) volumetric soil water (`swvl1`) and soil temperature (`stl1`) at lags 0, 1, and 3 months.
  - **Deep Soil Layers 2 & 3 (`era5_soil_deep_2003_seasia.nc` – `era5_soil_deep_2020_seasia.nc`, 18 files)**: Layer-2 (7–28 cm: `swvl2`, `stl2`) and layer-3 (28–100 cm: `swvl3`, `stl3`) volumetric soil water and temperature at lags 0, 1, and 3 months, plus 6-month cumulative layer-3 soil moisture deficit (`era5_swvl3_deficit_6mo`).

### 1.3 MODIS Vegetation Indices — MOD13A3 v061 (8 files, 6,769.27 MB / ~6.61 GB)
Terra MODIS Monthly 1 km Vegetation Indices:
- **Primary NetCDF Stack (`MOD13A3.061_1km_aid0001.nc`, ~6.61 GB)**: Monthly 1 km NDVI and EVI grids across Southeast Asia.
- **Quality & Metadata Files (7 files)**: VI quality lookup tables (`MOD13A3-061-1-km-monthly-VI-Quality-lookup.csv`), QA summary statistics (`MOD13A3-061-1-km-monthly-VI-Quality-Statistics-QA.csv`, `MOD13A3-061-Statistics.csv`), granule list (`South-East-Asia-granule-list.txt`), XML metadata (`South-East-Asia-MOD13A3-061-metadata.xml`), and request specification (`South-East-Asia-request.json`).
- **Role in Pipeline**: Aggregated to the 0.25° grid to produce raw NDVI and EVI (`ndvi_raw`, `evi_raw`), leak-free standardized monthly anomalies (`ndvi_anom`, `evi_anom` at lags 1–12 mo), cumulative negative vegetation stress (`ndvi_cumdeficit_3mo/6mo/12mo`), month-over-month greenness change (`ndvi_anom_delta`), and drought–vegetation interaction terms (`ndvi_drought_interact`).

### 1.4 Static Soil Organic Carbon — SoilGrids 250 m (1 file, 1,302.83 MB / ~1.27 GB)
- **GeoTIFF Archive (`organic.carbon.stock_msa.kgm2_m_250m_b0t10cm_19500101_20171231_go_epsg.4326_v0.2.tif`)**: Topsoil (0–10 cm) organic carbon stock density in $\text{kg m}^{-2}$ (EPSG:4326).
- **Role in Pipeline**: Resampled to 0.25° (`static_soc`) alongside a peatland extent indicator (`peat_mask`).

---

## 2. Compiled 0.25° `.npz` Datasets (Generated from Raw Data)

All raw variables are aligned onto a unified **0.25° × 0.25° Southeast Asia grid** (`168 lat × 268 lon = 45,024 pixels`) over **216 months** (Jan 2003 – Dec 2020), totaling **9,725,184 spatiotemporal rows** with 0% NaNs (temporal gaps imputed via 2003–2011 pixel historical means; all-null boundary pixels imputed via nearest valid pixels within the same sub-region). All standardized anomalies are computed strictly from the **2003–2011 pre-test training period** (first 108 months).

### 2.1 Feature Matrix (`bbur_enhanced_training_ready_seasia_025.npz` — 1,918.78 MB / 1.92 GB)
- **Dimensions**: `x_flat` of shape `9,725,184 × 104` (`float32`, ~4.05 GB in RAM), plus `y_flat`, `valid_mask`, `is_zero_flux`, `time_idx`, `lat_flat`, `lon_flat`, `feature_names` (`104,`), `years_pm` (`216,`), `lat` (`168,`), `lon` (`268,`).
- **104 Columns in `feature_names`**:
  - **Seasonality & Static Masks (5)**: `ann_sin`, `ann_cos`, `static_soc`, `peat_mask`, `fire_season`
  - **ENSO & Drought Proxies (7)**: `strong_enso`, `oni_lag0`–`oni_lag3`, `drought_stress_kbdi_proxy_3mo/6mo`
  - **ERA5 Surface, Pressure-Level & 3-Layer Soil (58)**: 13 surface/pressure-level variables × 3 lags (0, 1, 3), `era5_vpd` (lags 0, 1, 3), `era5_tp` (lags 0, 1, 3), deep soil layers `era5_swvl2/3` & `era5_stl2/3` × 3 lags (0, 1, 3), and `era5_swvl3_deficit_6mo`
  - **GFED4s Fire Memory & Land-Use History (19)**: `burned_area_pred`, `defo_ba` (lags 3, 6, 12, 18, 24), `years_since_first_defo`, `crop_ba` (lags 1, 2, 3, 6, 12, 18, 24), `defo_ba_cumulative_13_24mo`, and Mainland 9-/12-month cumulative sums (`defo_ba_cumulative_9mo/12mo`, `crop_ba_cumulative_9mo/12mo`)
  - **MODIS Vegetation Dynamics (14)**: `ndvi_raw`, `ndvi_anom` (lags 0, 1, 3, 6, 12 mo), `ndvi_cumdeficit_3mo/6mo/12mo`, `ndvi_anom_delta`, `ndvi_drought_interact`, `evi_raw`, `evi_anom` (lags 0, 1 mo)
  - **Legacy Column Excluded at Notebook Ingestion (1)**: `cams_tc_ch4` (retained for index compatibility, dropped via `BASE_EXCLUDE` in all 8 notebooks)

### 2.2 GFED4s Target Archive (`gfed4s_targets_seasia_025.npz` — 3.28 MB)
- **Contents**: Ground-truth observational burned area fractions aligned row-for-row (`9,725,184` rows) with `x_flat`:
  - `y_total_flat`: Total burned area fraction per 0.25° pixel
  - `y_peat_flat`, `y_defo_flat`, `y_crop_flat`: Partitioned burned area fractions by fire regime
  - `active_pixel_mask`, `time_idx`, `lat_flat`, `lon_flat`, `lat` (`168,`), `lon` (`268,`), `years` (`216,`)
- **Area Conversion**: Converting fractional burned area to physical area ($\text{km}^2$) uses cosine-latitude cell area:
  $$\text{Area (km}^2\text{)} = \text{fraction} \times (0.25 \times 111.32)^2 \times \cos(\text{lat})$$

---

## 3. How the Notebooks Load and Use the `.npz` Archives

In all 8 regional notebooks (`emulator_*.ipynb` and `forecaster_*.ipynb`), the compiled `.npz` files are ingested and processed as follows:

1. **Feature Matrix Extraction**: `np.load(..., allow_pickle=True)` loads `bbur_enhanced_training_ready_seasia_025.npz` and extracts `x_flat`, `y_flat`, `time_idx`, `lat_flat`, `lon_flat`, `feature_names`, `years_pm`, `lat`, and `lon` into `float32`/`int32` arrays, then immediately deletes the `NpzFile` handle and runs `gc.collect()` to free decompressed buffer memory.
2. **Auto-Detected Grid Resolution**: `GRID_RES_DEG` (`0.25°`) is automatically inferred from the median spacing of unique `lat` and `lon` coordinates so downstream cosine-latitude pixel-area conversions ($\text{km}^2$) adapt automatically to the loaded `.npz` dataset.
3. **GFED4s Target Override**: `np.load(...)` loads `y_total_flat` from `gfed4s_targets_seasia_025.npz` and overwrites the `burned_area_pred` column of `x_flat` (`x_flat[:, ba_idx] = y_true_ba`) to guarantee exact alignment with ground-truth GFED4s burned-area fractions.
4. **In-Notebook Deep-Lag & Antecedent Rolling Expansion**: Using the `(N_MONTHS=216, N_PIXELS=45,024)` grid structure of `x_flat`, each notebook constructs 6- and 12-month deep lag precursors (`era5_z_500hPa`, `oni`, `era5_tp`, `era5_vpd`, `burned_area_pred`) and lag-1 antecedent versions of rolling dryness/vegetation-stress features (`_ROLLING_NPZ_COLS`) for leak-free 1-month-lead forecasting.
5. **Active-Pixel & Variance Filtering**: Pixels with maximum historical burned area $\le 10^{-8}$ (`BA_ACTIVE_THRESHOLD`) or temporal variance $\le 10^{-10}$ (`VARIANCE_THRESHOLD`) are masked out before regional sub-setting and walk-forward training.
