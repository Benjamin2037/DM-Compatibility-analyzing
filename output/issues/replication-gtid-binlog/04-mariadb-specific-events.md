# MariaDB-specific binlog events (ANNOTATE_ROWS_EVENT, BINLOG_CHECKPOINT_EVENT)

Issue ID: A-04

## Summary
MariaDB emits additional events not present in MySQL. DM sets MariaDB flags but does not explicitly handle these events.

## Impact on DM
Events may be ignored or inconsistently processed, affecting audit or CDC correctness.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/relay/relay.go#L55-L62
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/relay/relay.go#L1174-L1180

## Evidence / Downstream Behavior
- https://github.com/go-mysql-org/go-mysql/blob/v1.13.0/replication/const.go#L104-L107
- https://github.com/go-mysql-org/go-mysql/blob/v1.13.0/replication/event.go#L830-L853
- https://github.com/MariaDB/server/blob/3218602d3100db9ce7a875511a591cddc173cc16/sql/log_event.h#L701-L723

## Primary Use Cases
- Audit/compliance replay
- CDC

## Recommended Handling
- Parse and record MariaDB-specific events with explicit policy.
- Expose metrics for skipped/ignored event types.
