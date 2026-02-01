# QueryEvent status variables differ (Q_HRNOW, Q_XID)

Issue ID: A-05

## Summary
MariaDB includes status vars not present in MySQL; these affect temporal and transactional decoding.

## Impact on DM
DDL restore and time-sensitive decoding can be wrong if these variables are ignored.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/event/util.go#L286-L340

## Evidence / Downstream Behavior
- None

## Primary Use Cases
- Low-latency trading
- Timezone-sensitive systems

## Recommended Handling
- Parse MariaDB status vars and apply them to session state.
- Add tests with MariaDB binlog QueryEvent samples.
