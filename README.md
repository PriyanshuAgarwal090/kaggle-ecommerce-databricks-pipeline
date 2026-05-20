# Kaggle E-Commerce Data Engineering Pipeline (Bronze Layer)

An end-to-end data engineering pipeline built on Databricks utilizing modern **Unity Catalog (UC)** governance architectures to securely ingest and catalog real-world e-commerce data from Kaggle.

## Architecture Overview
* **Orchestration & Compute:** Databricks Shared Unity Catalog Cluster (Spark Connect Engine)
* **Storage Layer:** Unity Catalog Managed Volumes (Staging) & Managed Delta Tables (Storage)
* **Data Layer Architecture:** Medallion Architecture (Current Phase: **Bronze Layer Completed**)
* **Source Dataset:** Kaggle E-Commerce Dataset

---

## Architectural Challenge: Overcoming `[DBFS_DISABLED]`

### The Problem
During initial pipeline development, attempts to write directly using legacy root paths or local driver files via Spark endpoints (`spark.read.csv("/tmp/...")`) triggered security failures:
> `[DBFS_DISABLED] Public DBFS root is disabled. Access is denied on path... SQLSTATE: 56038`

On high-concurrency, shared modern Unity Catalog clusters using Spark Connect, direct interaction between the Spark engine and the local file system driver is prohibited to enforce absolute data isolation.

### The Solution: Unity Catalog Volumes
To bypass legacy root restrictions securely, the pipeline was refactored to utilize **Unity Catalog Volumes** as a secure staging area. 

1. **Secure API Token Retrieval:** Programmatic integration via Databricks Interactive Widgets, removing hardcoded credentials (`kaggle.json`) from code control.
2. **Staging Zone Isolation:** Streamed raw multi-GB CSV files directly from the Kaggle API using native Python libraries into a secure Unity Catalog Volume path: `/Volumes/main/ecommerce/raw_data/`
3. **Decoupled Distributed Loading:** Spark Connect successfully targets the secure cloud volume path, ingesting the files into high-performance distributed Spark DataFrames without filesystem constraint violations.

---

## Project Structure & Current Progress

### Phase 1: Ingestion & Bronze Layer (Completed)
* **Source Ingestion:** Automated Python download handler pulls raw data files (`orders.csv`, `order_items.csv`, `products.csv`, `users.csv`) dynamically.
* **Bronze Storage:** Data is stored as schema-inferred, history-tracked **Managed Delta Tables** within the Unity Catalog metastore.

```sql
-- Target Catalog Strategy
main.ecommerce.bronze_orders
main.ecommerce.bronze_order_items
main.ecommerce.bronze_product
main.ecommerce.bronze_user