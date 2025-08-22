# BI & Data Project

## Table of Contents

1. The architecture/design of the solution (high-level overview)
2. Key decisions you made and why
3. Code samples or data models (as long as you’re able to share them)
4. Any challenges you ran into and how you solved them

## 1. Architecture Diagram

<img width="968" height="408" alt="image" src="https://github.com/user-attachments/assets/f9fd3569-5539-4540-abe4-07976615ede1" />

## 2. Tools

- **dbt** for data transformations
- **Snowflake** for data storage and computing
- **Hex** for data analysis and ad-hoc visualisations
- **Sigma** for dashboard creation
- **Airflow** for orchestration

## 2. Ingestion

## 3. DBT project

### dbt Models

The project consists of several layers:

**0. Raw:** These tables are the starting point. Small adjustments are made, such as changing the names of columns, but not much else.

**1. Sources:** These tables are the starting point. Small adjustments are made, such as changing the names of columns, but not much else.

**2. Transform:** In this step, the data from the "Sources" goes through more complex changes. It includes combining tables, performing calculations, and organizing the data. It is divided into two parts:
- Raptors: This section focuses on data specifically about the Toronto Raptors team.
- Teams: This section deals with data about all teams.

**3. Marts:** These tables are the finished product, ready for people to use. They come in two sub-folders:
- Business: Aimed at professionals who need to analyze data or make visualizations and dashboards, like business stakeholders or data analysts.
- Fans: Designed for NBA fans who want to explore and analyze the data for fun.

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

