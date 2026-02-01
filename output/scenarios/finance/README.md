# Finance (payments, core banking, risk control)

## Data Characteristics
- Strict ACID and strong audit requirements
- Low tolerance for schema drift
- High availability and fast failover

## Key Compatibility Risks
- A-01 GTID parsing errors
- A-02 GTID event mapping
- C-03 CHECK function limits
- B-03 system-versioned DDL

## Recommended Configuration
- Use strict transformation mode
- Enable JSON/CHECK/function default rules
- Enforce collation mapping
- Explicitly handle MariaDB GTID events

## Validation Checklist
- End-to-end checksum comparison
- DDL replay audit with transform logs
- Binlog continuity tests
