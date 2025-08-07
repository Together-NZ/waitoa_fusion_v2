# 🐔 Waitoa Fusion: Custom Store-Level Opportunity Scoring

This repository contains the full pipeline used for **audience segmentation and opportunity scoring** for the Waitoa Fresh Chicken brand. The goal was to identify and rank New Zealand supermarkets where Waitoa has the **highest potential for growth**, by integrating multiple datasets and applying a data science–driven clustering methodology.

---

## 📦 Project Summary

The project leverages:
- **Sales data** (2024–2025) across three product categories: Fresh (Butchery), Frozen, and Deli
- **Store network metadata** (NZ supermarkets with geocoordinates)
- **Stats NZ SA2 population data**
- **Geo-mapping and fuzzy-matching algorithms**
- **Opportunity scoring based on volume/sales gaps + local population**
- **K-Means clustering with spatial constraints**
- **Interactive map outputs and Excel workbooks for marketing activation**

---

## 🧱 Pipeline Overview

### 1. 🚚 Data Ingestion (GCS Buckets)
- Connects to Google Cloud Storage buckets using `google.cloud.storage`
- Pulls:
  - `together-dunnhumby-*.csv` files for historical category-level sales
  - Monthly `.xlsx` files for latest 2025 sales (by product type)
  - Stats NZ SA2 shapefiles + population estimates
  - NZ supermarkets list with geospatial coordinates

### 2. 🧹 Data Processing
- Cleans column names, fills missing values, and calculates:
  - `TOTAL_SALES`
  - `TOTAL_VOLUME`
- Aggregates data by:
  - `STORE_NAME`, `STORE_CODE`, `PRODUCT_NAME`, and `PRODUCT_CODE`
- Segments products into **Waitoa vs. competitors** based on product naming

### 3. 📊 Market Share Analysis
- Calculates `Waitoa` share of:
  - Total store-level sales
  - Total store-level volume
- Merges with local SA2 population to enrich with catchment context

### 4. 🤖 Fuzzy Store Matching
- Performs **fuzzy string matching** between Waitoa sales data and the NZ supermarkets master list
- Joins successfully matched stores with full location metadata (lat/lon)

### 5. 🧠 Opportunity Scoring
For each matched store, computes:
- `opp_sales` = (competitor sales - Waitoa sales)
- `opp_volume` = (competitor volume - Waitoa volume)
- Normalized values for:
  - `opp_sales`, `opp_volume`, `population`
- Composite weighted score:
  - `50%` sales opportunity
  - `30%` volume opportunity
  - `20%` SA2 population

Adds `rank` column based on descending opportunity score.

---

## 📍 Clustering Logic

Clustering is performed using **K-Means** on lat/lon coordinates of stores.

### Key details:
- Top 100 stores in **Fresh**, **Frozen**, and **Deli** are clustered separately
- Non-top stores are also clustered into their own groups
- Radius logic is enforced:
  - `MIN_RADIUS_KM` and `MAX_RADIUS_KM` bounds
  - Dynamic radius based on geodesic distance to cluster centroid
- Non-Waitoa stores within cluster radii are marked for adjacent market targeting

---

## 🗺️ Deliverables

### ✅ Output Files
- `waitoa_top_100_stores_analysis.xlsx`
  - Fresh and Frozen top 100 clusters
- `waitoa_others_stores_analysis.xlsx`
  - Remaining Fresh and Frozen clusters

### 🗺️ Interactive Maps
- `fresh_stores_clusters_map.html`
- `frozen_stores_clusters_map.html`
- `fresh_stores_others_map.html`
- `frozen_stores_others_map.html`

---

## 🧠 Tools & Dependencies

- `pandas`, `numpy`
- `geopandas`, `folium`, `geopy`
- `fuzzywuzzy`, `sklearn.cluster`
- `google-cloud-storage`
- `openpyxl`

---

## 🧪 Use Cases

- Identify high-growth-potential stores for **digital media activation**
- Overlay clustered zones for **hyper-local targeting**
- Refresh opportunity rankings as **monthly data updates**

---

## 📌 Notes

- This project assumes access to Google Cloud Storage buckets with valid service credentials
- All outputs are suitable for use in media planning, location targeting, and campaign prioritization

---

## 📂 File Structure

```plaintext
waitoa_fusion_v2.py          # Main processing pipeline
│
├── data/
│   ├── historical_sales/    # Dunnhumby .csvs (2024)
│   ├── 2025_sales/          # Monthly Excel files (2025)
│   ├── supermarkets/        # NZ supermarket network
│   ├── stats_nz/            # SA2 shapefiles + population
│
├── outputs/
│   ├── waitoa_top_100_stores_analysis.xlsx
│   ├── waitoa_others_stores_analysis.xlsx
│   ├── *.html maps
