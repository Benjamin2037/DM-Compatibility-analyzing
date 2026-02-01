# Generated column dependency and JSON generated columns

Issue ID: C-04

## Summary
TiDB enforces stricter rules for generated columns and JSON expressions.

## Impact on DM
DDL fails or generated columns are miscomputed.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/generated_column.go#L32-L76

## Primary Use Cases
- BI/reporting
- Derived metrics

## Recommended Handling
- Reorder dependencies and rewrite expressions as needed.
- Add tests for generated column evaluation.
