# DM Compatibility Goals for One-Click Migration (MySQL and MariaDB)

## Objective
Make MySQL and MariaDB source migrations to TiDB simple, predictable, and close to one-click, without breaking existing MySQL behavior. The system should detect differences, transform incompatibilities safely, and provide a clear, auditable report.

## Guiding Principles
- No silent data loss or schema drift
- Deterministic and replayable transformations
- Clear observability and audit trail for every rewrite
- Safe defaults with explicit opt-in for lossy changes
- Same config works for full and incremental migration

## Priority 0 (Must Have)
These unblock reliable, mostly automatic migration.

1. Source flavor detection and GTID normalization
   - Detect MySQL vs MariaDB reliably
   - Normalize GTID sets and start position logic
   - Explicit handling for MariaDB GTID list events

2. Unified Schema Transformer for full and incremental
   - Apply the same DDL rewrite rules in dump-load and binlog replay
   - Ensure tracker and downstream receive identical transformed DDL

3. Core DDL compatibility rules
   - JSON and CHECK constraints rewrite or removal
   - TEXT/BLOB/JSON default removal
   - UUID type preprocessing
   - Index prefix and key length rules
   - Collation mapping for unsupported MariaDB collations

4. Strict vs loose policy with audit
   - Strict: fail fast on unsupported DDL
   - Loose: transform and continue with warnings
   - Always emit a transformation report

5. Pre-flight compatibility check
   - Scan schema before migration
   - Generate a blocking list and a fix list
   - Provide recommended config toggles

## Priority 1 (High Value)
These reduce manual effort and raise success rate for complex schemas.

1. MariaDB object features
   - Sequences, system-versioned tables, storage engine mapping
   - Conditional comment parsing (/*M...*/)

2. Binlog event coverage and testing
   - MariaDB specific events and QueryEvent status vars
   - Cross-version MySQL 8.0 event safety

3. Charset and DML conversion
   - Expand charset conversions beyond latin1 and gbk
   - Provide charset policy checks

4. Migration UX and one-click flow
   - Auto-generate DM tasks with recommended rules
   - Dry-run mode that produces a report only
   - One-click template profiles for common scenarios

## Priority 2 (Quality and Scale)

1. Automated regression tests
   - Golden DDL transformations with rule snapshots
   - Binlog event compatibility fixtures

2. Performance safe defaults
   - Throughput tuning for relay and apply
   - Adaptive throttling when downstream is slow

3. Observability
   - Metrics for DDL rewrites, dropped clauses, skipped events
   - Compatibility dashboards per task

## Deliverables for Users
- One-click migration wizard (or CLI shortcut)
- Compatibility report with fixes and risks
- Config presets for common domains
- Consistent behavior across full and incremental sync

## Non-Goals (Short Term)
- Perfect semantic equivalence for every MariaDB-specific feature
- Automatic conversion of all system-versioned behavior
- Transparent engine-level behavior mapping for non-InnoDB engines
