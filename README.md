# dbt Fundamentals — Analytics Engineering Portfolio Project

![dbt](https://img.shields.io/badge/dbt-1.x-orange?logo=dbt)
![Snowflake](https://img.shields.io/badge/Snowflake-Data%20Platform-29B5E8?logo=snowflake)
![SQL](https://img.shields.io/badge/SQL-T--SQL%20%7C%20Snowflake%20SQL-blue)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

## Overview

This project demonstrates core **analytics engineering** competencies using **dbt (data build tool)** connected to a **Snowflake** data warehouse. It was built as part of the dbt Fundamentals certification track and showcases production-relevant patterns including layered data modeling, source configuration, materialization strategy, and data freshness monitoring.

The project transforms raw transactional data from a simulated e-commerce business (Jaffle Shop) and a payments processor (Stripe) into clean, analyst-ready dimensional and fact models.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| **dbt Core** | Data transformation, modeling, testing, documentation |
| **Snowflake** | Cloud data warehouse / execution engine |
| **SQL** | Model logic (Snowflake SQL dialect) |
| **YAML** | Source definitions, model configuration, schema tests |
| **Git / GitHub** | Version control, change tracking |
| **VS Code + dbt Extension** | Local development environment |

---

## Architecture

This project follows the **layered data modeling** pattern — a widely adopted approach in modern analytics engineering:

```
RAW LAYER (Snowflake)
├── raw.jaffle_shop.customers
├── raw.jaffle_shop.orders
└── raw.stripe.payment
        │
        │  {{ source() }}          ← boundary between raw and dbt
        ▼
STAGING LAYER  (models/staging/)
├── stg_jaffle_shop__customers.sql    → view
├── stg_jaffle_shop__orders.sql       → view
└── stg_stripe__payments.sql          → view
        │
        │  {{ ref() }}             ← all downstream references
        ▼
MARTS LAYER  (models/marts/)
├── finance/
│   └── fct_orders.sql            → table
└── marketing/
    └── dim_customers.sql         → table
```

### Layer Responsibilities

**Staging** — One-to-one mapping with source tables. Light transformations only: column renaming, type casting, and basic cleaning. All raw source references are isolated here via the `{{ source() }}` macro, ensuring a single point of change if upstream schemas evolve.

**Marts** — Business-logic layer. Joins, aggregations, and derived metrics live here. Models are organized by business domain (finance, marketing) and consumed by BI tools and analysts. References staging models exclusively via `{{ ref() }}`, enforcing unidirectional data flow.

---

## Project Structure

```
dbt_fundamentals/
├── models/
│   ├── staging/
│   │   ├── jaffle_shop/
│   │   │   ├── _src_jaffle_shop.yml          ← source definitions + freshness
│   │   │   ├── stg_jaffle_shop__customers.sql
│   │   │   └── stg_jaffle_shop__orders.sql
│   │   └── stripe/
│   │       ├── _src_stripe.yml
│   │       └── stg_stripe__payments.sql
│   └── marts/
│       ├── finance/
│       │   └── fct_orders.sql
│       └── marketing/
│           └── dim_customers.sql
├── dbt_project.yml                            ← materialization config
└── README.md
```

---

## Key Models

### `stg_jaffle_shop__customers`
Standardizes the raw customers table. Renames `id` → `customer_id` for consistency across the project.

### `stg_jaffle_shop__orders`
Standardizes the raw orders table. Renames `id` → `order_id`, `user_id` → `customer_id`. Preserves `order_date` and `status` for downstream use.

### `stg_stripe__payments`
Stages raw payment data from Stripe. Feeds payment amount into the finance mart layer.

### `fct_orders` *(Finance Mart)*
Fact table joining orders and payments. Exposes `order_id`, `customer_id`, and `amount` for financial reporting.

### `dim_customers` *(Marketing Mart)*
Customer dimension enriched with behavioral metrics derived from order history:
- `first_order_date`
- `most_recent_order_date`
- `number_of_orders`
- `lifetime_value` *(sum of all payments — total across all customers: $1,672)*

---

## Materialization Strategy

Configured in `dbt_project.yml` to optimize for query performance vs. compute cost:

```yaml
models:
  jaffle_shop:
    staging:
      +materialized: view      # lightweight, always fresh, low storage cost
    marts:
      +materialized: table     # pre-computed, fast for BI tool consumption
```

**Rationale:** Staging models are materialized as **views** because they are thin wrappers over raw data and don't benefit from storage. Mart models are materialized as **tables** because they contain aggregations and joins that are expensive to recompute on every query.

---

## Source Freshness Monitoring

Configured freshness thresholds on the `jaffle_shop.orders` source to detect pipeline delays:

```yaml
freshness:
  warn_after:  {count: 24, period: hour}
  error_after: {count: 48, period: hour}
loaded_at_field: _etl_loaded_at
```

Run with:
```bash
dbt source freshness
```

---

## How to Run

```bash
# Install dependencies
pip install dbt-snowflake

# Validate connection
dbt debug

# Run all models
dbt run

# Run staging models only
dbt run --select staging

# Run a specific model and all upstream dependencies
dbt run --select +dim_customers

# Test all models
dbt test

# Check source freshness
dbt source freshness

# Generate and serve documentation
dbt docs generate
dbt docs serve
```

---

## Skills Demonstrated

- **Data modeling** — staging/marts layered architecture following dbt best practices
- **Snowflake SQL** — CTEs, window functions, aggregations, left joins
- **dbt macros** — `{{ ref() }}` and `{{ source() }}` for dependency management
- **Materialization strategy** — views vs. tables with documented rationale
- **Source configuration** — YAML-based source definitions with freshness monitoring
- **Project organization** — domain-driven folder structure (finance, marketing)
- **Version control** — Git-based change tracking with meaningful commit history

---

## About

Built by **Mario Perez** | San Antonio, TX  
18+ years in enterprise and healthcare analytics  
[GitHub](https://github.com/MarioPerezDE) · [LinkedIn](https://linkedin.com/in/perez-mario)
