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

### Folder Structure 

```plaintext
models/
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


To ensure data quality and trust in the models, several types of tests were implemented in dbt:  

### **1. Freshness Tests**  
- Applied on key source tables to confirm data was updated on time (e.g., CRM, marketing, and product activity tables).  
- Example: verifying that Salesforce opportunities or HubSpot contacts were refreshed daily.  

### **2. Built-in dbt Tests**  
Standard schema tests were used to validate important columns:  
- **`unique`** – ensured IDs (like `user_id`, `account_id`) were not duplicated.  
- **`not_null`** – validated required fields (e.g., `email`, `created_at`).  
- **`accepted_values`** – confirmed fields like status or stage only contained expected values.  
- **`relationships`** – checked that foreign keys matched across related tables (e.g., `user_id` in events existed in the users table).  

### **3. Custom Tests**  
Custom SQL tests were added for business-specific rules:  
- **Email validation:** confirmed emails followed proper formatting.  
- **Date checks:** verified `created_at` was always earlier than `closed_at` for opportunities.  
- **Pipeline logic:** flagged rows where opportunity stage was inconsistent with probability.  

### **4. Package Tests**  
Community packages extended testing capabilities:  
- **`dbt_utils`** – used for advanced tests like `equal_rowcount` and `fewer_rows_than`.  
- **`dbt_expectations`** – applied for granular checks, such as value distributions and percentage of nulls in a column. 

## 5. Orchestration & Scheduling

### **Airflow**  
Airflow was used to orchestrate the overall pipeline. It defined the order of tasks, scheduled workflows, and monitored execution.  
- **Scheduling:** Controlled when ingestion, transformations, and checks should run.  
- **Dependencies:** Ensured dbt jobs only started after data ingestion finished.  
- **Monitoring & Alerts:** Provided visibility into workflow health and sent notifications on failures.  
- **Retries:** Automatically retried failed tasks to improve reliability.  

### **dbt Cloud**  
dbt Cloud was used to run transformations.  
- **Job scheduling:** dbt models were grouped into jobs (e.g., staging, marts) and executed inside dbt Cloud.  
- **Testing & Documentation:** dbt jobs also ran schema tests and refreshed documentation as part of scheduled runs.  
- **Integration with Airflow:** Airflow triggered dbt Cloud jobs via API, making sure transformations happened after ingestion and before dashboards refreshed.  

### **Workflow Example**  
1. **Fivetran sync** ingested new data into Snowflake.  
2. **Airflow** detected completion and triggered the **dbt Cloud job**.  
3. **dbt Cloud** ran transformations and data tests.  
4. After dbt jobs succeeded, **Airflow** triggered downstream tasks, such as dashboard refreshes in Sigma.  

### **Why This Setup Was Chosen**  
- dbt Cloud provided a managed, reliable way to run transformations with minimal setup.  
- Airflow gave flexibility to orchestrate across multiple tools (Fivetran, dbt Cloud, Sigma) in one place.  
- This combination kept transformations scalable while ensuring the whole pipeline stayed reliable and transparent.  


## 6. Environments and CI/CD

### **Environments**  
- **Development (Dev):** Used for building and testing new models. Changes were run against a development schema/database to avoid breaking production.  
- **Production (Prod):** Hosted the stable models consumed by stakeholders and dashboards. Only tested and reviewed changes were deployed here.  
- **Separation of Environments:** This setup ensured that experimental work never impacted business-critical reports.  

### **CI/CD with dbt Cloud & GitHub**  
- **Version Control:** All dbt code was stored in GitHub, with changes proposed through pull requests.  
- **Automated Testing:** Each pull request triggered dbt Cloud jobs (or GitHub Actions) to run tests and validate models before merging.  
- **Branching Strategy:** Feature branches were used for development, then merged into `main` only after tests passed.  
- **Deployment:** Once merged, dbt Cloud automatically deployed changes to the production environment.  

### **Why This Setup Was Chosen**  
- **Reliability:** Prevented untested code from reaching production.  
- **Collaboration:** Made it easy for multiple contributors to work on the project simultaneously.  
- **Transparency:** Every change was reviewed and tested, leaving a clear history of updates.  
- **Scalability:** Provided a foundation for adding more complex workflows as the team and data platform grew. 

