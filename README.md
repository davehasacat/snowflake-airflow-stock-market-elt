# Snowflake/Airflow Stock Market ELT Pipeline

End-to-end, containerized data pipeline for ingesting, storing, transforming, and visualizing U.S. stock and options data using **Apache Airflow**, **AWS**, **Snowflake**, and **dbt**.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Cloud Architecture](#cloud-architecture)
- [Pipeline Overview](#pipeline-overview)
  - [Ingestion](#ingestion)
  - [Loading](#loading)
  - [Transformation](#transformation)
  - [Dashboarding](#dashboarding)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Setup](#setup)
  - [Running Locally](#running-locally)
- [Demonstration](#demonstration)
- [Documentation & References](#documentation--references)

---

## Overview

This project implements a modern **ELT (Extract–Load–Transform)** data pipeline for stock and options data using open-source and cloud-native tools.

The pipeline:

1. Extracts data from the **Polygon.io API**
2. Loads it to **AWS S3**
3. Copies it into **Snowflake**
4. Transforms it with **dbt Core**
5. Visualizes results in a **Plotly Dash dashboard**

All components are orchestrated via **Apache Airflow** (running locally via **Astro CLI**), designed to scale easily to cloud-managed Airflow or MWAA.

---

## Features

- 🔁 **Automated Daily Ingestion** – Polygon stocks and options APIs feed S3 daily.
- 🧩 **Modular DAGs** – Separate DAGs for daily ingest, backfills, loads, and transformations.
- ☁️ **Cloud-Native ELT** – Built on AWS (S3, IAM, Secrets Manager) and Snowflake.
- 📊 **dbt Models + Snapshots** – Source → staging → marts with data tests and documentation.
- 🧱 **Containerized Environment** – Fully reproducible Docker setup via Astro CLI.
- 🧭 **Observability & Idempotency** – Airflow pools, retry logic, S3 manifest tracking.
- 📈 **Visual Analytics** – Plotly Dash dashboard for transformed data in Snowflake.

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-------------|----------|
| **Orchestration** | Apache Airflow (Astro CLI) | DAG scheduling, monitoring |
| **Data Ingestion** | Python (Requests), AWS S3 | Pulls Polygon API → object storage |
| **Data Lake** | Amazon S3 | Raw and manifest storage |
| **Data Warehouse** | Snowflake | Staging + analytical tables |
| **Transformation** | dbt Core | SQL-based ELT and data modeling |
| **Dashboarding** | Plotly Dash | Data visualization |
| **Containerization** | Docker | Local reproducibility |
| **Version Control** | Git | CI/CD and code management |

---

## Cloud Architecture

```
                   ┌────────────────────────────┐
                   │        Polygon API         │
                   └────────────┬───────────────┘
                                │
                                ▼
                   ┌────────────────────────────┐
                   │  Airflow (Astro, Docker)   │
                   │  - Ingest + Load DAGs      │
                   │  - dbt Transform DAG       │
                   └────────────┬───────────────┘
                                │
                         AWS SDK / S3Hook
                                │
                                ▼
                   ┌────────────────────────────┐
                   │        AWS S3 (Raw)        │
                   │  raw/stocks/, raw/options/ │
                   └────────────┬───────────────┘
                                │
                           Snowflake COPY
                                │
                                ▼
                   ┌────────────────────────────┐
                   │        Snowflake DW        │
                   │  source_*, staging_*, marts│
                   └────────────┬───────────────┘
                                │
                              dbt
                                │
                                ▼
                   ┌────────────────────────────┐
                   │       Plotly Dash App       │
                   │     (analytics dashboard)   │
                   └────────────────────────────┘
```

### Cloud Summary

- **AWS S3** serves as the raw data lake layer and artifact store.
- **Airflow (Astro)** manages all ELT orchestration, running in Docker for local dev.
- **Snowflake** is the central warehouse for transformations and analysis.
- **AWS Secrets Manager** securely stores credentials for Airflow connections and variables.
- **dbt Core** runs transformations directly in Snowflake, managed via Airflow DAGs.
- **Plotly Dash** consumes transformed tables for interactive analytics.

---

## Pipeline Overview

### Ingestion

- DAGs call Polygon’s REST APIs for stocks and options data.
- Each record is written to versioned paths under:

  ``` txt
  s3://stock-market-elt/raw/stocks/<TICKER>/<DATE>.json
  s3://stock-market-elt/raw/options/<CONTRACT>/<DATE>.json.gz
  ```

- Daily runs use POINTER manifests; backfills use flat manifests.

### Loading

- Airflow DAGs (`polygon_stocks_load`, `polygon_options_load`) read manifests.
- Files are copied into Snowflake external tables using `COPY INTO` and JSON parsing.

### Transformation

- `dbt_build` runs the full dbt workflow:
  - Renders dynamic profiles from Airflow connections.
  - Applies slim, stateful builds when available.
  - Publishes `manifest.json` and `run_results.json` to S3.

### Dashboarding

- A lightweight Dash app queries Snowflake using credentials managed via Airflow.
- Plots interactive trends and historical metrics for tickers and contracts.

---

## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/)
- [Astro CLI](https://www.astronomer.io/docs/astro/cli/overview)
- AWS CLI configured with access to your account
- A Snowflake account (with warehouse and database)
- Polygon.io API keys for stocks and options

### Setup

1. Clone this repository  

   ```bash
   git clone https://github.com/your-username/stock-market-elt.git
   cd stock-market-elt
   ```

2. Copy the example environment file and fill in credentials  

   ```bash
   cp .env.example .env
   ```

3. Start Airflow locally via Astro  

   ```bash
   astro dev start
   ```

4. Verify your environment  
   - Check Airflow web UI: http://localhost:8080  
   - Trigger `utils_smoke_detector` DAG to confirm connectivity.

### Running Locally

Run dbt manually inside the container:

```bash
astro dev bash --scheduler
dbt debug
dbt build --full-refresh
```

---

## Demonstration

### Airflow Orchestration

The Airflow UI shows individual DAGs for ingestion, loading, and transformations with clear dataset dependencies and retry policies.

### Interactive Dashboard

The Dash app (optional) visualizes metrics from Snowflake marts—volatility, price movement, volume, and other analytics.

---

## Documentation & References

For more detailed information on the tools and technologies used in this project, refer to:

- **[Apache Airflow Documentation](https://airflow.apache.org/docs/)**
- **[Astro CLI Documentation](https://www.astronomer.io/docs/astro/cli/overview)**
- **[dbt Documentation](https://docs.getdbt.com/)**
- **[Docker Documentation](https://docs.docker.com/)**
- **[Plotly Dash Documentation](https://dash.plotly.com/)**
- **[Snowflake Documentation](https://docs.snowflake.com/)**
