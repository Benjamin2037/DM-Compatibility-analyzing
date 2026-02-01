# Limited charset conversion in DML

Issue ID: E-02

## Summary
DM only applies explicit conversions for latin1 and gbk; other charsets rely on downstream compatibility.

## Impact on DM
Character corruption risk for legacy or uncommon encodings.

## DM Touchpoints
- https://github.com/pingcap/tiflow/blob/142713c45bf390219f102cb7bd7b4dedc6be6e6f/dm/syncer/dml.go#L50-L127

## Evidence / Downstream Behavior
- None

## Primary Use Cases
- Legacy migrations
- Historical data consolidation

## Recommended Handling
- Expand charset conversion coverage or enforce a charset policy.
- Validate with encoding-specific sampling tests.
