# End-to-End Retail Lakehouse Pipeline (Databricks Free + S3 + Delta)

End-to-end data engineering project using Databricks Free Edition implementing a Lakehouse (Medallion) ETL pipeline in an FMCG M&A scenario where a large retailer acquires a smaller company ("Sportsbar"). The pipeline consolidates data from both companies into a unified lakehouse and serves BI-ready tables.

## Tech Stack
- Databricks Free Edition (Spark, SQL)
- Delta Lake (ACID tables, MERGE, Change Data Feed)
- Amazon S3 (landing/processed storage)
- Python (PySpark) + SQL
- BI Dashboard + Databricks Genie

## Architecture (Medallion)
- **Bronze:** Ingest raw CSVs from S3 `landing/`, add ingestion metadata, write Delta tables
- **Silver:** Clean/standardize data (types, dates, dedupe, conformance), join dimensions
- **Gold:** Analytics-ready dims/facts + consolidation into parent company model

## Data Model (Gold)
**Small company (Sportsbar)**
- `fmcg.gold.sb_dim_products`
- `fmcg.gold.sb_dim_customers`
- `fmcg.gold.sb_dim_gross_price`
- `fmcg.gold.sb_fact_orders` (daily grain)

**Parent company**
- `fmcg.gold.dim_products`
- `fmcg.gold.dim_customers`
- `fmcg.gold.dim_gross_price`
- `fmcg.gold.fact_orders` (monthly grain)

## Key Engineering Patterns
### Full Load (historical)
1. Build child daily fact (`sb_fact_orders`)
2. Aggregate to monthly grain
3. `MERGE` into parent `fact_orders`

### Incremental Load
1. New CSVs arrive in S3 `landing/`
2. Ingest to Bronze + stage “just arrived” data
3. Build Silver staging + Gold child daily rows
4. Identify impacted months from incremental data
5. Recompute only impacted months and `MERGE` into parent `fact_orders`
6. Move processed files from `landing/` → `processed/`

## How to Run (Order)
1. Setup catalog/schemas (`fmcg`, `bronze`, `silver`, `gold`)
2. Create date dimension (`dim_date`)
3. Load/clean small-company dimensions (products/customers/pricing)
4. Run historical full load
5. Run incremental pipeline (drop new CSVs into S3 `landing/`)

## Notes
- This repo contains Databricks notebook exports (.py). Import them into Databricks Workspace to run.
- Do not commit AWS credentials; use Databricks secrets / instance profile / connector settings.
