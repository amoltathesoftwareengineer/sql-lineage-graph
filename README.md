# sql-lineage-graph

Parse SQL queries and visualize **data lineage** as a graph: see which tables and columns feed into which, and trace how data flows from source to final output.

> ⚠️ Status: early development. Edit the sections below to match what the project actually does.

## Why?

In real data pipelines, a single dashboard number can depend on dozens of tables and views. When something breaks, or a column needs to change, you need to answer:

- Where does this column come from?
- What downstream tables and reports will break if I change this?

`sql-lineage-graph` answers these by reading your SQL and building a dependency graph.

## Features

- Parse SQL (`SELECT`, `INSERT INTO ... SELECT`, `CREATE TABLE AS`, `CREATE VIEW`, CTEs, joins, subqueries)
- Table-level lineage (source tables → target tables)
- Column-level lineage (source columns → output columns)
- Graph output as JSON, DOT (Graphviz), or an interactive visualization
- Support for multiple SQL dialects (e.g. PostgreSQL, MySQL, BigQuery, Snowflake)

## Installation

```bash
git clone https://github.com/amoltathesoftwareengineer/sql-lineage-graph.git
cd sql-lineage-graph
pip install -r requirements.txt
```

## Quick start

Given this SQL:

```sql
CREATE TABLE sales_summary AS
WITH recent AS (
    SELECT order_id, customer_id, amount
    FROM orders
    WHERE order_date >= '2026-01-01'
)
SELECT c.region, SUM(r.amount) AS total_sales
FROM recent r
JOIN customers c ON r.customer_id = c.id
GROUP BY c.region;
```

Run:

```bash
python -m sql_lineage_graph path/to/query.sql --format dot
```

Expected lineage:

```
orders.amount      ──► sales_summary.total_sales
customers.region   ──► sales_summary.region
```

Graph view:

```
orders ──┐
         ├──► recent ──┐
customers ─────────────┴──► sales_summary
```

## Usage

```bash
# Single file
python -m sql_lineage_graph query.sql

# Whole folder of .sql files
python -m sql_lineage_graph ./sql/ --recursive

# Choose output format: json | dot | html
python -m sql_lineage_graph query.sql --format json --output lineage.json

# Pick the SQL dialect
python -m sql_lineage_graph query.sql --dialect bigquery
```

### As a library

```python
from sql_lineage_graph import build_lineage

graph = build_lineage(sql_text, dialect="postgres")

for edge in graph.edges:
    print(edge.source, "->", edge.target)
```

## Project structure

```
sql-lineage-graph/
├── sql_lineage_graph/   # core package (parser, graph builder, exporters)
├── tests/               # unit tests
├── examples/            # sample SQL files
├── requirements.txt
└── README.md
```

## Roadmap

- [ ] Table-level lineage
- [ ] Column-level lineage
- [ ] Interactive HTML visualization
- [ ] dbt project support
- [ ] CLI packaging on PyPI

## Contributing

Contributions are welcome.

1. Fork the repo
2. Create a branch: `git checkout -b feature/my-feature`
3. Commit your changes
4. Open a pull request

## License

MIT License. See [LICENSE](https://github.com/amoltathesoftwareengineer/sql-lineage-graph/blob/main/LICENSE).

## Author

Amol Tathe · [LinkedIn](https://linkedin.com/in/amoltathesoftwareengineer) · [GitHub](https://github.com/amoltathesoftwareengineer)
