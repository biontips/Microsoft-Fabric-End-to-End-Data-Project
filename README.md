# Microsoft Fabric End-to-End Data Project – NYC Taxi Data

This project demonstrates an **end-to-end analytics solution using Microsoft Fabric** by combining:

- Data Engineering
- Data Factory
- Data Warehousing
- Power BI

The solution processes [New York City Yellow Taxi Trip data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) and builds a complete analytics pipeline from raw data ingestion to reporting.

The primary focus of this project is on **Microsoft Fabric Data Warehouse and Data Factory experiences**, while also integrating Lakehouse storage, data transformation, and reporting.

---

# Dataset

The project uses **NYC TLC Yellow Taxi Trip Record data**.

Each dataset file contains **one month of taxi trip data** in **Parquet format**.

Data fields include:

- Vendor ID
- Pickup datetime
- Dropoff datetime
- Passenger count
- Trip distance
- Pickup location ID
- Dropoff location ID
- Payment type
- Total amount

Additional dataset:

- **Taxi Zone Lookup Table (CSV)** containing:
  - Location ID
  - Borough
  - Zone
  - Service Zone

This lookup table is used to enrich pickup and dropoff location data.

---

# Solution Architecture

The project implements a **modern analytics architecture in Microsoft Fabric**.

**Process Flow**

1. Raw data files are downloaded.
2. Files are uploaded into a Fabric **Lakehouse**.
3. Data is ingested into a **Data Warehouse staging layer**.
4. Data is cleaned and transformed.
5. Data is loaded into a **presentation table**.
6. Power BI consumes the presentation layer to build reports.

---

# Architecture Diagram

## 🏗️ Architecture

The following diagram shows the end-to-end Microsoft Fabric data pipeline architecture.

<img src="Images/Fabric%20Project%20Architecture.png" width="800">


---

# Data Storage Layer

## Lakehouse

A **Fabric Lakehouse** is used to store the raw files.

Folder structure:

```
Files
├── NYCtaxi_yellow
│ ├── yellow_tripdata_2025-01.parquet
│ ├── yellow_tripdata_2025-02.parquet
│ ├── yellow_tripdata_2025-03.parquet
│ ├── yellow_tripdata_2025-04.parquet
│ └── yellow_tripdata_2025-05.parquet
│
└── NYCtaxi_lookup_zones
└── taxi_zone_lookup.csv
```

Each Parquet file represents **one month of taxi trip data**.

---

# Data Warehouse

A **Fabric Data Warehouse** is used to store structured data.

Schemas used in the project:

- **staging** – temporary landing tables
- **dbo** – presentation tables
- **metadata** – pipeline tracking information

---

# Staging Layer

The staging layer contains:

- `staging.NYCtaxi_yellow`
- `staging.taxi_zone_lookup`

Characteristics:

- Raw data loaded from Lakehouse
- Temporary storage before transformation
- Staging tables are **reloaded during each pipeline run**

The `taxi_zone_lookup` table is loaded **once** and remains unchanged.

---

# Metadata Table

A metadata table is used to track processing progress.

**Table**

metadata.processing_log


Stored information includes:

- Pipeline Run ID
- Table processed
- Rows processed
- Latest processed pickup date
- Process timestamp

This table ensures:

- New data is processed only once
- The pipeline knows **which month should be processed next**

---

# Staging Pipeline

The staging pipeline performs the following steps:

1. Determine the **latest processed month** using the metadata table.
2. Identify the **next data file to process**.
3. Delete existing data from the staging table.
4. Copy the corresponding Parquet file into the staging table.
5. Execute a stored procedure to **remove invalid dates**.
6. Insert processing details into the **metadata table**.

---

# Staging Pipeline Diagram

<img src="Images/Staging Pipeline.png" width="1000">

---

# Data Cleansing

A stored procedure removes **outlier dates** from the staging table.

The procedure deletes records where pickup dates fall **outside the expected month** being processed.

This ensures the staging table only contains valid records.

---

# Presentation Pipeline

The presentation pipeline performs the following steps:

1. Transform staging data
2. Join with lookup table
3. Load into presentation table
4. Insert metadata records for tracking

---

# Presentation Pipeline Diagram

<img src="Images/Presentation Pipeline.png" width="500">

# Transformation

Data transformation combines taxi trip data with zone lookup data.

Key transformations include:

- Vendor ID mapped to vendor names
- Payment type mapped to payment method
- Pickup and dropoff locations joined with lookup table
- Removal of unused columns
- Date formatting

---

# Presentation Layer

Cleaned data is stored in the presentation table:

dbo.NYCtaxi_yellow


Characteristics:

- Contains **fully transformed data**
- Stores **historical data**
- Data is **appended monthly**

Unlike staging tables, presentation tables are **not cleared between loads**.




---

# Orchestration Pipeline

An orchestration pipeline runs the full process.

Execution order:

1. Run **Staging Pipeline**
2. Run **Presentation Pipeline**

This enables **end-to-end processing through a single pipeline execution**.

---

# Orchestration Pipeline Diagram

<img src="Images/Final Orchestration Pipeline.png" width="500">

---

# Automation

The pipeline supports automation.

Capabilities include:

- Detecting the **next data file to process**
- Processing new monthly data
- Tracking processing history
- Preventing duplicate processing

Pipelines can be scheduled to run **monthly when new data arrives**.

---

# Power BI Semantic Model

The presentation table is added to the **default semantic model** in Fabric.

This semantic model is then used to create Power BI reports.

---

# Power BI Report

<img src="Images/Power BI Report.png" width="800">

The Power BI report provides insights into NYC taxi activity.

Report features include:

Filters

- Date range
- Payment method
- Vendor
- Pickup and Dropoff Borough

KPIs

- Total revenue
- Total number of trips
- Total passengers

Visualizations

- Daily revenue by payment method
- Most frequent pickup and dropoff borough combinations

---

# End-to-End Processing

The full workflow is:

1. Upload raw Parquet files to Lakehouse
2. Run orchestration pipeline
3. Load data into staging tables
4. Clean and transform data
5. Append data to presentation table
6. Update metadata logs
7. Power BI report reflects latest data using Direct Lake

Each pipeline run processes **one additional month of data**.

---

# Performance Optimization

The project demonstrates two approaches for transformation:

1. **Dataflow Gen2**
2. **Stored Procedure**

Using a **stored procedure** can significantly improve performance compared to Dataflows due to lower overhead.

---

# Project Outcome

The final solution provides:

- Automated monthly data ingestion
- Data cleansing and transformation
- Historical storage of taxi trip data
- Metadata-driven processing
- Power BI analytics

The architecture demonstrates how **Microsoft Fabric can be used to build a complete analytics platform from ingestion to reporting**.
