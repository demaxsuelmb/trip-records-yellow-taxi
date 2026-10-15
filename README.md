# trip-records-yellow-taxi

End-to-end lakehouse on NYC TLC yellow taxi trip records with PySpark and Delta Lake: bronze/silver/gold layers, data quality with quarantine, idempotent MERGE, star schema, Spark performance tuning (shuffle, skew, broadcast), incremental ingestion with Auto Loader, and Databricks Workflows with Unity Catalog.

- Plan and modules: [`docs/PLAN.md`](docs/PLAN.md)
- Failure experiments and what each one taught: [`docs/FAILURES.md`](docs/FAILURES.md)
