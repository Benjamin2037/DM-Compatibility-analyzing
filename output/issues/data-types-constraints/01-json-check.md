# JSON semantics and JSON_VALID CHECK constraints

Issue ID: C-01

## Summary
MariaDB commonly uses JSON_VALID and CHECK constraints; TiDB restricts supported CHECK functions.

## Impact on DM
DDL fails or semantics drift when constraints are not supported.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/table/constraint.go#L83-L127

## Primary Use Cases
- E-commerce attributes
- Web3 metadata
- User profiles

## Recommended Handling
- Strip JSON_VALID CHECK constraints or rewrite to supported predicates.
- Record all removed constraints in audit logs.
