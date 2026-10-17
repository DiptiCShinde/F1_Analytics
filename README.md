# Formula 1 Analytics and Machine Learning

A Databricks lakehouse project for ingesting Formula 1 data, cleaning it into analytics-ready tables, building driver and constructor statistics, and experimenting with race-winner prediction. The notebooks use Apache Spark, Delta tables, Azure Data Lake Storage Gen2, and a Unity Catalog catalog named `f1_catalog`.

## Project workflow

The notebooks are intended to run in this order:

1. **`location.ipynb`** — creates the bronze, silver, and gold schemas and external locations in Unity Catalog.
2. **`All_ingestion_files.ipynb`** — reads raw CSV and JSON files from Azure storage and writes Delta tables to the bronze layer.
3. **`all_ceaning_files.ipynb`** — cleans and standardizes bronze tables and writes them to the silver layer.
4. **`gold_file.ipynb`** — creates gold tables and queries for driver performance, qualifying versus race position, constructor standings, race summaries, yearly rankings, championship winners, circuit analysis, and a driver performance score.
5. **`model_building.ipynb`** — creates driver/race features and trains/evaluates Random Forest and XGBoost winner classifiers.

## Data and cloud setup

The notebooks expect Formula 1 source data in an Azure Data Lake Storage Gen2 account, under a `raw` container with folders/files for circuits, races, constructors, drivers, results, pit stops, lap times, and qualifying. They reference paths in the form:

```text
abfss://raw@<storage-account>.dfs.core.windows.net/<dataset-path>
```

The location notebook also contains bronze, silver, and gold storage locations. Before running the notebooks:

- Replace the example storage account/container paths with your own Azure Data Lake paths.
- Create/configure a Databricks storage credential and grant it access to the containers.
- Update the credential name in `location.ipynb` to match the credential configured in your workspace.
- Ensure your Databricks identity has permission to create external locations, catalogs, schemas, and tables.
- Upload or otherwise provide the source data in the expected folders. The notebooks do not include the raw datasets.

Never commit cloud credentials, account keys, tokens, or other secrets to GitHub. Keep credentials in Databricks secrets or the workspace’s secure credential configuration.

## Running the project

Run the notebooks in the workflow order above from a Databricks workspace configured for Azure storage and Unity Catalog. The SQL cells use `%sql`, and several notebooks use Databricks-specific features such as `display`, `dbutils`, and notebook magics; they will not run unchanged in a standard local Jupyter setup.

The ML notebook uses Spark MLlib and also installs XGBoost with `%pip install xgboost`. XGBoost evaluation converts the Spark data to Pandas, so it requires enough driver memory for that data.

## Outputs

The notebooks create Delta tables under `f1_catalog`:

- `bronze`: ingested source tables.
- `silver`: cleaned tables such as circuits, races, drivers, constructors, results, qualifying, pit stops, and lap times.
- `gold`: driver and constructor performance metrics, race summaries, rankings, champions, circuit analysis, and performance scores.
- `ml_process`: driver/race features used by the machine-learning experiments.

The gold tables can be used as a source for a Power BI report. The `.pbix` dashboard file and its screenshot can be added to this repository separately if you want to publish the dashboard project too.

## Repository contents

Place the notebooks in a `notebooks/` directory and add your Power BI file and dashboard preview if applicable. A suggested layout is:

```text
f1-analytics/
├── README.md
├── notebooks/
│   ├── location.ipynb
│   ├── All_ingestion_files.ipynb
│   ├── all_ceaning_files.ipynb
│   ├── gold_file.ipynb
│   └── model_building.ipynb
├── dashboard/
│   └── f1-analytics.pbix
└── images/
    └── f1-dashboard.png
```

Raw datasets are not required in GitHub if they are large; document the source and setup instead. Include only small sample data if it is permitted to redistribute.

## Notes and known issues to review

- `all_ceaning_files.ipynb` appears to drop/rename driver columns using names that may not match the already-normalized columns produced by the ingestion notebook. Check the driver-cleaning section against the actual bronze table schema before running it.
- The notebooks use `overwrite` when writing Delta tables, so rerunning them replaces the corresponding table contents.
- The winner-prediction target is imbalanced (race winners are a small fraction of driver-race rows). The model notebook experiments with class weighting and thresholds; accuracy by itself is not enough to judge model quality. Review precision, recall, and the confusion matrix.
- The notebooks contain some exploratory/repeated cells and assume the referenced tables already exist at each stage.

## Technologies

- Azure Databricks
- Apache Spark / PySpark and Spark SQL
- Delta Lake
- Unity Catalog
- Azure Data Lake Storage Gen2
- Spark MLlib and XGBoost
- Power BI (optional dashboard layer)
