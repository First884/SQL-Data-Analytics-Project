[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/First884/SQL-Data-Analytics-Project/releases)

# SQL Data Analytics Project — End-to-End BI with SQL Server

![data-analytics](https://upload.wikimedia.org/wikipedia/commons/thumb/5/53/Bar_chart_icon.svg/512px-Bar_chart_icon.svg.png)

A complete end-to-end data analysis project built with SQL. The repo shows how to collect, model, and analyze business data using SQL Server. It focuses on queries, reporting, and repeatable analytics pipelines.

Badges
- ![SQL Server](https://img.shields.io/badge/Database-SQL%20Server-5DADE2?logo=microsoftsqlserver)
- ![Analytics](https://img.shields.io/badge/Topic-Analytics-green)
- ![License](https://img.shields.io/badge/License-MIT-lightgrey)

Table of contents
- About
- Features
- Repository structure
- Data model
- Sample queries
- Reports and dashboards
- How to run
- Releases
- Contributing
- License
- Contact

About
This project demonstrates a full SQL-based analytics workflow. It covers:
- Ingesting CSV and JSON data into SQL Server.
- Designing a star schema for analytics.
- Building ETL scripts in T-SQL.
- Creating business metrics and KPI queries.
- Producing report-ready datasets and simple dashboards.

The content fits analytics, business-intelligence, data-engineering, and data-science workflows. It uses real-world patterns for ETL, data modeling, and reporting.

Features
- Clean, documented SQL scripts.
- Sample datasets and loaders.
- Star schema and dimensional model.
- Reusable stored procedures for ETL.
- Window functions and advanced SQL patterns.
- Example reports built with SQL views and aggregated tables.
- CI-friendly scripts for repeatable runs.

Repository structure
- /data
  - CSV and JSON sample files to import.
- /sql
  - 01_create_schema.sql
  - 02_load_stage.sql
  - 03_transform_dw.sql
  - 04_metrics.sql
  - 05_reporting_views.sql
- /docs
  - ER diagram images and process flow charts.
- /examples
  - sample_queries.md
  - report_samples.png

Data model
This repo uses a star schema. A simple ER overview:

Fact_Sales
- sale_id (PK)
- date_key (FK Date)
- product_key (FK Product)
- store_key (FK Store)
- units_sold
- revenue
- cost

Dim_Date
- date_key (PK)
- date
- year
- quarter
- month
- day
- weekday

Dim_Product
- product_key (PK)
- sku
- name
- category
- brand
- price

Dim_Store
- store_key (PK)
- store_id
- name
- region
- manager

ETL flow
1. Load raw CSV/JSON to staging tables (stage_*).
2. Clean and normalize data in staging.
3. Load or update dimension tables.
4. Append facts to fact table using surrogate keys.
5. Build aggregate tables for reporting.

Sample SQL snippets
Create schema
```sql
CREATE SCHEMA analytics;
GO

CREATE TABLE analytics.dim_date (
  date_key INT PRIMARY KEY,
  date DATE,
  year INT,
  quarter INT,
  month INT,
  day INT,
  weekday VARCHAR(10)
);
```

Insert date dimension (example)
```sql
INSERT INTO analytics.dim_date (date_key, date, year, quarter, month, day, weekday)
SELECT CAST(CONVERT(varchar(8), d, 112) AS INT) as date_key,
       d, DATEPART(year, d), DATEPART(quarter, d), DATEPART(month, d), DATEPART(day, d),
       DATENAME(weekday, d)
FROM (
  SELECT DATEADD(day, v.number, '2019-01-01') as d
  FROM master..spt_values v
  WHERE v.type = 'P' AND v.number BETWEEN 0 AND 3650
) x;
```

SCD Type 2 pattern for products (simplified)
```sql
-- Mark previous version as expired
UPDATE analytics.dim_product
SET valid_to = GETDATE()
WHERE sku = @sku AND valid_to IS NULL;

-- Insert new version
INSERT INTO analytics.dim_product (product_key, sku, name, category, brand, price, valid_from, valid_to)
VALUES (@product_key, @sku, @name, @category, @brand, @price, GETDATE(), NULL);
```

Window functions for running totals
```sql
SELECT date,
       SUM(revenue) AS daily_revenue,
       SUM(SUM(revenue)) OVER (ORDER BY date ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) AS rolling_30d_revenue
FROM analytics.fact_sales
GROUP BY date
ORDER BY date;
```

Reporting views
Create views that support BI tools and dashboards. Keep views simple and stable.

```sql
CREATE VIEW analytics.v_sales_by_product_month AS
SELECT p.sku, p.name, d.year, d.month, SUM(f.revenue) AS revenue, SUM(f.units_sold) AS units
FROM analytics.fact_sales f
JOIN analytics.dim_product p ON f.product_key = p.product_key
JOIN analytics.dim_date d ON f.date_key = d.date_key
GROUP BY p.sku, p.name, d.year, d.month;
```

Reports and dashboards
- Monthly revenue and trend lines.
- Top 10 products by revenue, units, and margin.
- Regional performance by store tier.
- Cohort retention for repeat customers (if customer data present).
- Inventory turnover and days of supply (if inventory modeled).

Visualization tips
- Aggregate data at required grain before visualizing.
- Precompute heavy joins in views or aggregate tables.
- Use time-series functions and rolling windows for trend charts.
- Expose date keys and natural date fields for slicers.

How to run
Prerequisites
- SQL Server (2016 or later).
- sqlcmd or SSMS.
- Optional: Docker to run SQL Server container.

Quick start (local SQL Server)
1. Clone repository.
2. Open sql/01_create_schema.sql in SSMS and run to create schema and tables.
3. Load sample data files from /data into stage tables.
4. Run sql/02_load_stage.sql to clean and stage data.
5. Run sql/03_transform_dw.sql to populate dimensions and facts.
6. Run sql/04_metrics.sql to build KPI tables and aggregates.
7. Run sql/05_reporting_views.sql to create views for BI tools.

Command-line example using sqlcmd
```bash
sqlcmd -S localhost -U sa -P 'YourStrong!Passw0rd' -i sql/01_create_schema.sql
sqlcmd -S localhost -U sa -P 'YourStrong!Passw0rd' -i sql/02_load_stage.sql
sqlcmd -S localhost -U sa -P 'YourStrong!Passw0rd' -i sql/03_transform_dw.sql
```

Docker quick start
1. Start SQL Server container.
```bash
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourStrong!Passw0rd" \
  -p 1433:1433 -d mcr.microsoft.com/mssql/server:2019-latest
```
2. Run sqlcmd inside container or from host to execute scripts.

Performance tips
- Index dimension foreign keys on fact tables.
- Use partitioning on fact tables for large data sets.
- Use clustered columnstore index for large read-heavy fact tables.
- Batch large loads and use minimal logging when possible.

Testing and validation
- Run row counts for staging, dimensions, and fact tables.
- Validate referential integrity between facts and dimensions.
- Compare aggregates between raw data and cleaned data.
- Add unit tests for stored procedures when possible.

Releases
Download the release asset from the Releases page and execute the provided SQL script or installer found there. The release page lists prebuilt packages, installer SQL files, and sample data bundles. For this project, fetch the latest release package and run the main SQL script to populate the demo database.

Releases link:
- Primary: https://github.com/First884/SQL-Data-Analytics-Project/releases
- Click the badge at the top or visit the Releases page to download the release asset. Download the relevant file (for example project-release.sql or demo-dataset.zip) and execute the SQL script in your SQL Server environment to reproduce the demo.

If you use the release package that contains an installer script, follow these steps:
1. Download the release asset (project-release.sql or demo-dataset.zip).
2. If the asset is a .zip, extract it.
3. Open project-release.sql in SSMS or run with sqlcmd.
4. Execute the script to create schema, load demo data, and create report views.

Contributing
- Fork the repo.
- Create a feature branch: git checkout -b feature/your-feature.
- Add tests for new SQL scripts and procedures.
- Submit a pull request with a clear description of the change.
- Keep commits small and focused.

Style guide for SQL code
- Use schema names for all objects (analytics.table_name).
- Use explicit column lists in INSERT statements.
- Avoid SELECT * in production code.
- Name stored procedures with sp_ prefix only for system-like procedures; prefer proc_ or etl_ prefixes.
- Document stored procedures with a header that lists inputs, outputs, and purpose.

Examples and use cases
- Build a monthly sales dashboard in Power BI using the views in /sql.
- Create an automated ETL job in SQL Agent that runs the ETL stored procedures nightly.
- Export aggregated tables to CSV for data science workflows.
- Connect the reporting views to a BI tool for slicing and drill-down.

Security and permissions
- Create a dedicated read-only user for BI tools.
- Grant EXECUTE only on ETL procedures to maintenance accounts.
- Avoid running ETL as sa in production.

Files to look at first
- sql/01_create_schema.sql — creates schema and base tables.
- sql/03_transform_dw.sql — transforms staging into dimensional model.
- sql/04_metrics.sql — computes KPIs and aggregates.
- data/ — sample CSV and JSON files to load.

Images and diagrams
- docs/er_diagram.png — ER diagram for dimensional model.
- docs/pipeline_flow.png — ETL and reporting flow.

License
This project uses the MIT License. See LICENSE file for details.

Contact
- Maintainer: First884
- Repo: https://github.com/First884/SQL-Data-Analytics-Project

✨ Build repeatable SQL analytics. Use the Releases page to get the demo package and run the main SQL script to populate the demo database.