# Southeast Asia Burned-Area Predictability Dataset (2003–2020)

This document details the raw observational and reanalysis datasets (**445 files totalling 9,376.59 MB / ~9.16 GB**) and the compiled **$0.25^\circ \times 0.25^\circ$ `.npz` archives** covering Southeast Asia ($10^\circ\text{S}\text{–}30^\circ\text{N},\ 90^\circ\text{E}\text{–}150^\circ\text{E}$) from **January 2003 to December 2020** (216 months).

> [!IMPORTANT]
> **External Data Hosting**
> Because the raw spatial archives (~9.16 GB across 445 NetCDF, GeoTIFF, CSV, and metadata files) and the primary compiled feature matrix (~1.87 GB) exceed GitHub file-size limits, binary data files are excluded from version control. Preprocessed predictors and targets are consolidated into two compressed NumPy (`.npz`) archives, while GRACE terrestrial water storage is ingested directly at runtime:
> - `bbur_enhanced_training_ready_seasia_025.npz`: **1,918.78 MB (~1.87 GB)** on disk (~4.05 GB `float32` feature matrix in memory)
> - `gfed4s_targets_seasia_025.npz`: **3.28 MB** on disk
> - `GRCTellus.JPL.200204_202603.GLO.RL06.3M.MSCNv04CRI.nc`: **43.19 MB** on disk (located in `Raw_Data`; must be placed in the same directory as the `.npz` archives when executing the notebooks)
>
> **Compiled `.npz` archives and GRACE/GRACE-FO dataset:** [Google Drive Repository](https://drive.google.com/drive/folders/1JdrqukD05QXRoW2JGfpWtoDG2hE8l3Tv?usp=sharing)

---

## 1. Raw Data Inventory (445 Files, ~9.16 GB Total)

### 1.1 GFED4s — Burned Area Target and Fire-Type Partitioning (296 files, 342.89 MB)
Global Fire Emissions Database version 4 with small fires ($0.25^\circ$ monthly NetCDF archives):
- **Monthly Global Burned-Area Grids (`BA199701.nc` – `BA202012.nc`, 288 files):** Monthly grids spanning January 1997 to December 2020 containing total burned-area fraction and fire-type partitioning (`Peat`, `Defo` [deforestation], `Crop` [agricultural waste burning], `Sava` [savanna], `BorF`, and `TemF`).
- **Regional Subset Aggregates (7 NetCDF files and 1 utility script):** Pre-extracted regional subsets (`gfed4s_se_asia.nc`, `gfed4s_africa.nc`, `gfed4s_china.nc`, `gfed4s_europe.nc`, `gfed4s_india.nc`, `gfed4s_north_america.nc`, `gfed4s_south_america.nc`) and `renamedefo.py`.
- **Role in Pipeline:** Supplies the ground-truth burned-area target ($y$, fractional burned area) and lagged deforestation and cropland fire-history predictors (`defo_ba_lag3`–`24`, `crop_ba_lag1`–`24`, `years_since_first_defo`, `defo_ba_cumulative_13_24mo`, and 9- and 12-month cumulative sums computed with the 2001–2002 lookback buffer).

### 1.2 ERA5 Reanalysis (139 files: 43 full-domain files [798.32 MB] + 96 Southeast Asia files [120.09 MB])
Fifth-generation ECMWF atmospheric reanalysis ($0.25^\circ$ monthly NetCDF archives, 2000–2020):
- **Full-Domain Annual Archives (43 files, 798.32 MB):** 21 annual single-level files (`era5_single_level_2000.nc` – `era5_single_level_2020.nc`), 21 annual pressure-level files (`era5_pressure_levels_2000.nc` – `era5_pressure_levels_2020.nc`), and one connection verification file (`test_connection.nc`).
- **Southeast Asia Clipped Annual Archives (96 files, 120.09 MB):**
  - **Single-Level Surface (`era5_single_level_2000_seasia.nc` – `era5_single_level_2020_seasia.nc`, 21 files):** 2-metre air temperature (`t2m`), 2-metre dewpoint temperature (`d2m`), surface pressure (`sp`), boundary layer height (`blh`), and 10-metre wind components (`u10`, `v10`), used directly and to derive vapour pressure deficit (`era5_vpd`) and 3- and 6-month cumulative drought-stress indices (`drought_stress_kbdi_proxy_3mo/6mo`).
  - **Pressure Levels (`era5_pressure_levels_2000_seasia.nc` – `era5_pressure_levels_2020_seasia.nc`, 21 files):** Lower-tropospheric temperature and horizontal winds at 850 hPa (`t_850hPa`, `u_850hPa`, `v_850hPa`) and mid-tropospheric vertical velocity and geopotential height at 500 hPa (`w_500hPa`, `z_500hPa`).
  - **Total Precipitation (`era5_precip_2003_seasia.nc` – `era5_precip_2020_seasia.nc`, 18 files):** Monthly total precipitation (`tp`) at 0-, 1-, and 3-month lags.
  - **Topsoil Layer 1 (`era5_soil_2003_seasia.nc` – `era5_soil_2020_seasia.nc`, 18 files):** Layer-1 (0–7 cm) volumetric soil water (`swvl1`) and soil temperature (`stl1`) at 0-, 1-, and 3-month lags.
  - **Deep Soil Layers 2 and 3 (`era5_soil_deep_2003_seasia.nc` – `era5_soil_deep_2020_seasia.nc`, 18 files):** Layer-2 (7–28 cm: `swvl2`, `stl2`) and layer-3 (28–100 cm: `swvl3`, `stl3`) volumetric soil water and soil temperature at 0-, 1-, and 3-month lags, plus the 6-month cumulative layer-3 soil-moisture deficit (`era5_swvl3_deficit_6mo`).

### 1.3 MODIS Vegetation Indices — MOD13A3 v061 (8 files, 6,769.27 MB / ~6.61 GB)
Terra MODIS monthly 1 km vegetation indices:
- **Primary NetCDF Stack (`MOD13A3.061_1km_aid0001.nc`, ~6.61 GB):** Monthly 1 km NDVI and EVI grids across Southeast Asia.
- **Quality and Metadata Files (7 files):** Quality lookup table (`MOD13A3-061-1-km-monthly-VI-Quality-lookup.csv`), QA summary statistics (`MOD13A3-061-1-km-monthly-VI-Quality-Statistics-QA.csv`, `MOD13A3-061-Statistics.csv`), granule list (`South-East-Asia-granule-list.txt`), XML metadata (`South-East-Asia-MOD13A3-061-metadata.xml`), and request specification (`South-East-Asia-request.json`).
- **Role in Pipeline:** Aggregated to $0.25^\circ$ to derive raw NDVI and EVI (`ndvi_raw`, `evi_raw`), standardized monthly anomalies (`ndvi_anom`, `evi_anom` at lags up to 12 months), cumulative negative vegetation stress (`ndvi_cumdeficit_3mo/6mo/12mo`), month-over-month greenness change (`ndvi_anom_delta`), and vegetation–drought interaction terms (`ndvi_drought_interact`).

### 1.4 Static Soil Organic Carbon — SoilGrids 250 m (1 file, 1,302.83 MB / ~1.27 GB)
- **GeoTIFF Archive (`organic.carbon.stock_msa.kgm2_m_250m_b0t10cm_19500101_20171231_go_epsg.4326_v0.2.tif`):** Topsoil (0–10 cm) organic carbon stock density ($\text{kg m}^{-2}$, EPSG:4326).
- **Role in Pipeline:** Resampled to $0.25^\circ$ (`static_soc`) alongside a peatland extent indicator (`peat_mask`).

### 1.5 GRACE / GRACE-FO Terrestrial Water Storage (1 file, 43.19 MB)
- **NetCDF Archive (`GRCTellus.JPL.200204_202603.GLO.RL06.3M.MSCNv04CRI.nc`):** NASA/JPL GRACE and GRACE-FO RL06.3Mv04 CRI mascon solution on a $360 \times 720$ global $0.5^\circ$ grid (255 monthly timesteps, April 2002–March 2026), providing liquid water equivalent thickness (`lwe_thickness`, cm) and `scale_factor`.
- **Role in Pipeline:** Loaded directly at notebook runtime from the same directory as the `.npz` archives, spatially interpolated to the $0.25^\circ$ grid, gap-filled across inter-mission gaps via temporal forward-filling followed by backward-filling, standardized over the 2003–2011 training period, and appended to `x_flat` as seven predictors (`grace_tws_lag0`, `grace_tws_lag1`, `grace_tws_lag3`, `grace_tws_lag6`, `grace_tws_lag12`, `grace_tws_trend_6mo`, `grace_tws_trend_6mo_lag1`), plus region-specific drawdown and peat-interaction terms in Maritime West and Peninsular Malaysia.

---

## 2. Compiled $0.25^\circ$ `.npz` Archives

All preprocessed variables are aligned onto a unified **$0.25^\circ \times 0.25^\circ$ Southeast Asia grid** (`168 latitude × 268 longitude = 45,024 pixels`) over **216 months** (January 2003–December 2020), yielding **9,725,184 spatiotemporal rows** without missing values (residual coastal or cloud-masked nulls are imputed using each pixel's 2003–2011 historical mean, or the spatial mean of nearest valid pixels within the same sub-region if null across all months). Standardized anomalies are computed strictly over the **2003–2011** baseline (first 108 months).

### 2.1 Feature Matrix (`bbur_enhanced_training_ready_seasia_025.npz` — 1,918.78 MB / ~1.87 GB)
- **Arrays:** `x_flat` of shape `9,725,184 × 104` (`float32`, ~4.05 GB in memory), `y_flat`, `valid_mask`, `is_zero_flux`, `time_idx`, `lat_flat`, `lon_flat`, `feature_names` (`104,`), `years_pm` (`216,`), `lat` (`168,`), and `lon` (`268,`).
- **104 Columns in `feature_names`:**
  - **Seasonality and Static Land-Surface Masks (5):** `ann_sin`, `ann_cos`, `static_soc`, `peat_mask`, `fire_season`
  - **ENSO and Drought-Stress Proxies (7):** `strong_enso`, `oni_lag0`–`oni_lag3`, `drought_stress_kbdi_proxy_3mo/6mo`
  - **ERA5 Surface, Pressure-Level, and Three-Layer Soil (58):** 13 surface and pressure-level variables across three lags (0, 1, 3 months), `era5_vpd` (lags 0, 1, 3), `era5_tp` (lags 0, 1, 3), deep soil layers `era5_swvl2/3` and `era5_stl2/3` across three lags (0, 1, 3), and `era5_swvl3_deficit_6mo`
  - **GFED4s Fire Memory and Land-Use History (19):** `burned_area_pred`, `defo_ba` (lags 3, 6, 12, 18, 24), `years_since_first_defo`, `crop_ba` (lags 1, 2, 3, 6, 12, 18, 24), `defo_ba_cumulative_13_24mo`, and 9- and 12-month cumulative sums (`defo_ba_cumulative_9mo/12mo`, `crop_ba_cumulative_9mo/12mo`)
  - **MODIS Vegetation State (14):** `ndvi_raw`, `ndvi_anom` (lags 0, 1, 3, 6, 12 months), `ndvi_cumdeficit_3mo/6mo/12mo`, `ndvi_anom_delta`, `ndvi_drought_interact`, `evi_raw`, `evi_anom` (lags 0, 1 month)
  - **Excluded Legacy Column (1):** `cams_tc_ch4` (retained for column-index compatibility and excluded via `BASE_EXCLUDE` in all eight notebooks)

### 2.2 GFED4s Target Archive (`gfed4s_targets_seasia_025.npz` — 3.28 MB)
- **Arrays:** Ground-truth observational burned-area fractions aligned row-by-row (`9,725,184` rows) with `x_flat`: `y_total_flat` (total burned-area fraction), partitioned fractions (`y_peat_flat`, `y_defo_flat`, `y_crop_flat`), `active_pixel_mask`, `time_idx`, `lat_flat`, `lon_flat`, `lat` (`168,`), `lon` (`268,`), and `years` (`216,`).
- **Physical Area Conversion:** Fractional burned area is converted to physical area ($\text{km}^2$) using cosine-latitude grid-cell area:
  $$\text{Area (km}^2\text{)} = \text{fraction} \times (0.25 \times 111.32)^2 \times \cos(\text{lat})$$

---

## 3. Data Ingestion and Preprocessing Within the Notebooks

Across all eight regional notebooks (`emulator_*.ipynb` and `forecaster_*.ipynb`), the archives are ingested in six steps:

1. **Feature Matrix Extraction:** `np.load(..., allow_pickle=True)` reads `bbur_enhanced_training_ready_seasia_025.npz` into `float32`/`int32` arrays (`x_flat`, `y_flat`, `time_idx`, `lat_flat`, `lon_flat`, `feature_names`, `years_pm`, `lat`, `lon`), closes the archive handle, and invokes `gc.collect()`.
2. **Grid Resolution Detection:** `GRID_RES_DEG` (`0.25°`) is determined from the median coordinate spacing of unique `lat` and `lon` values to parameterize cosine-latitude pixel-area conversions ($\text{km}^2$).
3. **GFED4s Target Override:** `np.load(...)` reads `y_total_flat` from `gfed4s_targets_seasia_025.npz` and replaces the `burned_area_pred` column in `x_flat` to ensure exact alignment with ground-truth GFED4s observations.
4. **GRACE Terrestrial Water Storage Ingestion:** When `GRCTellus.JPL.200204_202603.GLO.RL06.3M.MSCNv04CRI.nc` is present in the same base directory as the `.npz` files, `xarray` loads `lwe_thickness` multiplied by `scale_factor`, fills inter-mission gaps via temporal forward-filling followed by backward-filling, interpolates onto the $0.25^\circ$ grid, standardizes on 2003–2011, and appends the seven `grace_tws_*` predictors to `x_flat`.
5. **Deep-Lag and Antecedent Rolling Expansion:** Using the `(216 months, 45,024 pixels)` grid structure of `x_flat`, each notebook constructs 6- and 12-month deep-lag precursors (`era5_z_500hPa`, `oni`, `era5_tp`, `era5_vpd`, `burned_area_pred`) and one-month antecedent equivalents of rolling dryness and vegetation-stress features (`_ROLLING_NPZ_COLS`) for leak-free forecasting.
6. **Active-Pixel Filtering:** Non-terrestrial and zero-variance pixels (maximum burned area $\le 10^{-8}$ or temporal variance $\le 10^{-10}$) are masked out prior to sub-regional partitioning and walk-forward training, retaining 7,612 active fire pixels across Southeast Asia.
