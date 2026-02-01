# MariaDB conditional comments (/*M...*/)

Issue ID: B-07

## Summary
TiDB treats MariaDB conditional comments as regular comments and does not execute their contents.

## Impact on DM
Conditional SQL can be silently dropped, leading to incomplete schema.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/pkg/parser/common.go#L19-L45

## Evidence / Downstream Behavior
- https://github.com/pingcap/tidb/blob/85389ef3740fcc5058a5660e4382e2f1e80c0f28/pkg/parser/lexer.go#L549-L551

## Primary Use Cases
- Multi-version SQL compatibility

## Recommended Handling
- Preprocess conditional comments and execute relevant statements.
- Log and warn when conditional SQL is ignored.
