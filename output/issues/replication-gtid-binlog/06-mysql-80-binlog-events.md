# MySQL 8.0 binlog events not emitted by MariaDB

Issue ID: A-06

## Summary
MySQL 8.0 adds ROWS_QUERY, PARTIAL_UPDATE, TRANSACTION_PAYLOAD events; MariaDB does not emit them.

## Impact on DM
Cross-version replication may mis-handle or ignore newer events when parsing.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L32-L110

## Evidence / Downstream Behavior
- https://github.com/mysql/mysql-server/blob/666701570c392a6052341b6ddb9c21869bb1d733/libbinlogevents/include/binlog_event.h#L333-L358

## Primary Use Cases
- Legacy enterprise cross-version sync
- Heterogeneous upgrades

## Recommended Handling
- Detect event type by flavor and ignore/handle safely.
- Add compatibility tests for MySQL 8.0 events.
