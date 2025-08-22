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

## 3. Ingestion

Fivetran was used to automatically pull data from different source into Snowflake. We chose Fivetran because it allowed us to set up pipelines in hours instead of weeks, reduced maintenance work, and ensured reliable, up-to-date data without the team having to manage custom ETL code.

## 4. DBT project

Each layer has a specific purpose, which makes the pipeline easier to maintain and understand.  

### **1. Raw**  
- Contains all raw data as it lands from Fivetran into Snowflake.  
- No changes are made here—it’s a direct copy of the source systems.  
- Example schema: `raw.salesforce_accounts`, `raw.hubspot_contacts`.  

### **2. Staging**  
- Cleans and standardizes raw tables.  
- Tasks include: renaming columns, fixing data types, handling nulls, and selecting only needed fields.  
- Each source table has a matching staging model, usually with a `stg_` prefix.  
- Example: `stg_salesforce__accounts`, `stg_hubspot__contacts`.  

### **3. Intermediate**  
- Combines data from staging models to create more useful datasets.  
- This layer is where joins, calculations, and logic across systems happen (e.g., combining CRM + product usage data).  
- Example: `int__customer_activity`, `int__marketing_performance`.  

### **4. Marts**  
- Final layer with business-friendly tables, designed for reporting and dashboards.  
- Organized into subfolders for different audiences:  
  - **Business:** Metrics for Sales, Marketing, Finance, etc.  
  - **Product/Other:** User engagement, funnel analysis, or other domain-specific tables.  
- Example: `mart_sales__pipeline_summary`, `mart_marketing__channel_performance`.  

## Folder Structure in dbt  

```plaintext
models/
  raw/               # (optional if you define sources in yml)
  staging/
    salesforce/
      stg_salesforce__accounts.sql
      stg_salesforce__opportunities.sql
    hubspot/
      stg_hubspot__contacts.sql
  intermediate/
    int__customer_activity.sql
    int__marketing_performance.sql
  marts/
    sales/
      mart_sales__pipeline_summary.sql
    marketing/
      mart_marketing__channel_performance.sql
    product/
      mart_product__usage_summary.sql
```


### Testing & Data Validation

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


### Documentation

Every model has detailed descriptions and column definitions.


## 5. Orchestration & Scheduling

* **Daily Production Job:** Daily full refresh of the whole dbt project


## 6. Environments and CI/CD

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

