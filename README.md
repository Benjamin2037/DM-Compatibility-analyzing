# DM-Compatibility-analyzing

This repo captures the ongoing compatibility analysis for DM with MySQL and MariaDB, including code references, gap list, and design notes.

## Structure
- `DMC.md`: original requirements and constraints
- `context/`: working notes and code link index (`analyze.md`)
- `output/`: final report (`result.md`)
- `ToDo.md`: task tracking
- `codeDiff/`: upstream references (as submodules)

## Submodules
- `codeDiff/mariadb/mariadb2tidb`: MariaDB → TiDB schema transformer
- `codeDiff/mariadb/go-mysql`: binlog parsing library used by DM
- `codeDiff/mysql/mysql-server`: MySQL Server 8.0 (sparse checkout)
- `codeDiff/mariadb/mariadb-server`: MariaDB Server 10.11 (sparse checkout)

## Notes
- The analysis is written in Chinese as required.
- `output/result.md` contains the finalized layered differences and design summary.
