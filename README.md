# VulnScope: K-means vulnerability tiers for schools and colleges (SIH26012)

K-means clustering of educational institutions by disaster-vulnerability profile, to decide which institutions get disaster-preparedness training first.

| Step in the problem statement | Where it lives |
|---|---|
| 1. Compile infrastructure, location-hazard and past-incident features | `js/data.js`, **Compile features** tab, `data/*.csv` |
| 2. Apply k-means to segment institutions into vulnerability tiers | `js/kmeans.js`, **Cluster with k-means** tab |
| 3. Validate tiers against regional disaster-history records | `validate()` in `js/kmeans.js`, **Validate the tiers** tab |
| Use the result | **Training plan** tab (priority ranking, playbooks, CSV export) |

## Run the web demo

No build step and no install. Open `index.html` in a browser.
For best results serve it locally: `python3 -m http.server 8000` then visit http://localhost:8000.
Fonts load from Google Fonts when online and fall back to system fonts offline.

## What the demo does

- **Overview**: headline numbers, tier mix map (one ring per district, click to see every institution), first ten to train, findings.
- **Compile features**: the 13 features in three groups, a searchable and sortable institution table, a profile drawer per institution, CSV upload and download.
- **Cluster with k-means**: change k (2 to 6), group weights and random seed. Watch k-means converge step by step on a PCA projection. Silhouette, Davies-Bouldin and stability (adjusted Rand index across restarts) are reported. Elbow and silhouette charts for k = 2 to 8. A heatmap shows what sets each tier apart.
- **Validate the tiers**: the clustering never sees district disaster records. Districts are ranked by mean vulnerability and compared with a history score (events, deaths per lakh, schools damaged, closure days) using Spearman correlation, tier-by-history tables and top-district agreement.
- **Training plan**: choose how many institutions can be trained. See the share of training need covered, students reached, hazard modules required, and download the priority list.

## Python version

```
cd python
pip install -r requirements.txt
python vulnscope.py --k 4          # writes ../output/
python vulnscope.py --inst my_schools.csv --hist my_history.csv
```

It runs the same steps with scikit-learn and writes `tiers_and_priority.csv`, `district_validation.csv` and two figures. Its random generator differs from the browser one, so cluster sizes differ slightly, but the validation result matches (Spearman rho about 0.71 on the demo data).

## Data

All demo records are **generated** (`node scripts/export-data.js` rewrites `data/`). District hazard profiles are loosely modelled on regional patterns and are not official figures. Agreement with the generated history shows the method works. It is not evidence about real schools.

To use real data, supply two CSV files (the **Method and data** tab lists the columns):

- `institutions.csv`: `id`, `district`, `seismic_zone`, `flood_risk`, `cyclone_risk`, `landslide_risk`, `building_age`, `structure_quality`, `plinth_height_m`, `enrollment`, `fire_safety`, `has_dm_plan`, `drills_per_year`, `staff_trained_pct`, `dist_hospital_km`, `incidents_10y`, `closure_days_10y`, `damage_level`; optional `name`, `state`, `type`, `management`, `lat`, `lon`.
- `district_history.csv`: `district`, `state`, `events_20y`, `fatalities_per_lakh`, `schools_damaged_pct`, `avg_closure_days`.

Suggested sources: UDISE+ (school facilities and enrolment), BIS seismic zones, BMTPC Vulnerability Atlas, IMD cyclone data, NDMA and state disaster management authority reports.

## Method notes

- Hazard exposure = 0.6 x worst of four hazards + 0.4 x their average.
- Features are z-scored, scaled by group weight, clustered with k-means++ (10 restarts).
- Clusters are ranked by mean vulnerability score (direction-adjusted z-scores, rescaled 0 to 100) to become tiers.
- Priority = vulnerability x (0.35 + 0.65 x training gap) x exposure factor from enrolment.
- Silhouette is low (about 0.1) because risk varies smoothly. Treat tiers as operational bands, and check the stability figure before relying on them.
- 12 districts make a coarse validation. District history cannot confirm any single building.

## Files

```
index.html          page
css/style.css       styles (light and dark)
js/data.js          feature schema, data generator, CSV read and write
js/kmeans.js        k-means++, silhouette, PCA, tiering, priority, validation
js/charts.js        SVG charts and tooltips (no libraries)
js/app.js           interface logic
data/               demo CSVs
python/             scikit-learn pipeline
scripts/            export-data.js
```
