# ToDo

- [x] 补充 Go-MySQL 对 MariaDB 特有 binlog 事件（如 ANNOTATE_ROWS_EVENT、BINLOG_CHECKPOINT_EVENT 等）的支持状况调研，并映射到 DM 事件处理路径。
- [x] 核对 TiDB 对 MariaDB 特性（System-Versioned Tables、Sequences、UUID/JSON 等）的支持/限制清单，补充到兼容性矩阵。
- [x] 核验 dumpling 在 MariaDB 上的 schema dump 输出差异（JSON/UUID/字符集等）与 DM loader 的衔接策略。
- [x] 完成 DM 端“Schema Transformer”模块的接口定义与测试矩阵（全量/增量 DDL、规则可配、回滚与失败策略）。
- [x] 从 TiDB 源码抽取与 MariaDB 兼容性相关的实现/限制（parser、ddl、infoschema），并补充到差异清单与设计方案。

## 全量对比任务（本轮新增）
- [x] 梳理 TiDB parser 对 MariaDB 语法/对象支持范围（System-Versioned、Sequence、UUID 类型、CHECK/Generated Column、表选项/引擎）。
- [x] 梳理 TiDB DDL 校验与限制点（默认值/函数、JSON/文本类型、索引前缀与长度、NO PAD/Collation）。
- [x] 梳理 TiDB 字符集/Collation 映射与默认行为（与 MariaDB/MySQL 差异）。
- [x] 梳理 TiDB 对 MariaDB 额外数据类型/函数支持情况（如 UUID/JSON、虚拟列、系统版本表）。
- [x] 梳理 DM 与 TiDB 的接口衔接点（DDL 解析、Schema Tracker、DML 转换、dumpling/loader），并标注需新增的适配层。
- [x] 输出“差异分层报告”：按复制/DDL/类型/Collation/限制/工具链分类，附用户主要应用场景分析。

## 全量对比补充任务（本轮执行）
- [x] 收集 TiDB 对 MariaDB 特有表选项/引擎/注释的解析与忽略行为（含行号链接）。
- [x] 收集 TiDB 对 CREATE/ALTER/INDEX 的 MariaDB 兼容字段（IF EXISTS/IF NOT EXISTS）与限制（含行号链接）。
- [x] 收集 TiDB DDL 限制点（JSON/Blob 默认值、Check 约束限制、索引前缀、生成列依赖）并映射到 DM 处理路径。
- [x] 核对 DM Schema Tracker 对 Sequence/系统版本表等 MariaDB DDL 的覆盖度，补充差异项与影响。
- [x] 更新 analyze.md（新增代码链接与差异备注），同步更新 result.md（分层差异 + 场景）。

## 进一步验证任务（本轮新增，立即执行）
- [x] 调研 go-mysql 对 MariaDB 特有 binlog 事件的覆盖与解析路径，并补充 DM 事件适配建议与行号链接。
- [x] 核验 dumpling 在 MariaDB 上的 schema dump 输出差异点（JSON/UUID/字符集/表选项），形成与 loader 的衔接策略说明。
- [x] 补齐 DM Schema Transformer 模块的接口定义、接入点与测试矩阵（全量/增量/回滚/失败策略），并更新设计描述。

## 源码下载与对照（本轮新增）
- [x] 下载 go-mysql v1.13.0 源码并记录 commit（用于 MariaDB 事件解析对照）。
- [x] 下载 MySQL Server 8.0 源码并稀疏拉取 binlog/DDL 相关目录（用于事件类型对照）。
- [x] 下载 MariaDB Server 10.11 源码并稀疏拉取 binlog/DDL 相关目录（用于差异对照）。
