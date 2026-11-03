# DataForm
- workplace 
  - definition
    - schema 
      - backfill
        - batch loading 30 days interval 
      - incremental
      - functions 
    - sources
      - declare tables can be refed 
    - views
  - Includes 
    - helper function 
  - workflow_settings.yaml
    - define default project
    - dataset
    - location
    - and vars
      - PII
        - policy tags

## .sqlx : transform data into bigquery
- **only need select function** 
  - dataform translate select into create insert
- SQL + JS
  - components 
    - config 
      - type
        - operation 
          - customize 
            - create multiple table
            - run delete 
        - incremental (not suitable for small table)
          - uniquekey 
          - cluster: for performance
        - table 
  - dynamic create sql
    - use js 
    - [].map


### how to use ref and declare
```js

// definitions/sources/silver_tables.js

const silverTables = [
    "inventory",
    "orders", 
    "customers",
    "products",
    "fulfillment",
    "shipments"
];

silverTables.forEach((name) =>
    declare({
        schema: "silver",
        name,
    })
);
-- Now ALL of these work in any .sqlx file!

SELECT * FROM ${ref("inventory")}
SELECT * FROM ${ref("orders")}
SELECT * FROM ${ref("customers")}
SELECT * FROM ${ref("products")}
SELECT * FROM ${ref("fulfillment")}
SELECT * FROM ${ref("shipments")}
```