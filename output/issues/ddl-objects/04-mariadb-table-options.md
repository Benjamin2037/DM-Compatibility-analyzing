# MariaDB-specific table options (PAGE_CHECKSUM, PAGE_COMPRESSED, etc.)

Issue ID: B-04

## Summary
MariaDB includes table options not supported by TiDB. These options are ignored or rejected.

## Impact on DM
DDL may fail or silently lose storage semantics.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/parser.y#L13149-L13186
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L3033-L3063

## Primary Use Cases
- Large table storage optimization
- Hot/cold tiering

## Recommended Handling
- Strip unsupported table options with warnings.
- Provide a rule report for every removed option.
