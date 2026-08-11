# Jaffle Shop — dbt Portfolio Project

## What this is

A dbt project that transforms raw jaffle_shop order and customer data in Snowflake into clean, tested, analytics-ready models. It was built to practice the core dbt workflow: staging raw sources, layering business logic on top, and enforcing data quality with automated tests.

## Structure

Models are organized in two layers: `staging` (`stg_customers`, `stg_orders`) does light 1:1 cleaning and renaming directly on top of the raw source tables, and `marts` (`dim_customers`) builds on the staging models to apply business logic — joining orders to customers and aggregating order history into one row per customer. Keeping these layers separate means the cleaning logic lives in exactly one place, and downstream models never touch raw sources directly.

## Data quality

- **`unique` + `not_null` on `stg_customers.customer_id` and `stg_orders.order_id`** — the minimum bar for trusting a primary key; without these, joins and aggregations downstream can silently duplicate or drop rows.
- **`relationships` on `stg_orders.customer_id` → `stg_customers.customer_id`** — referential integrity enforced in the pipeline. It catches orphaned orders that point at a customer who doesn't exist, the same instinct as a foreign key constraint at the database layer, just enforced at the transformation layer instead.
- **`accepted_values` on `stg_orders.status`** — catches upstream systems introducing a new status code without telling anyone, before it silently breaks downstream reporting.

All of the tests above pass. The repo also still contains the default `models/example/` scaffold from `dbt init`, which includes an intentionally failing `not_null` test as a tutorial demo — that failure is unrelated to the jaffle_shop models above and was left in place rather than deleted, to keep the scaffold's teaching example intact.

## Lineage

![Lineage graph](docs/lineage.png)

---

Built while completing dbt Fundamentals (dbt Labs, 2026) — credential: credentials.getdbt.com/acd20f1b-b60a-479b-aa3a-4e5964e603b9
