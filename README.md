# BI & Data Project

## Table of Contents

1. The architecture/design of the solution (high-level overview)
2. Key decisions you made and why
3. Code samples or data models (as long as you’re able to share them)
4. Any challenges you ran into and how you solved them

## Data Architecture

- [dbt](https://www.paradime.io/) for data transformations
- [Snowflake](https://www.snowflake.com/en/) for data storage and computing
- [Hex](https://hex.tech/) for data analysis and ad-hoc visualisations
- [Sigma](https://www.sigmacomputing.com/) for dashboard creation
- [Airflow](https://airflow.apache.org/) for orchestration

<img width="968" height="408" alt="image" src="https://github.com/user-attachments/assets/f9fd3569-5539-4540-abe4-07976615ede1" />

## DBT project architecture

### Layers

The project consists of 3 layers:

**1. Sources:** These tables are the starting point. Small adjustments are made, such as changing the names of columns, but not much else.

**2. Transform:** In this step, the data from the "Sources" goes through more complex changes. It includes combining tables, performing calculations, and organizing the data. It is divided into two parts:
- Raptors: This section focuses on data specifically about the Toronto Raptors team.
- Teams: This section deals with data about all teams.

**3. Marts:** These tables are the finished product, ready for people to use. They come in two sub-folders:
- Business: Aimed at professionals who need to analyze data or make visualizations and dashboards, like business stakeholders or data analysts.
- Fans: Designed for NBA fans who want to explore and analyze the data for fun.
