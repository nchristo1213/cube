## fix(schema-compiler): Preserve Full Time Buckets in Period-over-Period Queries

This PR replaces `INNER JOIN` with `LEFT JOIN` when stitching sub-queries in `outerMeasuresJoinFullKeyQueryAggregate`, ensuring all expected time buckets appear when calculating period-over-period (PoP) metrics.
### Problem

When PoP queries used `INNER JOIN`, buckets missing from auxiliary sub-queries (`q_1`, `q_2`, ...) were dropped—causing incomplete results or incorrect ratios.
### Solution

Use `LEFT JOIN` instead, preserving all keys from the anchor query (`q_0`). Missing values in auxiliary queries yield `NULL` rather than removing the row entirely.

```js
LEFT JOIN (${subQuery}) AS q_${i+1}   ON ${this.dimensionsJoinCondition(`q_${i}`, `q_${i+1}`)}
```
### Benefits

- Retains full calendar continuity for PoP metrics
- Allows ratio calculations to gracefully degrade when prior period data is missing
- Ensures accuracy without introducing gaps or dropped time buckets
### Tests

- **New fixture**: `sales` cube with `date`, `category`, `region`, and PoP measures
- **Adjusted**:
    - `CAGR`: Added `NULL` rows
    - `multi-stage graph` tests: Added secondary ordering
- **New tests**:
    - Time-only PoP
    - One-dimension PoP
    - Two-dimension PoP
        
These tests ensure the correct number of periods appear and that PoP measures return `NULL` instead of dropping rows when previous-period values are unavailable.
