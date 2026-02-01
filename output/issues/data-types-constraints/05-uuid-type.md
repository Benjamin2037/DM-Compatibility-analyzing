# UUID type and name handling

Issue ID: C-05

## Summary
MariaDB supports a UUID type/name pattern; TiDB does not.

## Impact on DM
DDL fails or indexes conflict if UUID type is not rewritten.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327

## Evidence / Downstream Behavior
- https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/parser/loader.go#L89-L205

## Primary Use Cases
- Distributed IDs
- Cross-system integration

## Recommended Handling
- Rewrite UUID to CHAR(36) or BINARY(16) with proper defaults.
- Update index definitions to match new type length.
