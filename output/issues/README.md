# Issue Index

Categories and their issue files:

## replication-gtid-binlog
- A-01 replication-gtid-binlog/01-gtid-format.md - GTID format and semantics differ
- A-02 replication-gtid-binlog/02-gtid-event-type.md - GTID event type differs
- A-03 replication-gtid-binlog/03-replication-start-exposure.md - Replication start exposure differs
- A-04 replication-gtid-binlog/04-mariadb-specific-events.md - MariaDB-specific binlog events
- A-05 replication-gtid-binlog/05-queryevent-status-vars.md - QueryEvent status variables differ
- A-06 replication-gtid-binlog/06-mysql-80-binlog-events.md - MySQL 8.0 binlog events not in MariaDB

## ddl-objects
- B-01 ddl-objects/01-create-or-replace-schema.md - CREATE OR REPLACE SCHEMA
- B-02 ddl-objects/02-sequence-ddl.md - Sequence DDL
- B-03 ddl-objects/03-system-versioned-tables.md - System-versioned tables
- B-04 ddl-objects/04-mariadb-table-options.md - MariaDB-specific table options
- B-05 ddl-objects/05-mariadb-storage-engines.md - MariaDB storage engines
- B-06 ddl-objects/06-index-if-exists.md - Index IF EXISTS/IF NOT EXISTS
- B-07 ddl-objects/07-mariadb-conditional-comments.md - MariaDB conditional comments

## data-types-constraints
- C-01 data-types-constraints/01-json-check.md - JSON_VALID and CHECK
- C-02 data-types-constraints/02-text-blob-json-defaults.md - TEXT/BLOB/JSON defaults
- C-03 data-types-constraints/03-check-constraint-function-limits.md - CHECK function limits
- C-04 data-types-constraints/04-generated-columns.md - Generated column rules
- C-05 data-types-constraints/05-uuid-type.md - UUID type handling

## index-key-length
- D-01 index-key-length/01-json-text-index.md - JSON and TEXT index limits
- D-02 index-key-length/02-index-length-limits.md - Index length limits

## charset-collation
- E-01 charset-collation/01-collation-set.md - Collation set differences
- E-02 charset-collation/02-dml-charset-conversion.md - DML charset conversion

## tooling-pipeline
- F-01 tooling-pipeline/01-no-schema-transformer.md - No schema transformer in full chain
- F-02 tooling-pipeline/02-dumpling-gap.md - Dumpling MariaDB gap
