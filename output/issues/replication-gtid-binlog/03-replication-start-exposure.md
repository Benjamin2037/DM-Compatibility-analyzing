# Replication start exposure differs (SHOW MASTER STATUS vs gtid_binlog_pos)

Issue ID: A-03

## Summary
MariaDB exposes GTID state via gtid_binlog_pos and related variables, not identical to MySQL SHOW MASTER STATUS output.

## Impact on DM
Start-point calculation can miss MariaDB GTID state and begin from an incorrect position.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/utils.go#L98-L236

## Evidence / Downstream Behavior
- None

## Primary Use Cases
- Cutover for bulk migration
- Gray/blue-green sync

## Recommended Handling
- Use flavor-specific queries for GTID and binlog position.
- Validate the initial checkpoint before starting relay.
