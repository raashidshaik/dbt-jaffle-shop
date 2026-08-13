# Jaffle Shop — dbt Portfolio Project

## What this is

A dbt project that transforms raw jaffle_shop order and customer data in Snowflake into clean, tested, analytics-ready models. It was built to practice the core dbt workflow: staging raw sources, layering business logic on top, and enforcing data quality with automated tests.

## Structure

Models are organized in three layers: `sources` (declared in `models/staging/_sources.yml`) point at the raw `jaffle_shop.customers` and `jaffle_shop.orders` tables in Snowflake; `staging` (`stg_customers`, `stg_orders`) selects from those sources via `source()` and does light 1:1 cleaning and renaming; and `marts` (`dim_customers`) builds on the staging models to apply business logic — joining orders to customers and aggregating order history into one row per customer. Keeping these layers separate means the cleaning logic lives in exactly one place, raw tables are only ever referenced through `source()`, and downstream models never touch raw sources directly.

## Data quality

- **`unique` + `not_null` on `stg_customers.customer_id` and `stg_orders.order_id`** — the minimum bar for trusting a primary key; without these, joins and aggregations downstream can silently duplicate or drop rows.
- **`relationships` on `stg_orders.customer_id` → `stg_customers.customer_id`** — referential integrity enforced in the pipeline. It catches orphaned orders that point at a customer who doesn't exist, the same instinct as a foreign key constraint at the database layer, just enforced at the transformation layer instead.
- **`accepted_values` on `stg_orders.status`** — catches upstream systems introducing a new status code without telling anyone, before it silently breaks downstream reporting.
- **`not_null` on `dim_customers.number_of_orders`** — a customer with no orders arrives as `NULL` from the left join, not `0`. Rather than drop the test, the model coalesces the count to `0`, so the column means what a consumer would assume it means.

All of the tests above pass on `dbt build`.

## Running this project

Built in dbt Studio against Snowflake, on the **dbt Fusion** engine.

One thing to know before running it elsewhere: the generic tests in `models/staging/_stg_jaffle_shop.yml` pass their parameters under a nested `arguments:` key. That is the form Fusion expects — on older dbt Core versions the same file will fail at parse time, before any model runs.

1. Point a dbt project at this repo with Snowflake as the warehouse.
2. Load the raw `customers` and `orders` tables into a `raw.jaffle_shop` schema — that is where `models/staging/_sources.yml` expects to find them.
3. `dbt build` — runs the models and their tests together, in dependency order.
4. `dbt docs generate` to rebuild the documentation and the lineage graph below.

## Lineage

![Lineage graph](docs/lineage.png)

---

Built while completing dbt Fundamentals (dbt Labs, 2026) — credential: credentials.getdbt.com/acd20f1b-b60a-479b-aa3a-4e5964e603b9
