## 新功能说明
我需要你仔细评估一下 DM 与 MySQL 和 MariaDB 的兼容性 gap，分别给出详细的对比兼容性问题清单，并且给出设计文档，说明如何解决这些兼容性问题，确保 DM 可以无缝的支持 MySQL 和 MariaDB 作为上游数据库。
DM 代码路径：/Users/benjamin2037/Desktop/workspace/sourcecode/tiflow
本项目路径：/Users/benjamin2037/Desktop/workspace/sourcecode/project/DMCompatibility
MariaDB to TiDB schema transformer 代码路径请参考开源代码路径，自行分析；如有需要，可以将对应的差异文件，下载到项目路径下面的 codeDiff 目录下分别建立 mysql 和 mariadb 子目录进行对比分析。
tidb 的代码路径： /Users/benjamin2037/Desktop/workspace/sourcecode/tidb

下面有一些输入：
GitHub - developer-Bushido/mariadb2tidb: MariaDB → TiDB schema transformer (Go)
https://www.linkedin.com/posts/akhamidov_tidb-mariadb-databasemigration-activity-7421898945456431105-QNBT
张建伟（Brian.Zhang）
昨天 12:29
回复 唐刘: 
GitHub - developer-Bushido/mariadb2tidb: MariaDB → TiDB schema transformer (Go)
@唐刘 i just shared the exact the same post to my teams a couple of days ago. And we are studying that. cc @Bear. C @Oliver(吕夫洋)
Oliver（吕夫洋）
昨天 12:43
回复 张建伟（Brian.Zhang）: 
@唐刘 i just shared the exact the same post to my teams a couple of days ago. And we are studying that. cc @Bear. C @Oliver(吕夫洋)
@张建伟(Brian.Zhang)Yes. I took a look at the open-source tool earlier this week. It’s interesting and could help mitigate the schema incompatibilities we run into in MariaDB → TiDB migrations. Some cases (like those described in the original post) likely won’t be handled out of the box, so users will usually need targeted adjustments to make it work for our schemas — which is also one of the current blockers for MariaDB migration.

We could consider integrating this into the DM workflow between the dump and load phases to reduce friction in the migration process. We will loop in PM @Airton Lastori for further discussion and evaluation of this idea.

## 我们的标准：
- 请你详细阅读代码， 开源 mysql 和 mariadb 的文档和代码，然后给出一个设计文档，然后基于充分阅读代码与理解了 mariadb2tidb 的相关代码，给出 dm 如何无缝支持 mariadb 作为上游数据库的设计；
- 请根据这些 mysql 和 mariabd 的差异点，帮我分析一下这些功能，哪些是 mysql 社区最受欢迎的功能，并且广泛被哪些类型的客户使用在什么场景，例如 fintech，web3，AI，电商等等领域；
- 请给出一个非常详细的兼容性 gap 清单，说明 mariadb 和 mysql 之间的差异点，以及这些差异点如何影响 dm 的使用；
- 包括我们需要在 dm 代码中做哪些改动来支持这些差异点，以及 mysql 和 mariadb 之间的差异点如何影响 dm 的使用；
- 每一条差异点都必须有对应的代码行号链接（GitHub URL），
- 所有的中间步骤请将 context 做比较的保存在 ./context 中，并且确保你可以下次从这里恢复之前的分析，并可以在那之上继续工作；
- 所有的输出请保存到 ./output 目录中；
- 请使用 ToDo.md 文档，追加你分析中产生的子任务，并跟据实际情况分析如何继续；
- 验收标准是输出一份完全符合我上述需求的详细设计文档，并良好的拆分出子模块设计，说明清楚，

## 重要指示
- 你有无限的上下文额度，仔细深入分析所有代码 tidb x 的 dxf 代码，请务必仔细阅读每一行代码； 
- 所有中间 /Users/benjamin2037/Desktop/workspace/sourcecode/project/DMCompatibility/context/analyze.md；
- 最终结论输出到 result.md 不能再出现 ToDo 或者建议项，这些都输出到 analyze.md 并自动循环分析，并逐步收敛；
- 你必须反复推演，直到得出最有说服力的设计文档；
- 用中文输出所有内容 
