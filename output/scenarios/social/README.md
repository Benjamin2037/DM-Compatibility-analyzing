# Social Platforms (user content, messaging)

## Data Characteristics
- Very high concurrency
- utf8mb4 and emoji heavy
- Large secondary indexes

## Key Compatibility Risks
- E-01 collation mapping gaps
- D-01 JSON/text index limits
- D-02 index length limits

## Recommended Configuration
- Apply collation mapping and index prefix rules
- Drop or rewrite JSON indexes
- Keep Tracker and downstream DDL aligned

## Validation Checklist
- Encoding integrity checks
- Search/index parity sampling
- Latency under peak load
