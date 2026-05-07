# SQL_dbt_airflow

This repository contains a dbt project built on Snowflake sample TPCH data. It models raw order and line item data into reusable staging, intermediate, and fact-layer tables, with source tests and business-rule validations.

## Project Overview

The project follows a standard dbt transformation pattern:

- `staging` models clean and rename raw source columns
- `intermediate` models join and enrich order data
- `marts` models create analytics-ready tables for reporting

Current pipeline flow:

`tpch.orders` + `tpch.lineitem` -> `stg_tpch_orders` + `stg_tpch_line_items` -> `int_order_items` -> `int_order_items_summary` -> `fct_orders`

## Tech Stack

- dbt Core
- Snowflake
- `dbt_utils` package
- SQL + Jinja macros

## Models

### Staging
- `stg_tpch_orders`
  - Renames and standardizes order columns from `snowflake_sample_data.tpch_sf1.orders`
- `stg_tpch_line_items`
  - Renames line item fields and generates a surrogate key for each order item

### Intermediate
- `int_order_items`
  - Joins orders to line items
  - Calculates `item_discount_amount` using the `discounted_amount` macro
- `int_order_items_summary`
  - Aggregates item-level metrics to the order level

### Mart
- `fct_orders`
  - Final fact table combining order-level attributes with aggregated item sales and discount metrics

## Tests

This project includes both schema tests and singular SQL tests.

### Source and model tests
- `unique`
- `not_null`
- `relationships`
- `accepted_values`

### Custom singular tests
- `fct_orders_date_valid.sql`
  - Flags orders with invalid dates
- `fct_orders_discount.sql`
  - Validates discount-related business logic

## Project Structure

```text
ETl/
├── models/
│   ├── staging/
│   │   ├── stg_tpch_orders.sql
│   │   ├── stg_tpch_line_items.sql
│   │   └── tpch_sources.yml
│   └── marts/
│       ├── int_order_items.sql
│       ├── int_order_items_summary.sql
│       ├── fct_orders.sql
│       └── generic_tests.yml
├── macros/
│   └── pricing.sql
├── tests/
│   ├── fct_orders_date_valid.sql
│   └── fct_orders_discount.sql
├── dbt_project.yml
└── packages.yml

Notes
The project uses Snowflake sample data from snowflake_sample_data.tpch_sf1
dbt model materializations are configured in dbt_project.yml
Generated folders such as target/, dbt_packages/, and logs/ are excluded from Git
