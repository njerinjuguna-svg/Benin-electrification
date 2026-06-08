# ⚡ Benin Least-Cost Electrification Analysis

> A satellite-enriched, least-cost electrification model for **16,273 unelectrified settlements**
> across Benin, West Africa — identifying the optimal technology and financing model for
> 3.9 million people currently living without electricity.

![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![GEE](https://img.shields.io/badge/Google%20Earth%20Engine-4285F4?logo=google&logoColor=white)
![GeoPandas](https://img.shields.io/badge/GeoPandas-0.14-139C5A)
![Folium](https://img.shields.io/badge/Folium-Interactive%20Maps-77B829)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Key Findings](#key-findings)
- [Project Structure](#project-structure)
- [Data Sources](#data-sources)
- [Methodology](#methodology)
  - [Demand Estimation](#demand-estimation)
  - [Technology Options & Cost Model](#technology-options--cost-model)
  - [Least-Cost Decision Logic](#least-cost-decision-logic)
  - [Investment Prioritisation](#investment-prioritisation)
  - [Sensitivity Analysis](#sensitivity-analysis)
- [Assumptions](#assumptions)
- [Outputs](#outputs)
- [How to Run](#how-to-run)
- [Limitations & Future Work](#limitations--future-work)

---

## Overview

Benin has a rural electrification rate of approximately **13%** — one of the lowest in West Africa.
Over **3.9 million people** live without any electricity access, concentrated in remote settlements
far from the existing medium-voltage (MV) grid. The state utility SBEE is insolvent, importing
95% of national power from neighbouring countries.

This analysis implements a **least-cost electrification model** that:

1. Estimates electricity demand for each unelectrified settlement over a **15-year planning horizon (2025–2040)**
2. Calculates the Levelised Cost of Energy (**LCOE**) for three technology options per settlement
3. Assigns the **least-cost technology** with a productive-use upgrade rule grounded in VIDA's methodology
4. Incorporates **Relative Wealth Index** to map each settlement to a realistic financing model
5. Scores and ranks all 16,273 settlements for **investment prioritisation**
6. Tests **four cost scenarios** to assess recommendation robustness

The model is enriched with **seven Google Earth Engine (GEE) satellite layers** — solar radiation,
nighttime lights, land cover, terrain slope, vegetation index (NDVI), population density and rainfall —
extracted at point level for every unelectrified settlement centroid. This makes the model spatially
precise in a way that national-average approaches cannot achieve.

---

## Key Findings

| Metric | Value |
|--------|-------|
| Total Benin population modelled | 14.1 million |
| Unelectrified population (planning target) | 3.9 million **(28.1%)** |
| Unelectrified settlements | **16,273** |
| Households to electrify | **758,686** |
| Total Year 1 electricity demand | 275 GWh |
| Total Year 15 electricity demand | 416 GWh (+51%) |

### Technology Recommendation Split

| Technology | Settlements | Share | Population | Avg LCOE |
|------------|:-----------:|:-----:|:----------:|:--------:|
| Solar Home System (SHS) | 13,528 | 83.1% | 1,464,252 | $0.87/kWh |
| Solar Mini-Grid | 2,546 | 15.6% | 2,149,644 | $0.47/kWh |
| Grid Extension | 199 | 1.2% | 334,990 | $0.16/kWh |

### Key Insight — The Energy Poverty Trap

The poorest settlements (W1, RWI < −0.6) are on average **46.2 km** from the existing grid,
compared to **17.8 km** for the wealthiest unelectrified settlements — a gap of **28.4 km**.

Grid extension — the cheapest technology per kWh when close to the grid — is structurally
unavailable to the communities that need electricity most. **Off-grid solar is not a compromise;
it is the primary and often only viable pathway.**

### Sensitivity Analysis — Robustness Check

| Scenario | Grid Ext | Mini-Grid | SHS | Key change |
|----------|:--------:|:---------:|:---:|------------|
| Baseline | 199 | 2,546 | 13,528 | Reference |
| Mini-grid −20% | 199 | 3,102 | 12,972 | +556 switch SHS→MG |
| Mini-grid −40% | 199 | 3,891 | 12,183 | +1,345 switch SHS→MG |
| Grid extension +30% | 118 | 2,608 | 13,547 | −81 switch GE→MG/SHS |

**SHS dominates across all four scenarios** — the finding is robust. The mini-grid/SHS boundary
is sensitive to solar cost trends; a 40% reduction (IRENA 2030 projection) would shift ~1,345
settlements from SHS to mini-grid.

---

## Project Structure

```
benin-electrification/
│
├── notebooks/
│   └── benin_electrification_analysis.ipynb   ← Main analysis notebook (28 cells)
│
├── data/
│   ├── raw/                                    ← Input data (see Data Sources)
│   │   ├── Benin_settlement_properties.geojson
│   │   ├── Benin_existing_transmission_lines_2017.geojson
│   │   └── Benin_GEE_Master.csv
│   └── processed/
│       ├── Benin_Unelectrified_Final.csv       ← GEE export input (generated)
│       └── benin_results.csv                   ← Full model output (generated)
│
├── outputs/
│   ├── maps/                                   ← Interactive HTML maps (generated)
│   │   ├── Map1_Technology_Assignment.html
│   │   ├── Map2_Solar_Resource.html
│   │   ├── Map3_Wealth_Affordability.html
│   │   ├── Map4_Priority_Settlements.html
│   │   └── Map5_VIIRS_Nightlights.html
│   └── figures/                                ← Static PNG charts (generated)
│       ├── Chart1_Summary_Dashboard.png
│       ├── Chart2_Department_Breakdown.png
│       ├── Chart3_Energy_Poverty_Trap.png
│       └── Chart4_Satellite_Insights.png
│
├── gee/
│   └── benin_gee_fixed_extraction.js          ← GEE point-sampling script
│
├── requirements.txt
├── .gitignore
└── README.md
```

> **Note:** All files in `outputs/` and `data/processed/` are generated by running the notebook.
> Raw data files are excluded from version control due to size — see [Data Sources](#data-sources).

---

## Data Sources

| Dataset | Source | Description |
|---------|--------|-------------|
| Settlement properties | **VIDA** | Boundaries, population, buildings, MV grid distance, social services, RWI, nightlight proxy |
| Existing transmission grid | **VIDA** | Benin MV transmission lines (2017) |
| Solar radiation (GHI) | NASA ERA5 via **GEE** | Monthly solar irradiance in J/m²/month → converted to kWh/m²/day |
| Nighttime lights (VIIRS) | NOAA VIIRS via **GEE** | Night radiance (nW/cm²/sr) — independent electrification validation |
| Land cover | ESA WorldCover 2021 via **GEE** | 10 m resolution land cover — cropland, built-up, forest classification |
| Terrain slope | CGIAR SRTM 90m via **GEE** | Slope in degrees — construction CAPEX adjustment |
| Vegetation index (NDVI) | Sentinel-2 SR via **GEE** | 2023 annual NDVI — agricultural productivity proxy |
| Population density | WorldPop 2020 via **GEE** | 100 m gridded population — settlement population validation |
| Rainfall | CHIRPS v2.0 via **GEE** | Annual rainfall in mm (2023) — irrigation demand adjustment |
| Relative Wealth Index | **Meta AI Research** | Satellite + ML household wealth estimate (pre-loaded in VIDA GeoJSON) |

### Re-Extracting GEE Data

The file `data/raw/Benin_GEE_Master.csv` contains pre-extracted GEE values for all 16,233
matched unelectrified settlements. To re-extract from scratch:

1. Run **Cell 4** of the notebook to generate `Benin_Unelectrified_Final.csv`
2. Upload as an asset to [Google Earth Engine Code Editor](https://code.earthengine.google.com/)
3. Run `gee/benin_gee_fixed_extraction.js` — uses `sampleRegions()` at 500m scale
4. Export `Benin_GEE_Master.csv` to `data/raw/`

> **Why point sampling?** Settlement polygons range from tiny hamlets to large towns.
> `reduceRegions()` at 1km scale returns null for small polygons.
> Point sampling at each centroid works for all settlement sizes.

---

## Methodology

### Demand Estimation

Demand is estimated using the **Multi-Tier Framework (MTF)** — the international standard for
rural energy access measurement, developed by the World Bank ESMAP programme.

**Step 1 — MTF Tier Assignment using four signals**

Rather than assigning tiers by population alone, the model uses four signals — population,
building characteristics (from VIDA), ESA land use and RWI wealth (from GEE and Meta AI):

| Tier | kWh/HH/year | Assignment criteria |
|------|:-----------:|---------------------|
| 1 | 22 | Population < 100 AND no large buildings — remote hamlet |
| 2 | 73 | Population 100–299 AND no health/education facilities |
| 3 | 365 | Pop ≥ 300, OR health/education facility, OR large buildings, OR active cropland (NDVI > 0.3), OR wealth ≥ W4 |
| 4 | 1,250 | Pop > 3,000, OR built-up land + pop > 1,500, OR wealthy + pop > 2,000 |

**Step 2 — Three-Component Demand Formula**

```
Total Demand (kWh/yr) = Residential + Institutional + Productive

Residential  = Households × (MTF_kWh × RWI_Factor × Rainfall_Factor) × 0.70
Institutional = (Health_facilities × 500 kWh) + (Education_facilities × 300 kWh)
Productive   = Residential × Productive_Uplift[ESA_land_cover]
```

**Satellite-derived modifiers:**
- `RWI_Factor`: Wealth demand multiplier — W1=0.80, W2=0.90, W3=1.00, W4=1.10, W5=1.20
- `Rainfall_Factor`: CHIRPS data — dry north (<900mm) +15%, wet south (>1200mm) −5%
- `Productive_Uplift`: ESA WorldCover — Cropland +25%, Built-up +30%, Grassland +10%

**Step 3 — 15-Year Projection**

```
Demand(year t) = Total_Demand_Year1 × (1 + 0.03)^(t−1)
```

---

### Technology Options & Cost Model

Three technologies compared per settlement using **Levelised Cost of Energy (LCOE)**:

```
LCOE = (CAPEX + OPEX × NPV_factor) / (Annual_Demand × NPV_factor)

NPV_factor = Σ [1/(1+r)^t]  for t = 1 to lifetime
Discount rate (r): 10%  (World Bank standard for SSA)
```

**Two GEE-derived CAPEX adjustments applied to all solar technologies:**

```python
# Solar adjustment — per settlement GHI from NASA ERA5
solar_adj = max(0.85, min(1.15, 1 - 0.05 × (GHI - 5.49) / 5.49))

# Slope adjustment — terrain from CGIAR SRTM
slope_adj = 1.00  if slope ≤ 5°
          = 1.10  if slope 5–10°
          = 1.20  if slope > 10°
```

| Technology | CAPEX | OPEX | Lifetime | Key assumption |
|------------|-------|:----:|:--------:|----------------|
| Grid Extension | $15,000/km + $500/connection | 2% | 30 yrs | Distance from VIDA GeoJSON |
| Solar Mini-Grid | $15,000 fixed + $900/connection | 5% | 20 yrs | Min viable pop: 250 |
| Solar Home System | $200/HH (Tier 1–2) or $400/HH (Tier 3+) | 3% | 10 yrs | ENGIE PAYGO Benin |

> Sources: ESMAP Africa grid benchmarks, ENGIE West Africa operational data, World Bank Mini Grids 2023

---

### Least-Cost Decision Logic

Technology assignment uses a three-step logic per settlement:

```
Step 1 — Economic LCOE
  → Calculate LCOE for all three technologies
  → Remove Mini-Grid if population < 250 (fixed cost unviable)
  → Assign technology with lowest LCOE

Step 2 — VIDA Productive Use Upgrade
  → If Step 1 assigns SHS AND population ≥ 250 AND
    (has_health_facility OR has_education_facility OR
     large_buildings > 0 OR (ESA_Cropland AND NDVI > 0.3)):
  → Upgrade to Mini-Grid
  → Rationale: LCOE undervalues anchor loads — a clinic alone
    needs ~1,825 kWh/yr not captured in residential demand

Step 3 — RWI Financial Viability Flag
  → W1 (any technology)  → Subsidy required (World Bank / MCC)
  → W2 (any technology)  → Subsidised PAYGO (DFI-backed)
  → W3                   → Standard PAYGO ($0.19/day — ENGIE model)
  → W4                   → Mini-grid tariff (private operator)
  → W5                   → Commercial tariff
  → Finance source: World Bank, MCC, EIB via ABERME
    (not Benin government budget — SBEE debt = 9× annual turnover)
```

---

### Investment Prioritisation

Settlements ranked using a **multi-criteria priority score (0–100)**:

| Criterion | Max Points | Formula |
|-----------|:---------:|---------|
| Population impact | 25 | `min(population / 5,000, 1) × 25` |
| Health facility present | 12 | Binary |
| Education facility present | 8 | Binary |
| Productive use (ESA + NDVI) | 20 | Cropland=20, Built-up=18, +2 NDVI bonus if active |
| Road access | 15 | Binary if road access; decay by distance otherwise |
| Financial sustainability (RWI) | 10 | Normalised RWI × 10 |
| Solar resource quality (GEE) | 5 | `min(GHI / 6.5, 1) × 5` |
| Rainfall urgency (GEE) | 5 | Higher score for drier / water-stressed areas |

**Top 5 priority settlements:**

| Rank | Settlement | Department | Population | Technology | Score |
|:----:|-----------|-----------|:----------:|:----------:|:-----:|
| 1 | Biguina 1 | Donga | 5,669 | Mini-Grid | 89.5 |
| 2 | Dangbo | Ouémé | 5,441 | Grid Extension | 87.0 |
| 3 | Kambara | Alibori | 3,046 | Mini-Grid | 81.1 |
| 4 | Souarou | Borgou | 4,318 | Mini-Grid | 81.1 |
| 5 | Péonga | Borgou | 2,794 | Mini-Grid | 79.6 |

---

### Sensitivity Analysis

Four cost scenarios tested to assess recommendation robustness:

| Scenario | MG Cost Factor | GE Cost Factor | Rationale |
|----------|:--------------:|:--------------:|-----------|
| Baseline | ×1.0 | ×1.0 | Current market conditions |
| Mini-grid −20% | ×0.8 | ×1.0 | Conservative near-term solar cost decline |
| Mini-grid −40% | ×0.6 | ×1.0 | IRENA 2030 optimistic solar projection |
| Grid extension +30% | ×1.0 | ×1.3 | Supply chain disruption / terrain inflation |

**Finding:** SHS is recommended for ~83% of settlements in every scenario — the model is robust.
The mini-grid/SHS boundary is sensitive to solar cost trends, suggesting settlements on this
boundary should be reassessed closer to deployment as solar costs evolve.

---

## Assumptions

| # | Parameter | Value | Source |
|---|-----------|-------|--------|
| 1 | Household size | 5.2 persons | World Bank Benin household survey |
| 2 | Electrification uptake rate | 70% | ABERME operational targets |
| 3 | Planning horizon | 15 years | Standard rural electrification practice |
| 4 | Demand growth rate | 3%/year | World Bank SSA income growth projections |
| 5 | Discount rate | 10% | World Bank SSA development finance standard |
| 6 | Mini-grid min. viable population | 250 persons | ENGIE West Africa / ABERME guidance |
| 7 | Health facility demand | 500 kWh/year | Vaccine refrigerator + basic lighting |
| 8 | Education facility demand | 300 kWh/year | Classroom lighting + teacher device charging |
| 9 | NDVI productive threshold | > 0.3 | FAO agricultural remote sensing literature |
| 10 | VIIRS electrified threshold | > 1.0 nW/cm²/sr | Falchetta et al. (2019), ESMAP/World Bank |
| 11 | Benin average solar GHI | 5.49 kWh/m²/day | GEE NASA ERA5 extraction — validated |
| 12 | ENGIE PAYGO reference | $0.19/day | ENGIE Benin EIB-funded SHS programme |

---

## Outputs

### Interactive Maps — open in any browser

| File | What it shows |
|------|---------------|
| `Map1_Technology_Assignment.html` | All 16,273 settlements coloured by least-cost technology — click any settlement for LCOE, wealth tier, solar GHI, priority rank |
| `Map2_Solar_Resource.html` | Per-settlement solar GHI from NASA ERA5 — north/south gradient visible |
| `Map3_Wealth_Affordability.html` | Wealth tiers and financing models from Meta RWI — spatial poverty distribution |
| `Map4_Priority_Settlements.html` | Top 100 priority settlements on dark basemap — coloured by technology, labelled top 10 |
| `Map5_VIIRS_Nightlights.html` | Nighttime light heatmap — the 3.9 million people in darkness |

### Static Charts — for presentations

| File | What it shows |
|------|---------------|
| `Chart1_Summary_Dashboard.png` | 6-panel dashboard — technology split, population, LCOE distribution, wealth tiers, departments, energy poverty trap |
| `Chart2_Department_Breakdown.png` | Technology mix and unelectrified population by department |
| `Chart3_Energy_Poverty_Trap.png` | RWI vs distance to grid — the core policy finding |
| `Chart4_Satellite_Insights.png` | Solar gradient, ESA land use, NDVI by land cover, CHIRPS rainfall |

---

## 💻 How to Run

### 1. Clone the repository

```bash
git clone https://github.com/njerinjuguna-svg/Benin-electrification.git
cd Benin-electrification
```

### 2. Set up a virtual environment

```bash
python -m venv benin_env

# Windows
benin_env\Scripts\activate

# macOS / Linux
source benin_env/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Add data files to `data/raw/`

Place these files in `data/raw/` (excluded from repo due to size):
- `Benin_settlement_properties.geojson` — from VIDA
- `Benin_existing_transmission_lines_2017.geojson` — from VIDA
- `Benin_GEE_Master.csv` — from GEE extraction (see [Data Sources](#data-sources))

### 5. Create output directories

```bash
mkdir -p data/processed outputs/maps outputs/figures
```

### 6. Launch the notebook

```bash
jupyter notebook notebooks/benin_electrification_analysis.ipynb
```

Run all cells in order: **Kernel → Restart & Run All**

Full analysis completes in approximately **3–5 minutes** on a standard laptop.

---

## Limitations & Future Work

| Limitation | Impact | Future improvement |
|-----------|--------|-------------------|
| Grid data from 2017 | MV lines built 2017–2025 not captured; some GE costs may be overstated | Update with ABERME current grid data |
| Straight-line grid distance | Actual routing follows roads and easements | Implement Dijkstra on OSM road network |
| Fixed household size (5.2) | Does not capture urban/rural variation | Use settlement-level census micro-data |
| Fixed population base | No population growth modelling within settlements | Add UN WPP growth scenarios |
| Mini-grid costs not capacity-scaled | $15,000 fixed regardless of system size | Apply capacity-cost scaling curves |
| Demand growth rate not varied | Only cost scenarios tested | Add demand sensitivity: 1%, 3%, 5%/yr |
| Discount rate fixed at 10% | Does not capture concessional vs commercial financing | Test 8% (IDA) vs 12% (commercial) |
| No grid densification option | Peri-urban LV extension not modelled | Add fourth technology option |
| No Monte Carlo uncertainty | Point estimates only | Full probabilistic sensitivity analysis |

### Suggested Next Steps

1. **Validate with ABERME data** — cross-check assignments against existing rural programmes
2. **Add grid routing** — use OpenStreetMap road network for realistic MV extension costs
3. **Cluster analysis** — identify groups of settlements that could share mini-grid infrastructure
4. **Load profile integration** — replace annual kWh with hourly demand curves for mini-grid sizing
5. **Demand growth scenarios** — add UN population projections at settlement level

---

## Author

**Fides N. Njuguna**  
Geospatial Analyst & GIS Developer — Nairobi, Kenya

[![Email](https://img.shields.io/badge/Email-njerinjuguna943%40gmail.com-D14836?logo=gmail&logoColor=white)](mailto:njerinjuguna943@gmail.com)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-fides--njuguna-0077B5?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/fides-njuguna-b4272a239/)
[![GitHub](https://img.shields.io/badge/GitHub-njerinjuguna--svg-181717?logo=github&logoColor=white)](https://github.com/njerinjuguna-svg)

---

*Analysis conducted May 2026 using Python 3.11, GeoPandas, Folium, and Google Earth Engine.*  
*Data: VIDA (settlement and grid), GEE (NASA ERA5, NOAA VIIRS, ESA WorldCover, Sentinel-2, WorldPop, CHIRPS), Meta AI (Relative Wealth Index).*
