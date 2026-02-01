# ALTER/CREATE INDEX IF EXISTS/IF NOT EXISTS (MariaDB extension)

Issue ID: B-06

## Summary
MariaDB supports IF EXISTS/IF NOT EXISTS on index DDL. TiDB AST restore can change semantics.

## Impact on DM
DDL may fail or behave differently during replay.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L145-L195
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L940-L944
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L1931-L1933
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L3352-L3358

## Primary Use Cases
- SaaS rolling upgrades
- Online schema changes

## Recommended Handling
- Preserve IF EXISTS semantics during transform.
- Fallback to raw SQL execution when safe.
