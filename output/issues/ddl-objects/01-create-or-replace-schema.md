# CREATE OR REPLACE SCHEMA (MariaDB)

Issue ID: B-01

## Summary
MariaDB supports CREATE OR REPLACE SCHEMA; TiDB does not.

## Impact on DM
DDL fails or tracker state diverges if not rewritten.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/schema/tracker.go#L184-L220

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/executor.go#L289-L305
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/schematracker/dm_tracker.go#L101-L109

## Primary Use Cases
- Multi-tenant provisioning
- Automated ops

## Recommended Handling
- Transform into DROP + CREATE only if safe and configured.
- Default to strict failure to avoid data loss.
