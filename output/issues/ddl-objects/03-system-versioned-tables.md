# System-versioned tables (WITH SYSTEM VERSIONING)

Issue ID: B-03

## Summary
MariaDB supports system-versioned tables; TiDB does not parse or execute the same syntax.

## Impact on DM
DDL fails or is ignored, breaking historical table semantics.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L19-L45
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/parser.y#L4840-L4859

## Primary Use Cases
- Audit/compliance history
- Government archiving

## Recommended Handling
- Block by default or rewrite to a history-table pattern.
- Record explicit warnings for audit purposes.
