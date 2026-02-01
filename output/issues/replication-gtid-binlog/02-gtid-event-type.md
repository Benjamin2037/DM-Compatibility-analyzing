# GTID event type differs (PREVIOUS_GTIDS_EVENT vs MARIADB_GTID_LIST_EVENT)

Issue ID: A-02

## Summary
MariaDB encodes GTID history with MARIADB_GTID_LIST_EVENT rather than MySQL PREVIOUS_GTIDS_EVENT.

## Impact on DM
Incorrect interpretation of previous GTID list can lead to wrong start position and replay errors.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L32-L110
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L142-L176

## Evidence / Downstream Behavior
- None

## Primary Use Cases
- Long-running incremental sync
- Point-in-time resume
- Switchover

## Recommended Handling
- Handle MARIADB_GTID_LIST_EVENT explicitly.
- Persist a normalized previous-gtid view for resume logic.
