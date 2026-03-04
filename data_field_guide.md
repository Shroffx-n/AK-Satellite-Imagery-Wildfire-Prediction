# Alaska Wildfire Prediction — Data Field Guide

This guide is the written companion to [`notebooks/00_data_overview.ipynb`](../notebooks/00_data_overview.ipynb). Read this before opening the notebook. It covers every data source used in the pipeline: what it is, how to access it step by step, what the downloaded files look like, and what preprocessing is required before the data can enter the model.

---

## Table of Contents

1. [Data Availability Matrix](#1-data-availability-matrix)
2. [Sentinel-2 (ESA)](#2-sentinel-2-esa)
3. [Sentinel-1 (ESA)](#3-sentinel-1-esa)
4. [Landsat 8 & 9 (NASA/USGS)](#4-landsat-8--9-nasausgs)
5. [MODIS (NASA Terra/Aqua)](#5-modis-nasa-terraaqua)
6. [VIIRS (Suomi NPP & NOAA-20)](#6-viirs-suomi-npp--noaa-20)
7. [ALOS-2 (JAXA)](#7-alos-2-jaxa)
8. [ERA5 (ECMWF)](#8-era5-ecmwf)
9. [NOAA NWS Weather Stations](#9-noaa-nws-weather-stations)
10. [Alaska Fire Service (AFS)](#10-alaska-fire-service-afs)
11. [Alaska-Specific Geospatial Notes](#11-alaska-specific-geospatial-notes)
12. [Recommended Bounding Boxes](#12-recommended-bounding-boxes)
13. [Data Volume Guide](#13-data-volume-guide)

---

## 1. Data Availability Matrix

| Source | Resolution | Revisit | Alaska Coverage | Time Range | Latency | File Format | Size / Scene | Access |
|--------|-----------|---------|----------------|------------|---------|-------------|-------------|--------|
| **Sentinel-2** | 10m / 20m | 5 days | Full | 2017–present | ~2 days | `.SAFE` folder (JP2) | 700 MB – 1.2 GB | Copernicus CDSE |
| **Sentinel-1** | 5–20m | 6–12 days | Full | 2014–present | ~1 day | `.SAFE` folder (TIFF) | 800 MB – 1.5 GB | Copernicus CDSE |
| **Landsat 8/9** | 30m / 100m thermal | 16 days | Full | 2013–present | ~1 day | `.tar.gz` (GeoTIFF) | 1–2 GB | USGS EarthExplorer |
| **MODIS** | 250m – 1km | Daily | Full | 2000–present | Same day | HDF4 (`.hdf`) | 10–50 MB | NASA EarthData |
| **VIIRS** | 375m – 750m | Daily | Full | 2012–present | Same day | CSV / HDF5 | 1–10 MB | NASA FIRMS |
| **ALOS-2** | 10–100m | 14 days | Full | 2014–present | ~2 weeks | GeoTIFF | 500 MB – 2 GB | JAXA EORC |
| **ERA5** | ~25 km | Hourly | Full | 1940–present | 5-day lag | NetCDF (`.nc`) | 50–200 MB / yr | ECMWF CDS |
| **NOAA NWS** | Point stations | Sub-hourly | Partial (~120 AK stations) | 1970s–present | Real-time | JSON (REST API) | Small | api.weather.gov |
| **AFS Fire Data** | Vector polygons | Seasonal | Alaska only | 1940s–present | End of season | Shapefile / GeoJSON | ~100 MB total | AFS GIS Portal |

---

## 2. Sentinel-2 (ESA)

**Role in this project:** Primary optical data source. Used for NDVI, NBR, burn severity mapping, and fire risk classification. The mentor's stated priority.

**Portal:** https://dataspace.copernicus.eu

### Step-by-step access

**Step 1 — Register**
Create a free account at https://dataspace.copernicus.eu. You will need credentials for both the web portal and the Python API.

**Step 2 — Install the Python client**


**Step 3 — Configure credentials**

**Step 4 — Search for scenes**
Always use **Level-2A (L2A)** — atmospherically corrected surface reflectance. Filter by cloud cover < 20% for Alaska.


**Step 5 — Download**
Each tile is ~700 MB – 1.2 GB. Download to `data/raw/sentinel2/`.

**Step 6 — Understand the file structure**
```
S2A_MSIL2A_20230715T211231_N0509_R142_T06VUN_20230716T002318.SAFE/
├── GRANULE/
│   └── L2A_.../
│       └── IMG_DATA/
│           ├── R10m/     B02.jp2  B03.jp2  B04.jp2  B08.jp2  TCI.jp2
│           ├── R20m/     B05.jp2  B06.jp2  B07.jp2  B8A.jp2  B11.jp2  B12.jp2  SCL.jp2
│           └── R60m/     B01.jp2  B09.jp2
├── MTD_MSIL2A.xml        (product metadata)
└── INSPIRE.xml
```

**Step 7 — Open and convert to reflectance**


**Step 8 — Apply SCL cloud mask**


**Step 9 — Reproject to EPSG:3338**


**Step 10 — Compute spectral indices**


### Key bands

| Band | Name | λ (nm) | Resolution | Use |
|------|------|---------|-----------|-----|
| B02 | Blue | 490 | 10m | True-colour |
| B03 | Green | 560 | 10m | True-colour |
| B04 | Red | 665 | 10m | NDVI |
| B08 | NIR | 842 | 10m | NDVI, vegetation |
| B11 | SWIR-1 | 1610 | 20m | Burn severity, moisture |
| B12 | SWIR-2 | 2190 | 20m | Active fire |
| SCL | Scene Class | — | 20m | Cloud mask |

---

## 3. Sentinel-1 (ESA)

**Role in this project:** Primary SAR source. Cloud-penetrating. Used for vegetation moisture, fuel dryness, and burned area mapping.

**Portal:** https://dataspace.copernicus.eu (same account as Sentinel-2)

### Step-by-step access

**Step 1 — Search for scenes**
Use the same STAC API. Filter for `productType = GRD` and `sensorMode = IW`.

**Step 2 — Download**
Same process as Sentinel-2. Target `data/raw/sentinel1/`. Size: 800 MB – 1.5 GB.

**Step 3 — Understand file structure**
```
S1A_IW_GRDH_1SDV_20230715T160023_20230715T160048_049417_05F1E3.SAFE/
├── measurement/
│   ├── s1a-iw-grd-vh-20230715t160023-....tiff    (VH polarisation)
│   └── s1a-iw-grd-vv-20230715t160023-....tiff    (VV polarisation)
├── annotation/
│   ├── s1a-iw-grd-vh-....xml    (calibration metadata)
│   └── s1a-iw-grd-vv-....xml
└── manifest.safe
```

**Step 4 — Preprocess with ESA SNAP (required)**
Raw Sentinel-1 GRD cannot be used directly. In SNAP (or via `pyroSAR`):

1. Apply Orbit File
2. Remove Thermal Noise
3. Calibrate → σ⁰ (sigma-nought, linear scale)
4. Terrain Correct using Copernicus 30m DEM
5. Export as GeoTIFF


**Step 5 — Open and convert to dB**


**Step 6 — Reproject to EPSG:3338** (same as Sentinel-2 Step 9)

### Interpreting backscatter values (VV, dB)

| Land cover | Typical σ⁰ VV (dB) |
|------------|-------------------|
| Dense boreal forest | −8 to −4 |
| Tundra / open shrub | −14 to −8 |
| Burned area | −18 to −12 |
| Calm open water | −25 to −20 |

---

## 4. Landsat 8 & 9 (NASA/USGS)

**Role in this project:** Pre/post-fire vegetation tracking and burn severity mapping. Thermal band detects fire hotspots. Long archive (1984–present across all Landsat missions) enables historical baselines.

**Portal:** https://earthexplorer.usgs.gov

### Step-by-step access

**Step 1 — Register**
Free account at https://ers.cr.usgs.gov/register

**Step 2 — Install the Python client**

**Step 4 — Extract the tar archive**

**Step 5 — Understand file structure**
```
LC09_L2SP_068017_20230720_20230722_02_T1/
├── LC09_..._SR_B2.TIF    Blue    30m
├── LC09_..._SR_B3.TIF    Green   30m
├── LC09_..._SR_B4.TIF    Red     30m
├── LC09_..._SR_B5.TIF    NIR     30m
├── LC09_..._SR_B6.TIF    SWIR-1  30m
├── LC09_..._SR_B7.TIF    SWIR-2  30m
├── LC09_..._ST_B10.TIF   Thermal 100m (resampled to 30m)
├── LC09_..._QA_PIXEL.TIF Cloud/shadow/water mask
└── LC09_..._MTL.txt      Metadata + scale factors
```

**Step 6 — Convert DN to physical values**


**Step 7 — Apply QA_PIXEL cloud mask**

### Burn severity with dNBR

| dNBR range | Burn severity class |
|------------|-------------------|
| < −0.25 | Enhanced regrowth |
| −0.25 – 0.10 | Unburned |
| 0.10 – 0.27 | Low severity |
| 0.27 – 0.44 | Moderate-low |
| 0.44 – 0.66 | Moderate-high |
| > 0.66 | High severity |

---

## 5. MODIS (NASA Terra/Aqua)

**Role in this project:** Historical fire perimeters (ground truth labels) and daily thermal anomaly monitoring. Daily revisit makes it the best source for tracking fire evolution.

**Portal:** https://earthdata.nasa.gov  
**Direct FIRMS access (recommended):** https://firms.modaps.eosdis.nasa.gov

### Step-by-step access

**Step 1 — Register**
Free account at https://urs.earthdata.nasa.gov

**Step 2 — Install earthaccess**


**Step 3 — Search and download MCD64A1 (monthly burned area)**


**Step 4 — Open HDF4 file**


**File naming convention:**
`MCD64A1.A2023213.h11v02.061.hdf`
- `MCD64A1` — product name
- `A2023213` — acquisition year 2023, day 213 (August 1)
- `h11v02` — MODIS tile grid (h=horizontal, v=vertical)
- `061` — collection version

---

## 6. VIIRS (Suomi NPP & NOAA-20)

**Role in this project:** Real-time and near-real-time active fire detection. Higher resolution than MODIS (375m vs 1km) and lower false alarm rate. The recommended source for fire hotspot labels.

**Portal:** https://firms.modaps.eosdis.nasa.gov  
**API key:** Register free at https://firms.modaps.eosdis.nasa.gov/api/

### Step-by-step access

**Step 1 — Get a free MAP_KEY**
Register at https://firms.modaps.eosdis.nasa.gov/api/ — you'll receive a MAP_KEY by email within minutes.

**Step 2 — Fetch active fire CSV via API**


**Step 3 — Understand CSV structure**

| Column | Description | Units |
|--------|-------------|-------|
| `latitude` | Fire detection location | degrees |
| `longitude` | Fire detection location | degrees |
| `bright_ti4` | Brightness temperature Band I4 | Kelvin |
| `bright_ti5` | Brightness temperature Band I5 | Kelvin |
| `frp` | Fire Radiative Power | MW |
| `acq_date` | Detection date | YYYY-MM-DD |
| `acq_time` | Detection time (UTC) | HHMM |
| `satellite` | N=Suomi NPP, 1=NOAA-20 | |
| `confidence` | Detection confidence | high/nominal/low |

**Step 4 — Filter to high-quality detections**


**Step 5 — Download historical archive (for training data)**
For dates older than 7 days, download from the FIRMS archive:
https://firms.modaps.eosdis.nasa.gov/download/ — select product, date range, and Alaska bounding box.

---

## 7. ALOS-2 (JAXA)

**Role in this project:** L-band SAR for detecting dry fuel beneath the forest canopy and terrain changes post-fire. Complements Sentinel-1 (C-band) by penetrating deeper into the canopy.

**Portal:** https://www.eorc.jaxa.jp/ALOS/en/index_e.htm  
**Annual mosaic (easier):** https://www.eorc.jaxa.jp/ALOS/en/dataset/fnf_e.htm

### Step-by-step access

**Step 1 — Register**
Free account at https://www.eorc.jaxa.jp/ALOS/en/index_e.htm. Registration may require institutional affiliation for full scene access.

**Step 2 — Download options**

Option A — Full scenes (PALSAR-2):
Search and order via the JAXA G-Portal: https://gportal.jaxa.jp/gpr/

Option B — Annual 25m global mosaics (easier, recommended for initial work):
Direct download at the FNF portal. Covers Alaska. Pre-processed HH/HV GeoTIFF mosaics.

**Step 3 — Preprocessing pipeline**
Similar to Sentinel-1 but uses HH/HV polarisations instead of VV/VH:
1. Apply orbit file
2. Radiometric calibration → σ⁰
3. Terrain correction (use SRTM or Copernicus DEM)
4. Convert to dB: `10 * log10(sigma_linear)`

**Key difference from Sentinel-1:**

| Property | Sentinel-1 (C-band) | ALOS-2 (L-band) |
|----------|---------------------|-----------------|
| Frequency | 5.4 GHz | 1.27 GHz |
| Wavelength | 5.6 cm | 23.5 cm |
| Canopy penetration | Surface + upper canopy | Through canopy to ground |
| Soil moisture | Moderate sensitivity | High sensitivity |
| Dry fuel detection | Limited | Strong |

---

## 8. ERA5 (ECMWF)

**Role in this project:** Historical weather time-series (30+ years) for temperature, wind, humidity, precipitation. Used as time-series input features for the CNN-LSTM. Also the basis for computing the Fire Weather Index (FWI).

**Portal:** https://cds.climate.copernicus.eu

### Step-by-step access

**Step 1 — Register**
Free account at https://cds.climate.copernicus.eu

**Step 2 — Install cdsapi and configure credentials**

Create `~/.cdsapirc`:

Your UID and API key are found at https://cds.climate.copernicus.eu/user (after login).

**Step 3 — Download data**


**Step 4 — Open with xarray**


**Step 5 — Clip to study area and resample**


### Key variables for FWI computation

| ERA5 variable | Short name | Units | FWI input |
|---------------|-----------|-------|-----------|
| 2m temperature | `t2m` | K → convert to °C | Temperature |
| 2m dewpoint | `d2m` | K → compute RH | Relative humidity |
| 10m U wind | `u10` | m/s → combine to speed | Wind speed |
| 10m V wind | `v10` | m/s → combine to speed | Wind speed |
| Total precipitation | `tp` | m → × 1000 for mm | 24h rain |

---

## 9. NOAA NWS Weather Stations

**Role in this project:** Near real-time weather (< 5-minute latency). Station observations validate ERA5 reanalysis values and can supplement the model with real-time inputs at inference time.

**Portal:** https://www.weather.gov  
**API:** https://api.weather.gov (no key required)

### Step-by-step access

**Step 1 — No registration needed**
The NOAA NWS API is completely open. Always include a descriptive `User-Agent` header.

**Step 2 — Find Alaska stations**


**Step 3 — Fetch observations for a specific station**

**Step 4 — Derive fire-relevant variables**


### Key Interior Alaska stations

| ID | Location | Lat | Lon | Elevation |
|----|----------|-----|-----|-----------|
| PAFA | Fairbanks International Airport | 64.82° N | 147.86° W | 138m |
| PAEI | Eielson AFB | 64.66° N | 147.10° W | 160m |
| PFTO | Tok Junction | 63.33° N | 142.99° W | 582m |
| PAIN | McKinley Park | 63.73° N | 148.91° W | 658m |
| PABT | Bettles Field | 66.91° N | 151.53° W | 191m |
| PAWT | Wasilla | 61.57° N | 149.54° W | 90m |

---

## 10. Alaska Fire Service (AFS)

**Role in this project:** Official fire perimeter polygons are the **ground truth labels** for the ML model. The `CAUSE` field (Lightning / Human / Unknown) is specifically called out by the project mentors as a target variable.

**Portal:** https://fire.ak.blm.gov/content/maps/aicc/Data/

### Step-by-step access

**Step 1 — Download fire perimeter data (no login required)**
Navigate to https://fire.ak.blm.gov/content/maps/aicc/Data/ and download:
- `AlaskaFireHistory_Perimeters_1940_to_Present.zip` — full historical archive


**Step 2 — Open with geopandas**


**Step 3 — Filter to study area and time period**

**Step 4 — Understand key attributes**

| Field | Type | Description |
|-------|------|-------------|
| `FIRENAME` | String | Official fire name |
| `FIREYEAR` | Integer | Year fire occurred |
| `STARTDTM` | DateTime | Ignition date/time |
| `ENDDTM` | DateTime | Containment date/time |
| `ACRES` | Float | Official burned area (acres) |
| `CAUSE` | String | Lightning / Human / Unknown |
| `geometry` | Polygon | Fire perimeter |

**Step 5 — Use perimeters as model labels**


---

## 11. Alaska-Specific Geospatial Notes

### Always use EPSG:3338 for analysis
All spatial operations — area calculations, overlays, distance measurements, rasterization — must use **EPSG:3338 (Alaska Albers Equal Area Conic)**. This is the standard used by AFS and the UAF Geophysical Institute. Using EPSG:4326 (geographic coordinates) will give incorrect area values and distorted maps.

### Antimeridian issue
Western Alaska (Aleutian Islands) crosses 180° longitude. If you are working with full-state data, avoid WGS84 longitude-based representations. For this project, clipping to Interior Alaska (east of −141°) sidesteps the issue entirely.

### UTM zone variation
Sentinel-2 tiles over Alaska span multiple UTM zones:
- Interior Alaska (Fairbanks area): **UTM Zone 6N (EPSG:32606)**
- Southeast Alaska: **UTM Zone 8N (EPSG:32608)**
- Southwest Alaska: **UTM Zone 5N (EPSG:32605)**

Always check `src.crs` after opening any raster before assuming UTM zone.

### Cloud cover
Alaska summers average 60–80% cloud cover. A single Sentinel-2 scene will often be partially or fully clouded. Always:
1. Filter search results to < 20% reported cloud cover as a first pass
2. Apply SCL cloud masking per-pixel
3. Build monthly median composites across multiple scenes (see notebook Section 9)

### Temporal alignment
When joining satellite observations with ERA5 weather:
- Sentinel-2 acquisition times are UTC — typically around 21:00–22:00 UTC for Alaska (early morning local time)
- ERA5 is hourly UTC — use the 12:00 UTC snapshot for FWI computation (noon local ≈ 02:00–04:00 UTC which doesn't match, but noon-UTC ERA5 is the FWI convention for Alaska)
- AFS fire dates are Alaska Daylight Time (UTC−8) — convert before joining

---

## 12. Recommended Bounding Boxes

Copy-paste ready Python dictionaries for use with STAC searches, ERA5 downloads, and spatial clipping.

```python
# All bounding boxes in EPSG:4326 (WGS84 lon/lat)

# Full Alaska state
ALASKA = {
    'min_lon': -179.9,
    'max_lon': -130.0,
    'min_lat':  51.0,
    'max_lat':  71.5,
}

# Interior Alaska — primary study area (boreal forest, most fires)
INTERIOR_ALASKA = {
    'min_lon': -152.0,
    'max_lon': -141.0,
    'min_lat':  62.0,
    'max_lat':  68.0,
}

# Fairbanks test region — small area for rapid prototyping
FAIRBANKS_TEST = {
    'min_lon': -149.5,
    'max_lon': -146.5,
    'min_lat':  64.0,
    'max_lat':  66.0,
}

# Southcentral Alaska (Anchorage / Kenai)
SOUTHCENTRAL_ALASKA = {
    'min_lon': -153.0,
    'max_lon': -147.0,
    'min_lat':  59.0,
    'max_lat':  63.0,
}

# As tuples (West, South, East, North) — for FIRMS API / rasterio
INTERIOR_ALASKA_TUPLE = (-152.0, 62.0, -141.0, 68.0)

# ERA5 format (North, West, South, East)
INTERIOR_ALASKA_ERA5 = [68, -152, 62, -141]
```

---

## 13. Data Volume Guide

Estimated storage requirements for a minimal working prototype (Interior Alaska, 2019–2023, fire season only May–September):

| Source | Scenes / files | Raw size | Processed size |
|--------|---------------|----------|---------------|
| Sentinel-2 (1 tile, 5 years, monthly best scene) | ~30 scenes | ~25 GB | ~5 GB (GeoTIFF, compressed) |
| Sentinel-1 (1 path, 5 years, monthly) | ~30 scenes | ~35 GB | ~3 GB |
| Landsat 8/9 (1 path/row, 5 years) | ~15 scenes | ~25 GB | ~2 GB |
| MODIS burned area (5 tiles × 5 years × 5 months) | ~125 HDF | ~3 GB | ~0.5 GB |
| VIIRS fire CSV (5 years, Alaska) | ~5 files | ~50 MB | ~50 MB |
| ERA5 (5 years, fire season, 5 variables) | 1 NetCDF | ~200 MB | ~200 MB |
| AFS perimeters | 1 Shapefile | ~50 MB | ~50 MB |
| **Total (approximate)** | | **~90 GB raw** | **~11 GB processed** |

**Recommendation for getting started:** Use the **Fairbanks test region** bounding box and a **single year (2019)** to develop and test your pipeline. This reduces raw data to roughly 5–10 GB and makes iteration fast. Scale to the full Interior Alaska extent once the pipeline is validated.

---

## Access Portal Quick Reference

| Source | URL |
|--------|-----|
| Copernicus Data Space (Sentinel-1/2) | https://dataspace.copernicus.eu |
| USGS EarthExplorer (Landsat) | https://earthexplorer.usgs.gov |
| NASA EarthData (MODIS) | https://earthdata.nasa.gov |
| NASA FIRMS (VIIRS/MODIS active fire) | https://firms.modaps.eosdis.nasa.gov |
| JAXA EORC (ALOS-2) | https://www.eorc.jaxa.jp/ALOS |
| ECMWF CDS (ERA5) | https://cds.climate.copernicus.eu |
| NOAA NWS API | https://api.weather.gov |
| Alaska Fire Service GIS | https://fire.ak.blm.gov/content/maps/aicc/Data/ |

---

*This document is part of the Alaska Wildfire Prediction project. See [`notebooks/00_data_overview.ipynb`](../notebooks/00_data_overview.ipynb) for runnable code examples for each source.*
