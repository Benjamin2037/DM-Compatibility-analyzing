# DM 与 MySQL/MariaDB 全量差异对比与兼容性设计报告

## 1. 范围与基线
- DM（tiflow）代码版本：`142713c45bf390219f102cb7bd7b4dedc6be6e6f`，路径：`/Users/benjamin2037/Desktop/workspace/sourcecode/tiflow`
- TiDB 代码版本：`85389ef3740fcc5058a5660e4382e2f1e80c0f28`，路径：`/Users/benjamin2037/Desktop/workspace/sourcecode/tidb`
- MariaDB→TiDB 规则引擎（mariadb2tidb）版本：`f7cf760b8d4a25281da9306fbc980c1855b11844`
- MySQL Server 代码版本：`666701570c392a6052341b6ddb9c21869bb1d733`（branch 8.0，稀疏拉取）
- MariaDB Server 代码版本：`3218602d3100db9ce7a875511a591cddc173cc16`（branch 10.11，稀疏拉取）
- 目标：在不改变 DM 既有 MySQL 能力的前提下，实现 MariaDB 作为上游的无缝迁移（全量 + 增量），并明确 MySQL/MariaDB 差异对 DM 的影响。

## 2. 分层差异报告（每项含 DM 代码链接与应用场景）

### A. 复制 / GTID / Binlog
| 差异点 | 影响（DM） | DM 触点（代码链接） | TiDB/下游证据 | 主要应用场景 |
|---|---|---|---|---|
| **GTID 格式与语义不同**（MySQL `uuid:interval` vs MariaDB `domain_id-server_id-seq_no`） | 断点续传与位点推进依赖 GTID 解析；格式识别错误将导致回放错位或重复 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/gtid/gtid.go#L33-L122<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/db.go#L54-L64 | — | 金融主备切换、电商跨地域容灾、SaaS 多租户复制 |
| **GTID 事件类型差异**（MySQL `PREVIOUS_GTIDS_EVENT` vs MariaDB `MARIADB_GTID_LIST_EVENT`） | 获取“上一个一致点”的方式不同；若事件未识别，增量起点无法校准 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L32-L110<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L142-L176 | — | 长周期增量同步、断点续传、主从切换 |
| **复制起点暴露方式不同**（MySQL `SHOW MASTER STATUS` vs MariaDB `gtid_binlog_pos` 等） | 统一的起点解析会导致 MariaDB 位点缺失或不正确 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/utils.go#L98-L236 | — | 存量迁移切换、灰度同步 |
| **MariaDB 特有事件**（`ANNOTATE_ROWS_EVENT`、`BINLOG_CHECKPOINT_EVENT`） | go-mysql 已能解析事件类型，但 DM 仅设置发送标志、未显式处理；可能被动跳过或影响审计一致性 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/relay/relay.go#L55-L62<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/relay/relay.go#L1174-L1180 | https://github.com/go-mysql-org/go-mysql/blob/v1.13.0/replication/const.go#L104-L107<br>https://github.com/go-mysql-org/go-mysql/blob/v1.13.0/replication/event.go#L830-L853<br>https://github.com/MariaDB/server/blob/3218602d3100db9ce7a875511a591cddc173cc16/sql/log_event.h#L701-L723 | 审计/合规回放、CDC |
| **QueryEvent 状态变量差异**（MariaDB `Q_HRNOW`/`Q_XID`） | SQL_MODE/时区/时间戳解析依赖 status_vars，字段差异会影响 DDL 还原与审计准确性 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/event/util.go#L286-L340 | — | 低延迟交易、时区敏感业务 |
| **MySQL 8.0 新增 binlog 事件**（ROWS_QUERY / PARTIAL_UPDATE / TRANSACTION_PAYLOAD） | MariaDB 不产生这些事件，跨版本复制时 DM 需要显式识别与处理，否则可能丢事件或解析失败 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L32-L110 | https://github.com/mysql/mysql-server/blob/666701570c392a6052341b6ddb9c21869bb1d733/libbinlogevents/include/binlog_event.h#L333-L358 | 传统企业库跨版本同步、异构升级 |

### B. DDL 语法 / 对象
| 差异点 | 影响（DM） | DM 触点（代码链接） | TiDB/下游证据 | 主要应用场景 |
|---|---|---|---|---|
| **CREATE OR REPLACE SCHEMA（MariaDB）** | TiDB DDL/Tracker 不支持，DDL 会失败或被错误处理 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/schema/tracker.go#L184-L220 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/executor.go#L289-L305<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/schematracker/dm_tracker.go#L101-L109 | 多租户环境初始化、自动化运维 |
| **Sequence DDL（CREATE/ALTER/DROP SEQUENCE）** | Parser 可识别但 Tracker 未覆盖，可能触发 panic 或 schema 不一致 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L185-L220<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/schema/tracker.go#L184-L220 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L1819-L1849 | 订单号/流水号、金融计费 |
| **System-Versioned Tables（MariaDB WITH SYSTEM VERSIONING）** | TiDB 解析语法仅包含 SYSTEM_TIME 分区，系统版本表无法直接解析/执行 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L19-L45<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/parser.y#L4840-L4859 | 审计/合规留痕、政企归档 |
| **MariaDB 特有表选项**（PAGE_CHECKSUM/PAGE_COMPRESSED/TRANSACTIONAL/SEQUENCE/IETF_QUOTES 等） | TiDB 解析但忽略，语义丢失；DM 若直传将导致存储参数无效 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/parser.y#L13149-L13186<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L3033-L3063 | 大表存储优化、冷热数据分层 |
| **MariaDB 存储引擎**（ARIA/MYROCKS/TOKUDB） | TiDB 仅做名称白名单校验，不提供引擎语义；DM 需转换/剥离 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/planner/core/preprocess.go#L1476-L1516 | 高写入日志、归档存储 |
| **ALTER/CREATE INDEX 的 IF EXISTS/IF NOT EXISTS（MariaDB 扩展）** | DM 复写 DDL 时会按 TiDB Restore 规则输出，语义差异可能导致 DDL 行为变化 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L145-L195<br>https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L940-L944<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L1931-L1933<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L3352-L3358 | SaaS 滚动发布、在线变更 |
| **MariaDB 注释前缀 `/*M...*/`** | TiDB 仅识别为注释，条件语句不执行；DM 解析后可能静默丢失语义 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L19-L45 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/lexer.go#L549-L551 | 多版本 SQL 脚本兼容 |

### C. 数据类型 / 默认值 / 约束
| 差异点 | 影响（DM） | DM 触点（代码链接） | TiDB/下游证据 | 主要应用场景 |
|---|---|---|---|---|
| **JSON 语义差异 + JSON_VALID Check**（MariaDB 常用 CHECK 约束） | TiDB 对 CHECK 表达式函数有限制；DM 不转换将导致 DDL 失败或语义偏差 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/table/constraint.go#L83-L127 | 电商商品属性、Web3 元数据、用户画像 |
| **TEXT/BLOB/JSON 默认值限制** | TiDB 严格禁止这些类型的默认值；DM 直传会导致建表失败 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/add_column.go#L1208-L1238 | CMS/内容平台、日志系统 |
| **CHECK 约束函数限制**（UUID/NOW/RAND 等） | MariaDB 常用函数在 TiDB CHECK 中被限制，DM 不改写会直接报错 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/table/constraint.go#L83-L127 | 风控/数据质量校验 |
| **生成列依赖与 JSON 生成列** | TiDB 对生成列依赖顺序/表达式更严格，DM 不调整会报错 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/generated_column.go#L32-L76 | 报表/分析型查询、衍生指标 |
| **UUID 类型/命名处理**（MariaDB 常见） | TiDB 无 UUID 类型；DM 缺少预处理会造成 DDL 失败或索引冲突 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/parser/loader.go#L89-L205 | 分布式 ID、跨系统数据整合 |

### D. 索引 / Key 长度
| 差异点 | 影响（DM） | DM 触点（代码链接） | TiDB/下游证据 | 主要应用场景 |
|---|---|---|---|---|
| **JSON 不可建普通索引，BLOB/TEXT 必须前缀** | MariaDB 允许的索引在 TiDB 会失败；DM 未做前缀补齐/降级 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/index.go#L244-L267 | 搜索/标签/日志索引 |
| **索引长度与前缀限制更严格** | 长文本/多字节字符集索引易超限；需要规则化截断 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/keylength.go#L8-L40<br>https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/index_prefix.go#L9-L74 | 跨境电商、多语言检索 |

### E. 字符集 / Collation
| 差异点 | 影响（DM） | DM 触点（代码链接） | TiDB/下游证据 | 主要应用场景 |
|---|---|---|---|---|
| **MariaDB Collation 集合差异（含 NO PAD）** | DM 仅在缺省时补齐 Collation，不做映射；不支持的 Collation 会导致建表失败 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1392-L1459 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/collation.go#L11-L209 | 多语言/跨境电商、社交 |
| **DML 字符集转换覆盖有限** | DM 对 latin1/gbk 有显式转换，其它字符集差异依赖下游兼容性 | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/dml.go#L50-L127 | — | 传统系统迁移、历史库整合 |

### F. 工具链 / 流程
| 差异点 | 影响（DM） | DM 触点（代码链接） | TiDB/下游证据 | 主要应用场景 |
|---|---|---|---|---|
| **全量链路缺少 Schema Transformer**（Dump→Load 之间无转换层） | MariaDB DDL 在 Loader 阶段直接失败，或生成不兼容 schema | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/worker/subtask.go#L51-L76 | — | 大型全量迁移、停机窗口敏感 |
| **Dumpling 仅补齐 Collation，无法消除 MariaDB 语法差异** | JSON/UUID/表选项/系统版本表等仍按原样输出；当 TiDB parser 不支持时直接保留原 SQL | https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/dumpling/dumpling.go#L322-L356 | https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/sql.go#L87-L101<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/dump.go#L430-L478<br>https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/dump.go#L536-L587 | 大规模迁移、异构语法库 |

## 3. 无缝支持 MariaDB 的设计（子模块）

### 3.1 上游能力探测与会话初始化
- 统一使用 flavor 驱动 GTID 解析、起点读取与 binlog 事件路径。关联 DM 触点：`dm/pkg/conn/db.go`、`dm/pkg/conn/utils.go`、`dm/pkg/binlog/reader/util.go`。
- 将 MariaDB 特有事件（GTID_LIST_EVENT/ANNOTATE_ROWS_EVENT/BINLOG_CHECKPOINT_EVENT）纳入事件兼容层，形成显式处理策略与统计。

### 3.2 Schema Transformer（DDL 转换层）
- 作为独立模块嵌入 DM：对上游 MariaDB DDL 进行 AST 级转换，输出 TiDB 可执行 DDL。
- 复用 mariadb2tidb 规则体系：
  - 规则引擎：`internal/transformer/engine.go`
  - 预处理（UUID/加密选项/字符集替换）：`internal/parser/loader.go`
  - 规则集合：`collation / keylength / index_prefix / text_blob_defaults / json_check / json_generated / function_default`
- 对应 DM 集成点：
  - **全量**：Dumpling schema 输出后、Loader 前置转换（补齐/替换/剥离不兼容项）。
  - **增量**：在 `dm/syncer/ddl.go` 解析后、写入 Tracker/下游前执行转换，并复写 DDL。

#### 3.2.1 模块接口定义（建议实现）
```go
// TransformMode 表示使用场景：全量 schema dump 或增量 binlog DDL。
type TransformMode string
const (
    TransformFullDump  TransformMode = "full_dump"
    TransformIncrement TransformMode = "incremental"
)

// TransformResult 描述转换结果与审计信息。
type TransformResult struct {
    SQL            string   // 转换后的 SQL
    AppliedRules   []string // 命中的规则列表
    RemovedClauses []string // 被剥离的子句（用于审计）
    Warnings       []string // 语义降级提示
}

// SchemaTransformer 定义统一的 DDL 转换接口。
type SchemaTransformer interface {
    TransformSQL(ctx context.Context, mode TransformMode, sql string) (TransformResult, error)
    TransformBatch(ctx context.Context, mode TransformMode, sqls []string) ([]TransformResult, error)
}
```

#### 3.2.2 失败策略与回滚语义
- 严格模式：一旦规则无法覆盖的 DDL 出现，直接阻断并记录原始 SQL（避免语义偏差）。
- 宽松模式：剥离不兼容子句并继续；同时写入 `RemovedClauses/Warnings` 以供审计与告警。
- 事务性：对同一 binlog DDL 事件生成的多条拆分 SQL，保证“全成或全失败”一致处理与回滚。

#### 3.2.3 测试矩阵（覆盖全量/增量/回滚）
| 场景 | 覆盖点 | 预期结果 |
|---|---|---|
| 全量 schema dump | JSON/UUID/Collation/Sequence/系统版本表/表选项 | 规则命中可执行；不支持项在 strict 模式报错、loose 模式剥离 |
| 增量 DDL | CREATE/ALTER/DROP/RENAME + 在线 DDL | 与全量一致的转换结果；Tracker 与下游 schema 保持一致 |
| 数据类型差异 | TEXT/BLOB/JSON 默认值、函数默认值 | 默认值剥离或转换；无执行失败 |
| 约束与生成列 | CHECK/生成列依赖/JSON_VALID | 不支持项剥离或重写，DDL 可执行 |
| 回滚/失败 | 规则缺失或 parser 失败 | strict 失败并停止；loose 记录降级并继续 |

### 3.3 Tracker 一致性与 DDL 落地
- 将转换后的 DDL 同步应用于 Schema Tracker 与下游执行，保证 DML 映射一致。
- 扩展 Tracker 对 Sequence、系统版本表等 MariaDB 语义的覆盖，避免异常分支触发 panic。

### 3.4 字符集/Collation 映射与 DML 转换
- 统一 Collation 映射表，消除 MariaDB NO PAD 等不兼容 collations。
- DML 层按映射规则做字符集一致化，减少 latin1/gbk 以外字符集的隐式差异。

### 3.5 配置与策略
- 提供规则开关、严格/宽松两种执行策略：严格模式确保语义一致；宽松模式保证可落地。
- 所有规则与策略在任务级配置中显式声明，便于回放与审计。

## 4. 规则映射表（MariaDB → TiDB）
| 规则 | 作用 | 规则实现链接 |
|---|---|---|
| Collation 映射 | 处理 MariaDB Collation 集合差异 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/collation.go#L11-L209 |
| KeyLength / IndexPrefix | 适配 TiDB 索引长度与前缀限制 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/keylength.go#L8-L40<br>https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/index_prefix.go#L9-L74 |
| Text/Blob/JSON 默认值 | 移除不支持的默认值 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/text_blob_defaults.go#L9-L66 |
| JSON Check / JSON 生成列 | 移除 JSON_VALID 约束与不兼容生成列 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/json_check.go#L9-L73<br>https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/json_generated.go#L9-L75 |
| 函数型默认值 | 移除/替换 TiDB 不支持的默认函数 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/function_default.go#L9-L70 |
| UUID 预处理 | UUID → CHAR(36) 并修正约束/默认值 | https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/parser/loader.go#L89-L205 |

## 5. 结论
- DM 已具备 MySQL/MariaDB 的基础 binlog 与 GTID 处理，但在 MariaDB 的 DDL 语法、对象特性、数据类型语义、索引/Collation 方面存在系统性差异。
- 通过引入 Schema Transformer、事件兼容层与 Tracker 一致性策略，可将 MariaDB 上游迁移收敛为可预测、可审计、可回放的无缝流程。
