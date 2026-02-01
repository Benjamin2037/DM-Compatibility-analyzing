# No schema transformer in the full chain

Issue ID: F-01

## Summary
DM lacks a unified MariaDB-to-TiDB schema transformer in the dump-load path.

## Impact on DM
Full load can fail when MariaDB-specific syntax reaches the loader.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/worker/subtask.go#L51-L76

## Evidence / Downstream Behavior
- None

## Primary Use Cases
- Large bulk migration
- Tight downtime window

## Recommended Handling
- Insert a Schema Transformer after dump and before load.
- Support strict/loose modes for transformation policy.
