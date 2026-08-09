# Urban Mobility & Economic Productivity — Latin America (2024)

## 🔍 Overview
Data-driven analysis integrating real-time traffic data (TomTom Traffic Index) with urban economic indicators (OECD Cities) across 15 Latin American cities in 7 countries, to evaluate how urban mobility relates to economic productivity. Built a unified city-level dataset via aggregation and inner join, then used correlation analysis to test whether congestion is driven by GDP per capita, population size, or unemployment. Found that congestion is **weakly correlated with GDP per capita (r = 0.28)**, but **strongly driven by population size (r = 0.88)** and **inversely correlated with unemployment (r = -0.69)** — identifying Bogotá and Lima as priority cities for transport infrastructure investment based on their congestion-to-productivity ratio.

## 🎯 Problem Statement
**Business Question:** In which cities should a development bank invest in transport infrastructure to raise economic productivity and urban well-being?

Acting as a data analyst for a development bank, the team needed a data-driven answer to:
- Which cities show high congestion combined with low economic productivity?
- Which show the best combined indicators (efficient mobility + strong economy)?
- Which variables show the strongest relationship with urban development?

## 💡 What I Did

### Phase 1: Data Loading & Exploration
- Loaded two independent sources: `tomtom_traffic.csv` (real-time traffic/congestion records) and `oecd_city_economy.csv` (annual economic indicators by city)
- Inspected structure and data types with `.info()` on both DataFrames
- Identified initial data quality issues: timestamp columns stored as text, economic indicators (`City GDP/capita`, `Unemployment %`, `Population (M)`, `PM2.5`) stored as text using European numeric formatting (`.` as thousands separator, `,` as decimal separator)

### Phase 2: Data Wrangling
Standardized both datasets for reliable merging:
- Renamed all columns to `snake_case` (e.g., `City GDP/capita` → `city_gdp_capita`, `UpdateTimeUTC` → `update_time_utc`)
- Converted `update_time_utc` and `update_time_utc_week_ago` to proper `datetime` objects with `pd.to_datetime()`
- Cleaned European number formatting in the economic dataset: stripped thousands separators, replaced decimal commas with dots, removed `%` signs, then cast to `float`
- Verified zero unexpected nulls in critical fields after conversion

### Phase 3: Year Extraction & Filtering
- Since `traffic` had no explicit year column, extracted it from `update_time_utc` using `.dt.year`
- Identified 2024 as the only year with full overlap between both sources
- Filtered both DataFrames to `year == 2024`

### Phase 4: Aggregation
- The `traffic` dataset contains multiple real-time records per city; grouped by `['city', 'country', 'year']` and calculated the mean for all key mobility metrics (`jams_delay`, `traffic_index_live`, `jams_length_in_kms`, `jams_count`, `mins_delay`, travel time variables) using `.groupby().agg()`
- Produced one row per city-year with consolidated annual averages

### Phase 5: Merging Mobility & Economic Data
- Selected relevant columns from each aggregated dataset
- Executed an **inner join** on `['city', 'year']`, keeping only cities present in both sources
- Result: a single dataset with one row per city (2024), combining mobility and economic indicators

### Phase 6: Visualization & Correlation Analysis
- Box plot of `jams_delay` to inspect distribution and detect outliers (`showmeans=True`)
- Histogram of `city_gdp_capita` distribution
- Bar chart comparing `jams_delay` vs. `city_gdp_capita` by city (noted scale disparity between the two variables)
- Correlation matrix and heatmap (`sns.heatmap`) across `city_gdp_capita`, `jams_delay`, `unemployment_pct`, and `population`
- Derived a custom **congestion-to-productivity ratio** (`jams_delay / city_gdp_capita`) to rank cities by traffic friction relative to economic output

### Phase 7: Export & Documentation
- Exported the final merged dataset as `ladb_mobility_economy_2024_clean.csv` (`index=False`)
- Documented the full workflow in Jupyter Notebook with an executive summary covering context, methodology, findings, limitations, and recommendations

## 🛠️ Technologies Used

| Category | Tools |
|---|---|
| Language | Python |
| Libraries | pandas, numpy, seaborn, matplotlib |
| Techniques | Data wrangling, string/format cleaning, datetime conversion, `groupby`/`.agg()` aggregation, inner join (`pd.merge`), correlation analysis, derived-metric engineering |
| Environment | Jupyter Notebook |
| Output | Clean, unified CSV dataset (one row per city, 2024) |

## 📊 Key Findings

Context → Findings → Implications (C→F→I)

📍 Context: The dataset covers 15 cities across 7 Latin American countries (Brazil, Argentina, Colombia, Uruguay, Mexico, Peru, Chile) for 2024, the only year with full overlap between the TomTom traffic data and OECD economic indicators.

🔍 Findings:

- Highest strategic urgency. Bogotá & Lima have severe traffic friction paired with lower GDP per capita (ratios 0.100 and 0.078), an atypical characteristic for relatively moderate populations compared to megacities
- High-scale congestion. Mexico City & São Paulo, although they have the highest absolute congestion in the dataset (ratios 0.134 and 0.118), are driven primarily by population scale and dense urban activity rather than inefficiency
- Efficient benchmarks. For small cities, Montevideo & Brasília combine high GDP per capita with minimal traffic delay, serving as models of balanced mobility and urban planning. For moderate-sized cities, Buenos Aires and Rio de Janeiro showcase a more efficient and balanced approach between mobility and economic indicators.
- Population is the main driver of congestion (r = 0.88), while GDP per capita is only weakly correlated (r = 0.28) — economic output alone does not explain traffic
- Unemployment moves inversely with congestion (r = -0.69), GDP (r = -0.51), and population (r = -0.57) — lower unemployment tracks with busier, larger, more congested cities
- Data quality flag: Santiago's 2024 GDP/capita ($2,277) contradicts its own 2023 figure (~$22,176) and its established position among top LATAM cities by GDP per capita, pointing to a source-data error rather than a real economic shift
Brazil contributes 9 of the 15 cities in the dataset, a country imbalance that could skew cross-country comparisons

💡 Implications:

- Prioritize Bogotá and Lima: the clearest case for infrastructure investment, combining high traffic friction with comparatively lower economic output
- Treat Mexico City / São Paulo as scale cases: their congestion reflects city size, not fixable inefficiency, a different investment logic than mid-sized, high-friction cities
- Study Rio de Janeiro, Buenos Aires Montevideo and Brasília as reference models for what balanced mobility-productivity outcomes look like
- Exclude or revalidate Santiago's GDP figure before using it in any resourcing decision
- Recommended investment: targeted transit projects in Bogotá and Lima (mass rapid transit, BRT expansion, smart traffic control) for the highest expected economic return per dollar invested
## 📁 Repository Files

```
├── notebook/
│   └── ladb_mobility_economy_project.ipynb   # Full analysis notebook (code + documentation)
├── data/
│   ├── tomtom_traffic.csv                    # Raw traffic/congestion source data
│   ├── oecd_city_economy.csv                 # Raw economic indicators source data
│   └── ladb_mobility_economy_2024_clean.csv  # Final merged, clean dataset (2024)
├── assets/                                   # visuals and infographics
└── README.md                                 # This file
```

## 🚀 How to Use

- Explore the notebook for the complete step-by-step workflow, from raw data to final merged dataset.
- Raw source files (`tomtom_traffic.csv`, `oecd_city_economy.csv`) are included in `data/` to allow full replication of the cleaning and merging process.
- The final CSV (`ladb_mobility_economy_2024_clean.csv`) is ready for further analysis or visualization.
