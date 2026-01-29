# de-zoomcamp-2026-homework
Code repository for the 2026 Data Engineering Zoomcamp coursework.

## Overview
This repo collects homework artifacts for the course modules. Each module folder contains its own exercises and workflow definitions.

## Module 1: Docker & SQL
Notebook: `Module 1 Homework: Docker & SQL.ipynb`

Short summary:
- Spins up a local Postgres container and connects to it.
- Loads NYC taxi data and runs basic SQL queries for exploration and validation.

## Module 2: Workflow Orchestration (Kestra)
Folder: `Module 2 Homework: Workflow Orchestration/`

### Foreach implementation (used in multiple questions)
Flow: `Module 2 Homework: Workflow Orchestration/postgres_taxi_foreach.yaml`

This flow builds a list of months from a `start` and `end` period (YYYY-MM) and iterates over them with `ForEach`. Each item triggers the base flow (`postgres_taxi`) via a `Subflow`, passing `taxi`, `year`, and `month`. This makes it reusable for multiple questions that need multi-month ingestion.

### Question 1: Uncompressed file size (yellow, 2020-12)
Exact question:
> Within the execution for Yellow Taxi data for the year 2020 and month 12: what is the uncompressed file size (i.e. the output file yellow_tripdata_2020-12.csv of the extract task)?

Implementation details:
- Flow: `Module 2 Homework: Workflow Orchestration/postgres_taxi.yaml`
- Task `csv_size` computes the uncompressed CSV size immediately after `extract` using `stat` on the extracted file.
- Task `size_label` stores the result in the execution labels so it remains accessible even after `purge_files`.

How to retrieve the value in Kestra UI:
1) Run the flow with `taxi=yellow`, `year=2020`, `month=12`.
2) Open the execution and check **Labels**.
3) Read `size_bytes` (this is the uncompressed file size in bytes).

## Module 2: Quiz Questions (Commands / Steps)
Use the commands below to reproduce the answers. Run them against the local `pgdatabase` container.

### Q1: Uncompressed file size for yellow 2020-12 (extract output)
Kestra UI steps (uses the `size_label` from `postgres_taxi.yaml`):
1) Run flow `postgres_taxi` with `taxi=yellow`, `year=2020`, `month=12`.
2) Open the execution → **Labels** → read `size_bytes`.
3) Convert to MiB if needed: `MiB = size_bytes / 1024 / 1024`.


### Q3: Rows for Yellow Taxi data for all CSV files in 2020
Run the flow `postgres_taxi_foreach` (or manually run `postgres_taxi`) for all months in 2020 for `taxi=yellow` to load the data into Postgres and then run the query below:

```
docker exec -it de-zoomcamp-2026-homework_pgdatabase_1 psql -U root -d ny_taxi -c "
SELECT COUNT(*) AS yellow_2020
FROM yellow_tripdata
WHERE filename LIKE 'yellow_tripdata_2020-%';
"
```

### Q4: Rows for Green Taxi data for all CSV files in 2020
Run the flow `postgres_taxi_foreach` (or manually run `postgres_taxi`) for all months in 2020 for `taxi=green` to load the data into Postgres and then run the query below:

```
docker exec -it de-zoomcamp-2026-homework_pgdatabase_1 psql -U root -d ny_taxi -c "
SELECT COUNT(*) AS green_2020
FROM green_tripdata
WHERE filename LIKE 'green_tripdata_2020-%';
"
```

### Q5: Rows for Yellow Taxi data for March 2021
Run the flow `postgres_taxi` for `taxi=yellow`, `year=2021`, `month=03` to load the data into Postgres and then run the query below:
```
docker exec -it de-zoomcamp-2026-homework_pgdatabase_1 psql -U root -d ny_taxi -c "
SELECT COUNT(*) AS yellow_2021_03
FROM yellow_tripdata
WHERE filename = 'yellow_tripdata_2021-03.csv';
"
```
