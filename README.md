# Delhi Metro Operations Optimization — Data Analysis

> End-to-end data pipeline, EDA, and ML modeling on official DMRC GTFS feed to detect congestion patterns and enable demand-driven scheduling.

---
## Table of Contents
- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Project Structure](#project-structure)
- [Methodology](#methodology)
  - [EDA & Visualization](#eda--visualization)
  - [Model 1 — Hourly Demand Forecasting](#model-1--hourly-demand-forecasting)
  - [Model 2 — Station Load Clustering](#model-2--station-load-clustering)
- [Results](#results)
- [Key Insights](#key-insights)
- [Installation & Usage](#installation--usage)
- [Tech Stack](#tech-stack)

---

## Project Overview

The Delhi Metro serves millions of daily commuters across a **390 km network** with **11 lines and 262 stations**. Despite its scale, it faces persistent operational inefficiencies — peak-hour overcrowding, infrequent off-peak service, and interchange congestion.

This project uses the **official DMRC GTFS (General Transit Feed Specification) dataset** to:
1. Perform exploratory data analysis and geospatial visualization of the entire network
2. Build a **Random Forest demand forecasting model** to predict hourly stop-visit volume per route
3. Build a **KMeans station clustering model** to automatically segment all 262 stations into operational load tiers
4. Translate findings into data-driven, actionable scheduling recommendations

---

## Dataset

**Source:** Delhi Metro Rail Corporation (DMRC) — Official GTFS Feed

| File | Description | Rows | Key Columns |
|------|-------------|------|-------------|
| `stop_times.txt` | Arrival/departure times at each stop per trip | 128,434 | `trip_id`, `arrival_time`, `stop_id`, `stop_sequence` |
| `trips.txt` | Individual trips linked to routes and service calendars | 5,438 | `trip_id`, `route_id`, `service_id`, `shape_id` |
| `stops.txt` | Station metadata with geographic coordinates | 262 | `stop_id`, `stop_name`, `stop_lat`, `stop_lon` |
| `routes.txt` | Metro line definitions | 36 | `route_id`, `route_long_name`, `route_color` |
| `calendar.txt` | Service patterns (weekday / Saturday / Sunday) | 3 | `service_id`, `monday`–`sunday` |
| `shapes.txt` | Geospatial polylines for each route | 6,643 | `shape_id`, `shape_pt_lat`, `shape_pt_lon` |
| `agency.txt` | Operator metadata | 1 | `agency_name`, `agency_timezone` |

**Lines covered:** RED · YELLOW · BLUE · VIOLET · GREEN · PINK · MAGENTA · AQUA · ORANGE/AIRPORT · GRAY · RAPID  
**Service types:** Weekday, Saturday, Sunday  
**Operating hours:** 04:00 – 23:00

---

## Project Structure

```
delhi-metro-optimization/
│
├── data/                        # GTFS feed files
│   ├── stop_times.txt
│   ├── trips.txt
│   ├── stops.txt
│   ├── routes.txt
│   ├── calendar.txt
│   ├── shapes.txt
│   └── agency.txt
│
├── notebooks/
│   ├── 01_EDA_Visualization.ipynb       # Exploratory analysis & folium maps
│   ├── 02_Demand_Forecasting.ipynb      # Random Forest demand model
│   ├── 03_Station_Clustering.ipynb      # KMeans station segmentation
│   └── DelhiMetro_ML_Models.ipynb       # Combined ML notebook (final)
│
├── outputs/
│   └── station_clusters_final.csv       # Station tier assignments (262 stations)
│
├── requirements.txt
└── README.md
```

---

## Methodology

### EDA & Visualization

All GTFS tables were merged into a unified DataFrame (`stop_times ⋈ trips ⋈ stops ⋈ routes`) for analysis.

**Key steps:**
- Handled missing values, fixed coordinate mismatches, removed duplicates
- Extracted `hour` from `arrival_time` with cyclical encoding
- Built an **interactive Folium map** plotting all 11 metro lines with color-coded routes and station popup markers
- Analyzed hourly stop-visit distribution across 6 time blocks
- Identified top 15 busiest stations and top 10 routes by trip count
- Computed weekday vs. weekend trip index (100 vs. 72 normalized avg)

---

### Model 1 — Hourly Demand Forecasting

**Goal:** Predict stop-visit volume for a given route, hour, and service type — to enable proactive, demand-driven frequency adjustments instead of static timetables.

**Feature Engineering**

Aggregated raw `stop_times` into **622 route-hour-service samples** (36 routes × up to ~17 operating hours × 2 service types), then engineered 6 features:

| Feature | Description |
|---------|-------------|
| `hour` | Hour of day (4–23) |
| `hour_sin`, `hour_cos` | Cyclical encoding of hour to capture periodicity |
| `is_weekday` | Binary flag: weekday (1) vs. Saturday (0) |
| `is_peak` | Binary flag: hours 8, 9, 17, 18 = peak (1) |
| `route_avg_demand` | Historical mean stop-visits for this route (route-level prior) |
| `num_stops_on_route` | Number of unique stops served by this route |

**Models Trained**

| Model | Details |
|-------|---------|
| Naive Baseline | Predict training mean for all samples |
| Linear Regression | OLS, same 6 features |
| Random Forest | `n_estimators=200`, `max_depth=8`, `random_state=42` |

**Validation Protocols**

Two evaluation protocols were used separately to be rigorous about what the model actually learns:

- **Experiment A — Random 80/20 split:** Model has seen each route's historical demand level. Represents realistic deployment (operator has historical data on all existing routes).
- **Experiment B — 5-fold GroupKFold CV (grouped by `route_id`):** Entire routes held out per fold. Tests true generalization to routes unseen during training — a much stricter setting.

---

### Model 2 — Station Load Clustering

**Goal:** Automatically segment all 262 stations into operational load tiers to systematically flag which stations need infrastructure upgrades, dwell-time increases, or additional staffing — replacing ad-hoc "top-N busiest" lists.

**Feature Engineering (per station)**

| Feature | Description |
|---------|-------------|
| `total_visits` | Total stop-visits across all trips in the dataset |
| `num_routes` | Number of distinct routes serving this station (interchange proxy) |
| `peak_ratio` | Fraction of visits occurring during peak hours (8–9h, 17–18h) |

All features were **StandardScaler-normalized** before clustering.

**Model Selection**

Silhouette score was computed for k = 2 through 7 to select the optimal number of clusters. **k=4** was chosen for its combination of a strong silhouette score and clean, operationally-interpretable tier labels:

| Tier | Label | Stations |
|------|-------|----------|
| 1 | Major Interchange | 16 |
| 2 | High Load | 8 |
| 3 | Moderate | 104 |
| 4 | Low Load | 134 |

`KMeans(n_clusters=4, n_init=10, random_state=42)`

---

## Results

### Demand Forecasting

| Model | MAE | RMSE | R² |
|-------|-----|------|----|
| Naive Baseline | 124.22 | — | — |
| Linear Regression | — | — | — |
| **Random Forest (Exp A — known routes)** | **17.6** | **27.2** | **0.97** |
| Random Forest (Exp B — unseen routes, 5-fold Group-CV) | 34.9 ± 14.5 | 50.8 ± 16.7 | 0.79 |

- RF reduces MAE by **85.8%** vs. naive baseline on known routes
- Holds **R²=0.79** under strict unseen-route cross-validation

**Feature Importances (Random Forest, Experiment A):**

| Feature | Importance |
|---------|-----------|
| `route_avg_demand` | 0.731 |
| `num_stops_on_route` | 0.137 |
| `hour` | 0.103 |
| `hour_sin` / `hour_cos` | 0.027 |
| `is_peak` | 0.002 |
| `is_weekday` | ~0.000 |

---

### Station Clustering

| Metric | Value |
|--------|-------|
| Algorithm | KMeans (k=4) |
| Silhouette Score | **0.537** |
| Tier-1 stations identified | **16 / 262** |

**Tier-1 Major Interchange stations (top by visits):**
Kashmere Gate (1,640 visits, 12 routes) · Rajiv Chowk · Central Secretariat · Mandi House · Qutab Minar · Dilli Haat-INA · Sikanderpur · Hauz Khas

---

## Key Insights

- **Evening (17–20h) is the peak demand block** at ~31K system-wide stop-visits — **1.8× the Early-AM volume** (~17.5K). Morning (9–12h) is a close second.
- **Weekday trips run 39% higher than weekends** (normalized index: 100 vs. 72), supporting calendar-based frequency reductions on weekends without significant commuter impact.
- **Kashmere Gate serves 12 distinct routes** — the highest interchange density in the network — making it the single most critical node for congestion management.
- **route_avg_demand dominates RF feature importance (73.1%)** — which route it is matters more than the time of day for predicting load. This suggests that route-level capacity planning is more impactful than hourly scheduling tweaks alone.
- **16 stations account for disproportionate interchange load** — a targeted intervention on these 16 Tier-1 stations (dwell-time increases, platform infrastructure upgrades, staffing) covers the highest-risk congestion points.

**Recommendations derived from analysis:**
- Increase peak-hour frequency to 20–24 trains/hour on Yellow and Blue lines
- Introduce short-loop trains between highest-demand segments (e.g. Dwarka ↔ Rajiv Chowk)
- Run 3–4 coach trains at higher frequency during off-peak hours to reduce wasted capacity
- Upgrade station infrastructure at all 16 Tier-1 stations with pick-up/drop-off bays and auto-rickshaw zones

---

## Installation & Usage

```bash
# Clone the repo
git clone https://github.com/SaurabhRaj-110/Metro-Operation-Optimization.git
cd Metro-Operation-Optimization

# Install dependencies
pip install -r requirements.txt

# Run the combined ML notebook
jupyter notebook notebooks/DelhiMetro_ML_Models.ipynb
```

**requirements.txt**
```
pandas
numpy
scikit-learn
matplotlib
seaborn
folium
jupyter
nbformat
```

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Data Processing | `pandas`, `numpy` |
| Machine Learning | `scikit-learn` (RandomForestRegressor, KMeans, GroupKFold, StandardScaler) |
| Visualization | `matplotlib`, `seaborn`, `folium` |
| Data Format | GTFS (General Transit Feed Specification) |
| Environment | Jupyter Notebook, Python 3.x |

---


> *Data Source: Delhi Metro Rail Corporation (DMRC) Official GTFS Feed*  
