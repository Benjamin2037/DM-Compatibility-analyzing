# MariaDB collation set differs (including NO PAD)

Issue ID: E-01

## Summary
MariaDB provides collations not supported by TiDB; NO PAD behaviors differ.

## Impact on DM
DDL can fail or produce different ordering/uniqueness behavior.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1392-L1459

## Evidence / Downstream Behavior
- https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/collation.go#L11-L209

## Primary Use Cases
- Multilingual e-commerce
- Social platforms

## Recommended Handling
- Map unsupported collations to the closest TiDB equivalents.
- Generate a mapping report for review.
