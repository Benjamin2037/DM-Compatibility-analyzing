# Sequence DDL (CREATE/ALTER/DROP SEQUENCE)

Issue ID: B-02

## Summary
MariaDB sequence statements may be parsed but schema tracker does not fully support them.

## Impact on DM
Tracker can panic or drift, causing DML mapping errors.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L185-L220
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/schema/tracker.go#L184-L220

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L1819-L1849

## Primary Use Cases
- Order IDs
- Finance billing

## Recommended Handling
- Add tracker support or transform to TiDB sequence semantics.
- Block if transformation is not guaranteed safe.
