# DM vs MySQL/MariaDB Comprehensive Diff & Compatibility Design Report

## 0. How to Navigate This Report
- For a single-file view, read result.md.
- For per-problem details, see output/issues/ (grouped by category).
- For domain templates, see output/scenarios/.


## 1. Scope and Baselines
- DM (tiflow) code version: `142713c45bf390219f102cb7bd7b4dedc6be6e6f`, path: `/Users/benjamin2037/Desktop/workspace/sourcecode/tiflow`
- TiDB code version: `85389ef3740fcc5058a5660e4382e2f1e80c0f28`, path: `/Users/benjamin2037/Desktop/workspace/sourcecode/tidb`
- MariaDB→TiDB rule engine (mariadb2tidb) version: `f7cf760b8d4a25281da9306fbc980c1855b11844`
- MySQL Server code version: `666701570c392a6052341b6ddb9c21869bb1d733` (branch 8.0, sparse checkout)
- MariaDB Server code version: `3218602d3100db9ce7a875511a591cddc173cc16` (branch 10.11, sparse checkout)
- Goal: Without changing DM’s existing MySQL capability, achieve seamless MariaDB-as-source migration (full + incremental) and make MySQL/MariaDB differences and their impact on DM explicit.

## 2. Layered Difference Report (each item includes DM code links and primary use cases)

### A. Replication / GTID / Binlog
| Difference | Impact on DM | DM Touchpoint (code links) | TiDB/Downstream Evidence | Primary Use Cases |
|---|---|---|---|---|
| **GTID format/semantics differ** (MySQL `uuid:interval` vs MariaDB `domain_id-server_id-seq_no`) | Resume position and GTID advance depend on correct parsing; mismatch leads to replay drift or duplication | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/gtid/gtid.go#L33-L122<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/db.go#L54-L64 | — | Finance HA, e-commerce DR, SaaS multi-tenant replication |
| **GTID event type differs** (MySQL `PREVIOUS_GTIDS_EVENT` vs MariaDB `MARIADB_GTID_LIST_EVENT`) | “Previous consistent point” is derived differently; unrecognized event causes incorrect start point | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L32-L110<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L142-L176 | — | Long-running incremental sync, point-in-time resume, switchover |
| **Replication start exposure differs** (MySQL `SHOW MASTER STATUS` vs MariaDB `gtid_binlog_pos`, etc.) | Unified start-point logic can miss MariaDB GTID state | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/utils.go#L98-L236 | — | Cutover for bulk migration, gray/blue-green sync |
| **MariaDB-specific events** (`ANNOTATE_ROWS_EVENT`, `BINLOG_CHECKPOINT_EVENT`) | go-mysql can parse them, but DM only sets send flags and does not explicitly handle; may be skipped and impact audit consistency | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/relay/relay.go#L55-L62<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/relay/relay.go#L1174-L1180 | https://github.com/go-mysql-org/go-mysql/blob/v1.13.0/replication/const.go#L104-L107<br>https://github.com/go-mysql-org/go-mysql/blob/v1.13.0/replication/event.go#L830-L853<br>https://github.com/MariaDB/server/blob/3218602d3100db9ce7a875511a591cddc173cc16/sql/log_event.h#L701-L723 | Audit/compliance replay, CDC |
| **QueryEvent status vars differ** (MariaDB `Q_HRNOW`/`Q_XID`) | SQL_MODE/timezone/timestamp decoding depends on status_vars; differences affect DDL restore and audit accuracy | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/event/util.go#L286-L340 | — | Low-latency trading, timezone-sensitive systems |
| **MySQL 8.0 new binlog events** (ROWS_QUERY / PARTIAL_UPDATE / TRANSACTION_PAYLOAD) | MariaDB doesn’t emit them; cross-version replication requires explicit detection or DM may lose events or fail parsing | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L32-L110 | https://github.com/mysql/mysql-server/blob/666701570c392a6052341b6ddb9c21869bb1d733/libbinlogevents/include/binlog_event.h#L333-L358 | Legacy enterprise cross-version sync, heterogeneous upgrades |

### B. DDL Syntax / Objects
| Difference | Impact on DM | DM Touchpoint (code links) | TiDB/Downstream Evidence | Primary Use Cases |
|---|---|---|---|---|
| **CREATE OR REPLACE SCHEMA (MariaDB)** | Not supported by TiDB DDL/Tracker; DDL fails or is mishandled | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/schema/tracker.go#L184-L220 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/executor.go#L289-L305<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/schematracker/dm_tracker.go#L101-L109 | Multi-tenant provisioning, automated ops |
| **Sequence DDL (CREATE/ALTER/DROP SEQUENCE)** | Parser recognizes it but Tracker doesn’t; may panic or drift schema | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L185-L220<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/schema/tracker.go#L184-L220 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L1819-L1849 | Order IDs, finance billing |
| **System-Versioned Tables (MariaDB WITH SYSTEM VERSIONING)** | TiDB parser only supports SYSTEM_TIME partition; system-versioned tables can’t be parsed/executed directly | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L19-L45<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/parser.y#L4840-L4859 | Audit/compliance history, government archiving |
| **MariaDB-specific table options** (PAGE_CHECKSUM/PAGE_COMPRESSED/TRANSACTIONAL/SEQUENCE/IETF_QUOTES, etc.) | TiDB parses but ignores; semantics are lost; DM passthrough yields invalid storage parameters | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/parser.y#L13149-L13186<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L3033-L3063 | Large table storage optimization, hot/cold tiering |
| **MariaDB storage engines** (ARIA/MYROCKS/TOKUDB) | TiDB only validates engine names; no engine semantics; DM must convert/strip | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/planner/core/preprocess.go#L1476-L1516 | High-write logs, archival storage |
| **ALTER/CREATE INDEX IF EXISTS/IF NOT EXISTS (MariaDB extension)** | DM re-serializes DDL via TiDB AST restore; semantics can differ | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L145-L195<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L940-L944<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L1931-L1933<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L3352-L3358 | SaaS rolling upgrades, online schema changes |
| **MariaDB comment prefix `/*M...*/`** | TiDB treats as comments only; conditional SQL not executed; DM parsing may silently drop semantics | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L19-L45 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/lexer.go#L549-L551 | Multi-version SQL compatibility |

### C. Data Types / Defaults / Constraints
| Difference | Impact on DM | DM Touchpoint (code links) | TiDB/Downstream Evidence | Primary Use Cases |
|---|---|---|---|---|
| **JSON semantics + JSON_VALID CHECK** (MariaDB typical) | TiDB restricts CHECK functions; without transformation DDL fails or semantics drift | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/table/constraint.go#L83-L127 | E-commerce attributes, Web3 metadata, user profiles |
| **TEXT/BLOB/JSON default value restrictions** | TiDB strictly forbids defaults; DM passthrough fails DDL | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/add_column.go#L1208-L1238 | CMS/content platforms, logging systems |
| **CHECK constraint function limits** (UUID/NOW/RAND, etc.) | Common MariaDB functions are disallowed in TiDB CHECK; without rewrite DDL fails | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/table/constraint.go#L83-L127 | Risk control, data quality rules |
| **Generated column dependency & JSON generated column** | TiDB is stricter on order/expressions; DM without adjustments fails | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/generated_column.go#L32-L76 | BI/reporting, derived metrics |
| **UUID type/name handling** (common in MariaDB) | TiDB has no UUID type; DM lacks preprocessing, causing DDL failures or index conflicts | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/parser/loader.go#L89-L205 | Distributed IDs, cross-system integration |

### D. Index / Key Length
| Difference | Impact on DM | DM Touchpoint (code links) | TiDB/Downstream Evidence | Primary Use Cases |
|---|---|---|---|---|
| **JSON cannot be indexed; BLOB/TEXT must have prefixes** | Indexes allowed in MariaDB fail in TiDB; DM doesn’t auto-prefix/relax | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/index.go#L244-L267 | Search/tag/log indexes |
| **Stricter index length/prefix limits** | Long text/multibyte charset indexes exceed limits; require rule-based truncation | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/keylength.go#L8-L40<br>https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/index_prefix.go#L9-L74 | Cross-border e-commerce, multilingual search |

### E. Charset / Collation
| Difference | Impact on DM | DM Touchpoint (code links) | TiDB/Downstream Evidence | Primary Use Cases |
|---|---|---|---|---|
| **MariaDB collation set differs (incl. NO PAD)** | DM only fills defaults, no mapping; unsupported collations can break DDL | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1392-L1459 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/collation.go#L11-L209 | Multilingual e-commerce, social platforms |
| **Limited charset conversion in DML** | DM has explicit latin1/gbk conversions; other charset differences depend on downstream compatibility | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/dml.go#L50-L127 | — | Legacy migrations, historical data consolidation |

### F. Tooling / Pipeline
| Difference | Impact on DM | DM Touchpoint (code links) | TiDB/Downstream Evidence | Primary Use Cases |
|---|---|---|---|---|
| **No Schema Transformer in full chain** (Dump→Load gap) | MariaDB DDL fails at Loader, or produces incompatible schema | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/worker/subtask.go#L51-L76 | — | Large bulk migration, tight downtime window |
| **Dumpling only fills collation, not MariaDB syntax differences** | JSON/UUID/table options/system-versioned features are dumped as-is; TiDB parser can reject | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/dumpling/dumpling.go#L322-L356 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/sql.go#L87-L101<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/dump.go#L430-L478<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/dump.go#L536-L587 | Large-scale migration, heterogeneous schemas |

## 3. Design for Seamless MariaDB Support (modules)

### 3.1 Upstream Capability Detection & Session Init
- Drive GTID parsing, start position, and binlog event handling by flavor; DM touchpoints: `dm/pkg/conn/db.go`, `dm/pkg/conn/utils.go`, `dm/pkg/binlog/reader/util.go`.
- Explicitly handle MariaDB events (GTID_LIST_EVENT/ANNOTATE_ROWS_EVENT/BINLOG_CHECKPOINT_EVENT) with compatibility policy and metrics.

### 3.2 Schema Transformer (DDL conversion layer)
- Embed a standalone module to convert MariaDB DDL into TiDB-compatible DDL at AST level.
- Reuse mariadb2tidb rule set:
  - Engine: `internal/transformer/engine.go`
  - Preprocess (UUID/encryption/charset): `internal/parser/loader.go`
  - Rules: `collation / keylength / index_prefix / text_blob_defaults / json_check / json_generated / function_default`
- DM integration points:
  - **Full**: after Dumpling schema dump, before Loader.
  - **Incremental**: after `dm/syncer/ddl.go` parse, before Tracker + downstream execution.

#### 3.2.1 Module Interface (recommended)
```go
// TransformMode indicates the usage: full schema dump or incremental binlog DDL.
type TransformMode string
const (
    TransformFullDump  TransformMode = "full_dump"
    TransformIncrement TransformMode = "incremental"
)

// TransformResult describes output plus audit metadata.
type TransformResult struct {
    SQL            string   // transformed SQL
    AppliedRules   []string // matched rule list
    RemovedClauses []string // stripped clauses (for audit)
    Warnings       []string // semantic downgrade notes
}

// SchemaTransformer defines the unified DDL conversion interface.
type SchemaTransformer interface {
    TransformSQL(ctx context.Context, mode TransformMode, sql string) (TransformResult, error)
    TransformBatch(ctx context.Context, mode TransformMode, sqls []string) ([]TransformResult, error)
}
```

#### 3.2.2 Failure Policy & Rollback Semantics
- **Strict**: block immediately on uncovered DDL and preserve original SQL (avoid semantic drift).
- **Loose**: strip incompatible clauses and proceed; record `RemovedClauses/Warnings` for audit and alerts.
- **Transactional**: for a single binlog DDL event split into multiple statements, ensure all-or-nothing consistency.

#### 3.2.3 Test Matrix (full/incremental/rollback)
| Scenario | Coverage | Expected Result |
|---|---|---|
| Full schema dump | JSON/UUID/Collation/Sequence/System-versioned/Table options | rules applied; strict fails on unsupported items, loose strips |
| Incremental DDL | CREATE/ALTER/DROP/RENAME + online DDL | consistent transformations; Tracker and downstream stay aligned |
| Data type diffs | TEXT/BLOB/JSON defaults, function defaults | defaults stripped or rewritten; no execution failure |
| Constraints & generated columns | CHECK/dependencies/JSON_VALID | unsupported parts stripped/rewritten; DDL executes |
| Rollback/failure | rule gaps or parser failure | strict stops; loose records downgrade and continues |

### 3.3 Tracker Consistency & DDL Apply
- Apply transformed DDL to both Tracker and downstream to keep DML mapping consistent.
- Extend Tracker coverage for Sequence/system-versioned semantics to avoid panics.

### 3.4 Charset/Collation Mapping & DML Conversion
- Standardize collation mapping to eliminate MariaDB-only collations (NO PAD).
- Apply charset consistency on DML to reduce implicit differences beyond latin1/gbk.

### 3.5 Config & Strategy
- Provide rule toggles and strict/loose modes; strict preserves semantics, loose favors executability.
- Make rules and strategies explicit in task-level config for replay/audit.

## 4. Rule Mapping (MariaDB → TiDB)
| Rule | Purpose | Implementation |
|---|---|---|
| Collation mapping | Normalize MariaDB collation set | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/collation.go#L11-L209 |
| KeyLength / IndexPrefix | Fit TiDB index length/prefix limits | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/keylength.go#L8-L40<br>https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/index_prefix.go#L9-L74 |
| Text/Blob/JSON defaults | Remove unsupported defaults | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/text_blob_defaults.go#L9-L66 |
| JSON Check / JSON generated | Remove JSON_VALID checks and incompatible generated columns | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/json_check.go#L9-L73<br>https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/json_generated.go#L9-L75 |
| Function defaults | Remove/replace unsupported default functions | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/function_default.go#L9-L70 |
| UUID preprocessing | UUID → CHAR(36) and fix constraints/defaults | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/parser/loader.go#L89-L205 |

## 5. Scenario Templates (domain-specific compatibility focus)

### 5.1 Finance (payments, core banking, risk control)
- **Data profile**: strict ACID, low tolerance for schema drift, strong audit requirements.
- **Common MariaDB features**: sequences, CHECK constraints, generated columns, system-versioned tables, strict time-related status vars.
- **Top risks**: GTID resume correctness, unsupported CHECK functions, system-versioned DDL, timezone/status var parsing.
- **DM actions**: strict mode; enable Schema Transformer rules for function defaults, JSON_CHECK, UUID preprocessing, text/blob defaults; explicit MariaDB GTID event handling; enforce collation mapping.
- **Validation**: end-to-end checksum, DDL replay audit, binlog continuity tests, strict fail-fast on unsupported DDL.

### 5.2 E-commerce (orders, catalog, inventory)
- **Data profile**: wide tables, high write peaks, frequent online DDL, multilingual data.
- **Common MariaDB features**: JSON attributes + JSON_VALID checks, long indexes, text/blob defaults, MariaDB collations.
- **Top risks**: DDL failures on JSON/CHECK/defaults, index length/prefix violations, collation incompatibility.
- **DM actions**: enable index_prefix/keylength/collation rules; strip unsupported table options; normalize DDL IF EXISTS semantics; use balanced (strict for data, loose for DDL) policy.
- **Validation**: transformed DDL diff review, index length audit, collation mapping report, peak write lag testing.

### 5.3 Government / Public Sector (archival, compliance)
- **Data profile**: long retention, auditable history, conservative change control.
- **Common MariaDB features**: system-versioned tables, conditional SQL comments, large text columns.
- **Top risks**: system-versioned DDL unsupported by TiDB, conditional SQL dropped, text/blob default restrictions.
- **DM actions**: strict mode; block or explicitly rewrite system-versioned DDL; record all removed clauses and warnings; keep audit logs for every transformation.
- **Validation**: compliance review of schema deltas, full data checksum, audit trail parity checks.

### 5.4 Social Platforms (user content, messaging)
- **Data profile**: very high concurrency, utf8mb4/emoji, large secondary indexes, semi-structured data.
- **Common MariaDB features**: MariaDB collations, JSON columns, generated columns, long index prefixes.
- **Top risks**: collation mapping gaps, index length limits, JSON index incompatibility.
- **DM actions**: apply collation mapping and index prefix rules; convert or drop JSON indexes; keep Tracker and downstream DDL aligned.
- **Validation**: encoding/emoji integrity checks, search/index parity sampling, latency under peak load.

### 5.5 AI / ML Systems (feature store, metadata)
- **Data profile**: semi-structured features, frequent schema evolution, large payloads.
- **Common MariaDB features**: JSON/text defaults, generated columns, long indexes for feature lookup.
- **Top risks**: unsupported defaults, generated column constraints, index length issues.
- **DM actions**: loose mode for DDL (with warnings), strict for DML; enable default/JSON/generate-column rules; maintain detailed transform logs.
- **Validation**: feature distribution sampling, schema-change replay tests, latency and lag monitoring.

### 5.6 Logging / Observability Platforms
- **Data profile**: append-heavy, huge tables, wide text columns, high throughput.
- **Common MariaDB features**: text/blob defaults, engine declarations, long indexes on tags.
- **Top risks**: DDL failures due to defaults, index prefix violations, unsupported engine/table options.
- **DM actions**: loose mode; strip incompatible table options; apply index prefix rules; tune relay and apply throughput.
- **Validation**: ingestion throughput, end-to-end lag, sampling-based data integrity checks.

## 6. Conclusion
- DM already supports basic binlog/GTID for MySQL/MariaDB, but has systematic gaps in MariaDB DDL syntax, object features, data type semantics, indexes, and collations.
- Introducing a Schema Transformer, binlog event compatibility layer, and Tracker consistency strategy can make MariaDB-as-source migration predictable, auditable, and seamless.
