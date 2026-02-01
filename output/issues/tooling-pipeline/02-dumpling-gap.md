# Dumpling only fills collation, not MariaDB syntax differences

Issue ID: F-02

## Summary
Dumpling passes MariaDB DDL mostly as-is; unsupported syntax reaches TiDB.

## Impact on DM
Full load fails on JSON/UUID/system-versioned and other DDL differences.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/dumpling/dumpling.go#L322-L356

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/sql.go#L87-L101
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/dump.go#L430-L478
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/dump.go#L536-L587

## Primary Use Cases
- Large-scale migration
- Heterogeneous schemas

## Recommended Handling
- Integrate transformer rules in the dump pipeline.
- Log every transformed statement for audit.
