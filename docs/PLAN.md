# Project plan

End-to-end lakehouse on the [NYC TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
(yellow taxi), built with PySpark and Delta Lake, locally and on Databricks.

## Why this dataset

- Monthly Parquet files (~3M rows each): enough volume for real shuffle and partitioning decisions.
- **Natural skew**: a few pickup zones concentrate most trips.
- A small dimension (taxi zone lookup, 265 rows): the textbook broadcast-join case.
- **Real schema drift** across years (column types change), the kind of problem bronze has to absorb.
- Files arrive month by month, which makes incremental ingestion (Auto Loader) realistic.

## Environment

- **Local (WSL):** `pyspark` + `delta-spark` + `pytest`. Unit tests run here, and the full Spark UI
  (`localhost:4040`) is used for performance work.
- **Databricks (Free Edition):** Unity Catalog, Auto Loader, Workflows and lineage.
- Data is never committed: `data/` is ignored, and the code downloads the public files.

## Engineering rules

- Code lives in a Python package (`src/`), not in loose notebooks. Notebooks are thin entry points.
- Pure, testable transformation functions; type hints; logging instead of prints.
- Every module includes a **failure experiment**: the failure is triggered on purpose, diagnosed from the
  error message, fixed, and written down in [`FAILURES.md`](FAILURES.md) (symptom → root cause → fix → prevention).
- No credentials in the repo. Secrets go in `.env` (ignored) or Databricks secret scopes.

## Modules

| # | Module | Failure experiment | Done when |
|---|---|---|---|
| M0 | **Setup:** package, `pyproject.toml`, pytest with a local Delta-enabled SparkSession, Databricks workspace and catalog | — | `pytest` green with one Delta write/read test |
| M1 | **Bronze:** raw ingestion of 12 months + metadata (`_ingested_at`, `_source_file`) | **Schema drift**: read files from different years together | Bronze in Delta with an explicit schema; drift documented |
| M2 | **Silver:** typing, cleaning, quality rules, quarantine table, **MERGE** for late-arriving data | **MERGE with duplicate source keys** ("multiple source rows matched") | Quarantine with reason codes; MERGE is idempotent (running twice changes nothing) |
| M3 | **Gold:** star schema (`fact_trips`, `dim_zone`, `dim_date`) and aggregates | **Corrupted load**, recovered with time travel / `RESTORE` | Model documented; restore demonstrated |
| M4 | **Performance tuning** (local, Spark UI) | (a) wrong `spark.sql.shuffle.partitions`; (b) **skew** on the hot zone with AQE off, then salting and AQE; (c) broadcast off vs on; (d) **small files** → `OPTIMIZE` / Z-ORDER or liquid clustering | Before/after table with duration, task count and shuffle bytes for each case |
| M5 | **Streaming:** Auto Loader on a landing folder, one monthly file at a time | (a) delete the checkpoint and observe reprocessing; (b) file with a new column → `_rescued_data` / schema evolution | Incremental, exactly-once ingestion proven by row counts |
| M6 | **Platform:** Databricks Workflow with task dependencies, Unity Catalog grants, lineage | Job failing mid-run → repair run | Scheduled job; lineage visible from bronze to gold |
| M7 | **Wrap-up:** README with architecture, short ADRs, `FAILURES.md`, cost notes | — | Repository is self-explanatory for a reader |

## Status

| Module | Status |
|---|---|
| M0 | in progress |
| M1–M7 | not started |
