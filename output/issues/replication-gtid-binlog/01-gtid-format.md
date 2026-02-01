# GTID format and semantics differ (MySQL vs MariaDB)

Issue ID: A-01

## Summary
MySQL uses uuid:interval, MariaDB uses domain_id-server_id-seq_no. The GTID set semantics and parsing rules are different.

## Impact on DM
Resume position, GTID advancement, and filtering depend on correct parsing. A mismatch can cause drift, duplication, or skipped events.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/gtid/gtid.go#L33-L122
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/db.go#L54-L64

## Evidence / Downstream Behavior
- None

## Primary Use Cases
- Finance HA
- E-commerce DR
- SaaS multi-tenant replication

## Recommended Handling
- Detect source flavor and parse GTID accordingly.
- Normalize GTID sets and enforce validation before resume.
- Add regression tests for mixed MySQL/MariaDB GTID inputs.
