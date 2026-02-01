# JSON cannot be indexed; BLOB/TEXT must have prefixes

Issue ID: D-01

## Summary
TiDB disallows JSON indexing and requires prefix indexes for BLOB/TEXT.

## Impact on DM
MariaDB indexes fail or need rewriting.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/index.go#L244-L267

## Primary Use Cases
- Search/tag/log indexes

## Recommended Handling
- Add prefix lengths or drop incompatible indexes.
- Introduce application-level search alternatives if needed.
