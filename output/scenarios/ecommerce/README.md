# E-commerce (orders, catalog, inventory)

## Data Characteristics
- Wide tables and frequent online DDL
- High write bursts
- Multilingual data and long indexes

## Key Compatibility Risks
- C-01 JSON/CHECK differences
- C-02 text/blob defaults
- D-02 index length limits
- E-01 collation incompatibilities

## Recommended Configuration
- Enable index_prefix and keylength rules
- Apply collation mapping
- Balanced mode: strict for DML, loose for DDL
- Strip unsupported table options

## Validation Checklist
- Transformed DDL review
- Index length audit
- Peak write lag tests
