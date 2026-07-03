## Overview

This project performs exploratory analysis on Delhi Metro GTFS datasets by integrating multiple transit files, engineering time-based features, and visualizing operational trends across metro routes and stations.

The analysis answers questions such as:

- Which hours experience the highest metro demand?
- Which routes are the busiest?
- Which stations receive the highest traffic?
- How does demand vary between peak and off-peak hours?
- Which routes may require higher or lower service frequency?

---

## Dataset

The project uses **GTFS (General Transit Feed Specification)** files:

- `agency.txt`
- `calendar.txt`
- `routes.txt`
- `shapes.txt`
- `stop_times.txt`
- `stops.txt`
- `trips.txt`

These files are merged to build a unified dataset for analysis.

---

# Project Workflow

```
GTFS Data
      │
      ▼
Load Multiple Transit Files
      │
      ▼
Data Cleaning
      │
      ▼
Merge GTFS Tables
      │
      ▼
Time Feature Engineering
      │
      ▼
Exploratory Data Analysis
      │
      ▼
Operational Insights &
Visualization
```

---

# Features Implemented

## Data Integration

- Loaded multiple GTFS files
- Merged routes, trips, stops and stop-times
- Converted arrival timestamps into hourly features
- Filtered invalid timestamps

---

## Hourly Demand Analysis

- Hour-wise metro traffic distribution
- Total stop visits throughout the day
- Identification of system-wide peak hours

Visualizations include:

- Heatmaps
- Folium
- Bar charts

---

## Route Analysis

The notebook identifies:

- Top busiest metro routes
- Route-wise hourly demand
- Routes with maximum trip volume
- Routes covering the highest number of stations

---

## Station Analysis

The project analyzes:

- Top busiest metro stations
- Station visit frequencies
- Route coverage across stations

---

## Peak vs Off-Peak Analysis

Traffic is categorized into:

- Peak Hours
- Off-Peak Hours
- Other Hours

The project compares route utilization across these periods to identify congestion patterns.

---

## Service Analysis

Using GTFS calendar data, the notebook evaluates:

- Weekday services
- Weekend services
- Distribution of active metro services

---

## Time Block Analysis

Demand is grouped into:

- Early Morning
- Morning
- Afternoon
- Evening
- Late Evening
- Night

This provides a higher-level understanding of daily metro operations.

---

# Technologies Used

- Python
- Pandas
- Matplotlib
- Seaborn
- Jupyter Notebook

---

# Repository Structure

```
DelhiMetro-Analytics/
│
├── Metro_Final.ipynb
├── README.md
└── GTFS Dataset Files
```

---

# Key Insights

- Identified peak operating hours across the metro network.
- Ranked metro routes based on traffic and trip volume.
- Detected overcrowded and underutilized routes using demand thresholds.
- Compared route utilization during peak and off-peak periods.
- Identified the busiest metro stations.
- Visualized station coverage across metro lines.

---

# Future Improvements

- Demand forecasting using Machine Learning
- Passenger count prediction
- Route recommendation system
- Interactive Streamlit dashboard
- Real-time GTFS integration
- Delay prediction models
- Clustering of stations based on demand

---

# Applications

This project can support:

- Smart City Planning
- Public Transport Analytics
- Capacity Planning
- Route Optimization
- Infrastructure Expansion
- Operational Decision Support

---
