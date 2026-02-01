# TEXT/BLOB/JSON default value restrictions

Issue ID: C-02

## Summary
TiDB disallows default values for TEXT/BLOB/JSON; MariaDB allows them.

## Impact on DM
DDL fails if defaults are preserved.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/add_column.go#L1208-L1238

## Primary Use Cases
- CMS/content platforms
- Logging systems

## Recommended Handling
- Remove unsupported defaults and document the change.
- Provide application-side or trigger-based defaults.
