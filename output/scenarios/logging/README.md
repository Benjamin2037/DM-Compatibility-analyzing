# Logging / Observability Platforms

## Data Characteristics
- Append-heavy writes
- Huge tables with wide text columns
- High throughput ingestion

## Key Compatibility Risks
- C-02 text/blob defaults
- D-02 index prefix/length limits
- B-04 table options/engines

## Recommended Configuration
- Loose mode with explicit rule logging
- Strip unsupported table options
- Tune relay and apply throughput

## Validation Checklist
- Ingestion throughput tests
- End-to-end lag monitoring
- Sampling-based data integrity checks
