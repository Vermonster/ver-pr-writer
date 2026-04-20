# PR Signals

PRs should expose enough structure for both human and agent reviewers to answer five questions quickly:

1. What changed?
2. Why did it change?
3. Where is the risk?
4. What evidence supports correctness?
5. What review and deployment attention is actually needed?

## Signal buckets

### Intent
What problem is being solved, what outcome is expected, and what scope does the change have?

### Mental model
How should a reviewer think about the change at a system level? Where does behavior start, where does the core logic live, and what is intentionally unchanged?

### Risk
What could go wrong? What systems are affected? What are the hotspots, failure modes, and edge cases?

### Evidence
What validation supports the change? Tests, manual verification, examples, screenshots, traces, or other artifacts.

### Focus
Where should reviewers spend their attention, and what can they skim?

### Operations
What matters at merge and deploy time? Rollout, migrations, feature flags, dependency changes, monitoring, and rollback.

### Uncertainty
What assumptions, known gaps, or judgment calls remain?

## Why this matters in AI-heavy code review

AI increases PR throughput faster than human review capacity. The old model of reading every line of every PR does not scale. Better PR signals let teams shift review toward intent validation, risk management, evidence, and system integrity instead of raw diff inspection.

## Minimum required signals

Every PR should include:

- Purpose
- Outcome
- Scope
- Mental model
- Main changes
- Risk level
- Risk areas
- Blast radius
- Review hotspots
- Validation performed
- How to test locally
- Expected behavior
- Rollout or migration notes when relevant
- Assumptions and known gaps

## Optional machine-readable metadata

Teams that want stronger automation can add a structured metadata block to the PR body. This can later support routing, scoring, dashboards, or automated signal-vs-diff validation.
