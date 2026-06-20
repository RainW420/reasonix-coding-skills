# Paper Checkpoint Strategy

Checkpoints control when the system may continue without user approval.

## Four Levels

### Forced Checkpoints

Never auto-advance:

- research question
- target journal
- paper type
- outline structure
- core claims
- final submission package
- any attempt to proceed with critical review findings
- any threshold breach for `MATERIAL GAP` or `NEEDS CONFIRMATION`

### Default Checkpoints

Show the result and continue only after the user approves or says "continue":

- each major section completion
- full review report
- major revision plan
- full manuscript verification report

### Automatic Steps

Run without interrupting, but log actions in notes or `revision_history`:

- sentence polishing
- citation formatting
- figure export format adjustments
- `paper-verify` lite mode passing
- non-critical citation supplementation

### Autopilot Mode

Only enabled by explicit user instruction. It may collapse section checkpoints into every 2-3 sections, but forced checkpoints remain unchanged.

## Thresholds

Escalate to a forced checkpoint when any of these is true:

- critical open `known_gaps`
- unsupported core claim
- fabricated or missing citation risk
- data source unclear
- figure conclusion not tied to supplied data
- review finding severity is critical

## Failure Consequences

- Lite verification failure blocks the next section.
- Full verification failure blocks finalization.
- Critical review finding blocks `paper-finalize`, but not `paper-response` or revision.
- Provenance threshold breach blocks auto-advance.

