# AI / ML Systems (feature store, metadata)

## Data Characteristics
- Semi-structured features
- Frequent schema evolution
- Large payload columns

## Key Compatibility Risks
- C-01 JSON/CHECK constraints
- C-04 generated columns
- D-02 index length issues

## Recommended Configuration
- Loose mode for DDL with warnings
- Strict mode for DML
- Enable JSON/default/generate-column rules

## Validation Checklist
- Feature distribution sampling
- Schema-change replay tests
- Lag and throughput monitoring
