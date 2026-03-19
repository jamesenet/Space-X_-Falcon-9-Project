# SpaceX Falcon 9 — Predicting First-Stage Landing Success
**Predictive Analytics | Python · SQL · Scikit-learn · Folium**

[![Python](https://img.shields.io/badge/Python-3.10-blue?style=flat-square)](https://python.org)
[![ML](https://img.shields.io/badge/Scikit--learn-Classification-orange?style=flat-square)](https://scikit-learn.org)
[![SQL](https://img.shields.io/badge/SQL-Data%20Extraction-blue?style=flat-square)](https://sqlite.org)

---

## 1. Business Problem

SpaceX's cost advantage in the commercial launch market comes largely from **reusing the Falcon 9 first stage booster**. A successful landing reduces launch cost from ~$165M to ~$62M — a $100M+ swing per mission.

For a competing launch provider or investor analysing the market:

> *"Can we predict whether a Falcon 9 first stage will successfully land, based on pre-launch variables? And which factors most reliably determine that outcome?"*

A reliable prediction model would allow:
- Competitive bid pricing (factor in landing probability when quoting against SpaceX)
- Insurance risk assessment for payload customers
- Engineering prioritisation for landing system improvements

---

## 2. Data

| Source | Method | Fields |
|--------|--------|--------|
| SpaceX REST API | `requests` + JSON parsing | Launch date, payload mass, orbit, launch site, booster version |
| Wikipedia (web scraping) | `BeautifulSoup` | Historical launch outcomes, grid fin usage, legs deployed |
| SQLite database | Custom schema | Structured for SQL querying |

**Data preparation:**
- Converted outcome labels to binary: `1` = successful landing, `0` = failure/no attempt
- One-hot encoded categorical features: orbit type, launch site, booster version
- Handled class imbalance (~34% success rate in early launches vs ~80%+ in recent) by stratifying train/test split by year
- Removed launches with no landing attempt (ocean delivery) from classification scope

---

## 3. Approach

```
SpaceX API + web scraping
        ↓
Data wrangling & feature engineering (Pandas)
        ↓
SQL queries — launch site analysis, payload distribution, success rate by orbit
        ↓
EDA — Seaborn charts, Folium geospatial map of launch sites
        ↓
Feature selection — payload mass, orbit, launch site, booster version, flight number
        ↓
Model training — Logistic Regression, SVM, Decision Tree, KNN
        ↓
GridSearchCV hyperparameter tuning (5-fold CV)
        ↓
Best model: Decision Tree → 83% accuracy on holdout set
        ↓
Feature importance analysis
```

---

## 4. SQL Analysis Highlights

### Success Rate by Launch Site
```sql
SELECT
    launch_site,
    COUNT(*) AS total_launches,
    SUM(CASE WHEN landing_outcome = 1 THEN 1 ELSE 0 END) AS successful,
    ROUND(100.0 * SUM(CASE WHEN landing_outcome = 1 THEN 1 ELSE 0 END) / COUNT(*), 1)
        AS success_rate_pct
FROM launches
GROUP BY launch_site
ORDER BY success_rate_pct DESC;
```

### Payload Mass Distribution by Orbit
```sql
SELECT
    orbit,
    ROUND(MIN(payload_mass_kg), 0) AS min_payload,
    ROUND(AVG(payload_mass_kg), 0) AS avg_payload,
    ROUND(MAX(payload_mass_kg), 0) AS max_payload
FROM launches
WHERE payload_mass_kg IS NOT NULL
GROUP BY orbit
ORDER BY avg_payload DESC;
```

---

## 5. Key Findings

### Finding 1 — Payload mass is the strongest continuous predictor
Heavier payloads significantly reduce landing probability. Launches with payload > 5,500 kg show a **22 percentage point lower** landing success rate compared to payloads under 3,000 kg. The likely mechanism: heavier payloads leave less propellant margin for the landing burn.

### Finding 2 — Launch site matters independently of payload
CCAFS LC-40 had the highest success rate of all sites analysed. KSC LC-39A showed the highest absolute launch volume. Site effect persists after controlling for payload mass, suggesting site-specific infrastructure and trajectory geometry are independent factors.

### Finding 3 — Booster reuse is positively associated with success
Boosters on their 2nd–4th flights show **higher** landing success rates than first flights. This contradicts the intuition that wear reduces reliability — the evidence suggests SpaceX's refurbishment process is effective, and first flights carry higher variance.

### Finding 4 — LEO orbits are most forgiving for landing attempts
Low Earth Orbit (LEO) missions have the highest landing success rate. High-energy orbits (GTO, HEO) require more propellant for the mission itself, leaving less for landing — consistent with the payload mass finding.

### Model Performance Summary

| Model | CV Accuracy | Test Accuracy |
|-------|-------------|---------------|
| Logistic Regression | 77.3% | 75.0% |
| SVM (RBF kernel) | 81.4% | 80.6% |
| **Decision Tree (tuned)** | **84.1%** | **83.3%** |
| KNN (k=5) | 79.2% | 77.8% |

---

## 6. Geospatial Analysis

A Folium interactive map was built to visualise:
- All launch site locations (markers with success rate tooltips)
- Proximity to coastline (correlated with trajectory options)
- Clustering of successful vs failed landings by geography

The map is included as `launch_sites_map.html` in the repo.

---

## 7. Files in This Repository

```
├── Winning the space race with data science.pdf   # Full project report
├── notebooks/
│   ├── 01_data_collection_api.ipynb
│   ├── 02_data_wrangling.ipynb
│   ├── 03_eda_sql.ipynb
│   ├── 04_eda_visualisation.ipynb
│   ├── 05_folium_map.ipynb
│   └── 06_machine_learning.ipynb
├── launch_sites_map.html          # Interactive Folium map
└── README.md
```

---

## 8. Tools & Libraries

| Tool | Use |
|------|-----|
| `requests` / `BeautifulSoup` | API calls & web scraping |
| Pandas / NumPy | Wrangling & feature engineering |
| SQLite / SQL | Data exploration queries |
| Seaborn / Matplotlib | EDA charts |
| Folium | Geospatial visualisation |
| Scikit-learn | Classification models, GridSearchCV |

---

## 9. How to Run

```bash
# Clone the repository
git clone https://github.com/jamesenet/My-Portfolio-

# Install dependencies
pip install pandas numpy scikit-learn matplotlib seaborn folium beautifulsoup4 requests

# Run notebooks in sequence (01 → 06)
jupyter notebook notebooks/01_data_collection_api.ipynb
```

> **Note:** SpaceX API calls in notebook 01 require an internet connection. Subsequent notebooks can run from the cached CSV output.

---

*Case study by Cornelius Enetomhe · [LinkedIn](https://www.linkedin.com/in/cornelius-enetomhe-01688a266/) · [Portfolio](https://jamesenet.github.io/CorneliusEnetomhe.github.io/)*
