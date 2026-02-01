# CHECK constraint function limits

Issue ID: C-03

## Summary
TiDB restricts functions allowed in CHECK constraints; MariaDB allows many common functions.

## Impact on DM
DDL fails or constraints are dropped, changing data quality rules.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/table/constraint.go#L83-L127

## Primary Use Cases
- Risk control
- Data quality rules

## Recommended Handling
- Rewrite to supported functions or drop with warning.
- Validate downstream data quality after migration.
