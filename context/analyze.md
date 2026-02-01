# DMCompatibility 分析上下文（持续更新）

- 更新日期：2026-02-01
- 目标：评估 DM 与 MySQL/MariaDB 兼容性差异，形成详细 gap 清单与设计文档。
- DM 代码路径：/Users/benjamin2037/Desktop/workspace/sourcecode/tiflow
- DM 代码版本（tiflow）：142713c45bf390219f102cb7bd7b4dedc6be6e6f（origin: git@github.com:pingcap/tiflow.git）
- 产出目录：/Users/benjamin2037/Desktop/workspace/sourcecode/project/DMCompatibility/output/result.md
- 中间对比目录：/Users/benjamin2037/Desktop/workspace/sourcecode/project/DMCompatibility/context

## 过程记录
- 初始化 context/output/codeDiff 目录与基线信息。
- 补充 TiDB 侧 MariaDB 解析/限制点与 DM 关键触点链接（表选项、CREATE OR REPLACE、Sequence、Check/默认值/索引限制等）。
- 输出分层差异报告与场景分析到 result.md，并同步规则映射与设计方案。
- 补充 go-mysql MariaDB 事件支持行号与 dumpling schema dump 行为，完善 DM 事件/全量链路差异说明。
- 下载 go-mysql 源码到本地用于对照：`project/DMCompatibility/codeDiff/mariadb/go-mysql`，commit `4dea9fec1aeb19916d9358a8fffb20d094733b2c`（v1.13.0）。
- 下载 MySQL Server 源码到本地用于对照：`project/DMCompatibility/codeDiff/mysql/mysql-server`，commit `666701570c392a6052341b6ddb9c21869bb1d733`（branch 8.0，稀疏拉取）。
- 旧的未完成克隆已移动为 `project/DMCompatibility/codeDiff/mysql/mysql-server.incomplete-20260201` 以避免污染当前对照。
- 下载 MariaDB Server 源码到本地用于对照：`project/DMCompatibility/codeDiff/mariadb/mariadb-server`，commit `3218602d3100db9ce7a875511a591cddc173cc16`（branch 10.11，稀疏拉取）。

## 资料收集清单（待补充）
- MySQL 官方兼容性/语法/数据类型文档
- MariaDB 官方兼容性/语法/数据类型文档
- MariaDB vs MySQL 差异汇总
- mariadb2tidb 开源实现（GitHub）
- DM/TiFlow 相关文档与代码引用


## 资料收集进展（已获取）
- MariaDB vs MySQL 差异与兼容性：
  - MariaDB 10.5 vs MySQL 8.0 不兼容清单（MariaDB KB）
  - MariaDB 与 MySQL 复制兼容性（MariaDB KB）
  - MariaDB GTID / GTID_LIST_EVENT 文档（MariaDB KB）
  - MariaDB JSON 类型文档（MariaDB Docs）
  - MariaDB 系统版本表（System-Versioned Tables）与 Sequence 文档（MariaDB KB）
- MySQL 官方文档：
  - GTID 格式与存储（MySQL 8.0 Reference Manual）
  - SHOW MASTER STATUS/SHOW BINARY LOG STATUS（MySQL 8.0 Reference Manual）
  - JSON 数据类型（MySQL 8.0 Reference Manual）
- 代码与实现：
  - DM/TiFlow 源码（本地 commit 142713c...）
  - mariadb2tidb（本地克隆，commit f7cf760b8d4a25281da9306fbc980c1855b11844）

## mariadb2tidb 代码要点（本地对照）
- 规则引擎与 AST 处理：internal/transformer/engine.go
- 预处理与 SQL 输入清洗：internal/parser/loader.go（UUID/加密选项/字符集替换）
- 规则集合：internal/rules/{collation, keylength, index_prefix, text_blob_defaults, json_check, function_default, json_generated}.go
- 规则注册与优先级：internal/rules/registry.go
- 配置与规则启用/禁用：internal/config/config.go

## DM 关键代码定位（后续输出将对应行号链接）
- 上游 Flavor/GTID/版本判断：dm/config/source_config.go, dm/pkg/conn/db.go, dm/pkg/conn/utils.go, dm/pkg/gtid/gtid.go
- Binlog/GTID 事件处理：dm/pkg/binlog/event/common.go, dm/pkg/binlog/event/util.go, dm/pkg/binlog/reader/util.go
- Relay（MariaDB 注释行事件标志）：dm/relay/relay.go
- DDL 解析与重写：dm/pkg/parser/common.go, dm/syncer/ddl.go
- Schema tracker（TiDB DDL 语义）：dm/pkg/schema/tracker.go
- DML 类型转换与字段处理：dm/syncer/dml.go


## 代码链接索引（GitHub 行号）
> commit: 142713c45bf390219f102cb7bd7b4dedc6be6e6f (pingcap/tiflow)
- SourceConfig/Flavor/GTID 校验：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/config/source_config.go#L68-L111
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/config/source_config.go#L284-L350
- 获取 flavor/GTID/UUID/Parser：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/db.go#L54-L64
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/db.go#L200-L243
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/db.go#L246-L275
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/db.go#L305-L308
- SHOW MASTER STATUS / gtid_binlog_pos：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/conn/utils.go#L98-L236
- GTID 解析：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/gtid/gtid.go#L33-L122
- Binlog GTID 事件与 MariaDB 扩展：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/event/common.go#L34-L143
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/event/util.go#L286-L340
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L32-L110
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/binlog/reader/util.go#L142-L176
- Relay MariaDB Annotate Rows 标志：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/relay/relay.go#L55-L62
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/relay/relay.go#L1174-L1180
- DDL 解析/切分/路由/Collation：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L19-L45
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L185-L375
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L269-L327
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1292-L1327
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/ddl.go#L1392-L1459
- Schema Tracker（TiDB DDL 语义）：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/schema/tracker.go#L55-L219
- DML 类型转换：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/dml.go#L50-L127
- Schema Tracker Exec 分支（未覆盖 Sequence 等 DDL）：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/schema/tracker.go#L184-L220
- 任务流程（dumpling → loader → syncer）：
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/worker/subtask.go#L51-L76

> commit: f7cf760b8d4a25281da9306fbc980c1855b11844 (developer-Bushido/mariadb2tidb)
- 规则引擎：
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/transformer/engine.go#L19-L61
- 预处理（UUID/加密/字符集）：
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/parser/loader.go#L89-L205
- 规则注册：
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/registry.go#L88-L110
- 规则实现（Collation/KeyLength/IndexPrefix/TextBlobDefault/JSONCheck/FunctionDefault/JSONGenerated）：
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/collation.go#L11-L209
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/keylength.go#L8-L40
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/index_prefix.go#L9-L74
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/text_blob_defaults.go#L9-L66
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/json_check.go#L9-L73
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/function_default.go#L9-L70
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/rules/json_generated.go#L9-L75
- 配置模型：
  - https://github.com/developer-Bushido/mariadb2tidb/blob/f7cf760b8d4a25281da9306fbc980c1855b11844/internal/config/config.go#L9-L110


## 输出状态
- 已生成设计文档与差异清单：/Users/benjamin2037/Desktop/workspace/sourcecode/project/DMCompatibility/output/result.md
- 差异清单中的每条均给出 DM 代码行号链接，并在需要处引用官方文档来源。

## 限制说明（供后续迭代）
- TiDB 对 MariaDB 特性支持的细粒度限制未完全纳入（需结合 TiDB 官方兼容性列表进一步细化）。
- Go-MySQL 对 MariaDB 特定 binlog 事件的覆盖度待独立核验（已列入 ToDo）。

## TiDB 代码基线
- TiDB 代码路径：/Users/benjamin2037/Desktop/workspace/sourcecode/tidb
- TiDB 代码版本：85389ef3740fcc5058a5660e4382e2f1e80c0f28（origin: git@github.com:pingcap/tidb.git）
- 需要对照的核心方向：
  - MariaDB 语法/类型支持（parser/ast、planner/ddl 路径）
  - 字符集/Collation 映射（parser/mysql、infoschema）
  - JSON/生成列/默认值限制（ddl/validator）
  - Sequence、System-Versioned Tables 支持边界（ddl/ast）

## TiDB 代码链接索引（补充，commit: 85389ef3740fcc5058a5660e4382e2f1e80c0f28）
- MariaDB 特有表选项（解析但忽略）：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/parser.y#L13149-L13186
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L3033-L3063
- MariaDB 特性兼容字段（IF EXISTS/IF NOT EXISTS）：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L940-L944
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L1931-L1933
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L3352-L3358
- CREATE OR REPLACE SCHEMA 不支持：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/executor.go#L289-L305
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/schematracker/dm_tracker.go#L101-L109
- MariaDB 注释前缀识别但无特殊处理：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/lexer.go#L549-L551
- Table Engine 名称白名单（含 aria/myrocks/tokudb）：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/planner/core/preprocess.go#L1476-L1516
- System_Time 分区语法：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/parser.y#L4840-L4859
- Sequence AST 支持：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/ast/ddl.go#L1819-L1849
- Check 约束校验与函数限制：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/create_table.go#L1434-L1492
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/table/constraint.go#L83-L127
- 默认值限制（TEXT/BLOB/JSON）：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/add_column.go#L1208-L1238
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/add_column.go#L855-L858
- 索引限制（JSON/Blob 前缀）：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/index.go#L244-L267
- 生成列依赖校验：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/ddl/generated_column.go#L32-L76

## go-mysql (v1.13.0) MariaDB 事件支持（用于 DM 事件适配）
- MariaDB 事件类型常量：  
  - https://github.com/go-mysql-org/go-mysql/blob/v1.13.0/replication/const.go#L104-L107
- MariaDB 事件结构体与解码：  
  - https://github.com/go-mysql-org/go-mysql/blob/v1.13.0/replication/event.go#L830-L853 (Annotate/Checkpoint)
  - https://github.com/go-mysql-org/go-mysql/blob/v1.13.0/replication/event.go#L903-L928 (GTID_LIST)

## Dumpling 在 MariaDB 上的 schema dump 行为
- Dumpling 直接使用 SHOW CREATE TABLE 输出 schema（不做 JSON/UUID/表选项改写），仅在 strict 模式补齐 Collation：  
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/sql.go#L87-L101
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/dump.go#L430-L478
  - https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/dumpling/export/dump.go#L536-L587
- DM 将 CollationCompatible 透传给 dumpling，但未引入其他 DDL 转换：  
  - https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/dumpling/dumpling.go#L322-L356

## MySQL Server (8.0) binlog 事件定义（用于与 MariaDB 差异对照）
- 事件类型定义（含 ROWS_QUERY/ PARTIAL_UPDATE/ TRANSACTION_PAYLOAD 等）：  
  - https://github.com/mysql/mysql-server/blob/666701570c392a6052341b6ddb9c21869bb1d733/libbinlogevents/include/binlog_event.h#L333-L358

## MariaDB Server (10.11) binlog 事件定义（用于 MariaDB 特有事件对照）
- MariaDB 特有事件编号（ANNOTATE_ROWS/BINLOG_CHECKPOINT/GTID_LIST）：  
  - https://github.com/MariaDB/server/blob/3218602d3100db9ce7a875511a591cddc173cc16/sql/log_event.h#L701-L723
