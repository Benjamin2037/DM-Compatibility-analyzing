# Government / Public Sector (archival, compliance)

## Data Characteristics
- Long retention and auditability
- Conservative change control
- Historical data access requirements

## Key Compatibility Risks
- B-03 system-versioned tables
- B-07 conditional comments
- C-02 text/blob defaults

## Recommended Configuration
- Strict mode with full audit logging
- Block or rewrite system-versioned DDL
- Record every removed clause

## Validation Checklist
- Compliance review of schema deltas
- Full data checksum
- Audit trail parity checks
