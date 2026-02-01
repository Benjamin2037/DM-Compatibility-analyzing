# MariaDB storage engines (ARIA, MYROCKS, TOKUDB)

Issue ID: B-05

## Summary
TiDB does not implement MariaDB storage engines; it only validates engine names.

## Impact on DM
Engine semantics are lost, possibly affecting performance or durability expectations.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/planner/core/preprocess.go#L1476-L1516

## Primary Use Cases
- High-write logs
- Archival storage

## Recommended Handling
- Map to InnoDB or strip engine clauses.
- Document semantic differences for downstream owners.
