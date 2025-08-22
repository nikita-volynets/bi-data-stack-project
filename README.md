# BI & Data Project

## Table of Contents

1. The architecture/design of the solution (high-level overview)
2. Key decisions you made and why
3. Code samples or data models (as long as you’re able to share them)
4. Any challenges you ran into and how you solved them

## Architecture Diagram

<img width="968" height="408" alt="image" src="https://github.com/user-attachments/assets/f9fd3569-5539-4540-abe4-07976615ede1" />

## 1. Tools

- [dbt](https://www.paradime.io/) for data transformations
- [Snowflake](https://www.snowflake.com/en/) for data storage and computing
- [Hex](https://hex.tech/) for data analysis and ad-hoc visualisations
- [Sigma](https://www.sigmacomputing.com/) for dashboard creation
- [Airflow](https://airflow.apache.org/) for orchestration

## DBT project architecture

## 2. Data Ingestion

The data was loaded to Google Cloud Storage Bucket and manually uploaded to `raw_jaffle_shop` dataset.


## 3. dbt Models

<img width="979" height="667" alt="image" src="https://github.com/user-attachments/assets/25a66767-83c4-4a65-978f-3bd0a94ebe19" />

This project implements a **medallion architecture** (Bronze → Silver → Gold) with the following key layers:

### Layers

The project consists of 3 layers:

**1. Sources:** These tables are the starting point. Small adjustments are made, such as changing the names of columns, but not much else.

**2. Transform:** In this step, the data from the "Sources" goes through more complex changes. It includes combining tables, performing calculations, and organizing the data. It is divided into two parts:
- Raptors: This section focuses on data specifically about the Toronto Raptors team.
- Teams: This section deals with data about all teams.

**3. Marts:** These tables are the finished product, ready for people to use. They come in two sub-folders:
- Business: Aimed at professionals who need to analyze data or make visualizations and dashboards, like business stakeholders or data analysts.
- Fans: Designed for NBA fans who want to explore and analyze the data for fun.

**Staging Layer (Bronze - Views):**
* `stg_customers`, `stg_orders`, `stg_order_items`, `stg_products`, `stg_supplies`, `stg_locations`
* Clean and standardize raw data from `raw_jaffle_shop` schema
* Convert cents to dollars, rename columns, add boolean flags

**Intermediate Layer (Silver - Views):**
* `int_customers_enriched` - adds first order date and customer status
* `int_order_items_enriched` - joins orders, products, and supply costs
* `int_customer_order_summary` - calculates lifetime metrics per customer
* `int_order_aggregates`, `int_product_performance` - business logic and aggregations

**Marts Layer (Gold - Tables):**
* `customers` - complete customer profiles with lifetime value
* `orders` - order fact table with all details
* `order_items` - line-item analysis with product and cost data
* `products`, `locations`, `supplies` - dimension tables


## 4. Testing & Data Validation

Comprehensive data quality testing implemented across all model layers:

**Built-in dbt Tests:**
* `unique` and `not_null` tests on primary keys (customer_id, order_id, etc.)
* `accepted_values` tests for categorical fields (customer_type: 'new' or 'returning')
* Data integrity checks across all staging and mart models

**Custom Business Logic Tests:**
* `dbt_utils.expression_is_true` for complex validations
* Row count and referential integrity tests between layers

**Source Data Validation:**
* Freshness monitoring on raw tables with `loaded_at_field` configured
* Source table schema tests defined in `__sources.yml`


## 5. Documentation

Every model has detailed descriptions and column definitions.


## 6. Orchestration & Scheduling

* **Daily Production Job:** Daily full refresh of the whole dbt project


## 7. Environments and CI/CD

We use a multi-environment setup to ensure safe and collaborative development:

- **Raw Data Source**  
  All raw data is stored in the **`raw_jaffle_shop`** dataset.

- **Development Environments**  
  Each developer has a personal BigQuery dataset (e.g., `dbt_nvolynets`) where they can:
  - Test changes  
  - Run models  
  - Validate results  
  These environments are connected to the developer's dbt Cloud development workspace.

- **Pull Request (PR) Workflow**  
  When a developer is ready to propose changes to production:
  - They must open a **Pull Request** in GitHub.
  - A **PR Template** is provided and must be filled in with context, purpose, and testing evidence.

- **CI/CD Automation**
  - **CI Job**: Runs automatically when a PR is created. It builds only the modified models to validate changes.
  - **CD Job**: Runs after a PR is merged to `main`, deploying updated models to the **production** environment.

This workflow ensures code quality, enables team collaboration, and maintains a reliable production environment.

