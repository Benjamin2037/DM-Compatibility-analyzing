# Stricter index length and prefix limits

Issue ID: D-02

## Summary
TiDB enforces shorter index length limits, especially for multibyte charsets.

## Impact on DM
Long indexes from MariaDB will fail without truncation.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327

## Evidence / Downstream Behavior
- https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/keylength.go#L8-L40
- https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/index_prefix.go#L9-L74

## Primary Use Cases
- Cross-border e-commerce
- Multilingual search

## Recommended Handling
- Apply keylength and index_prefix rules during transform.
- Log all truncated index definitions for review.
