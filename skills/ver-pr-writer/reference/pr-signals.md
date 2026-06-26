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

### Technical
What files changed and how? What is the risk level, hotspots, and edge cases? What automated tests cover this, and how do you run them? Are there migration or rollout concerns?

### Human Review
Where should reviewers start and what can they skim? How do you demo or spot-check the change locally? What explicit questions need human judgment? What assumptions or gaps remain?

## Why this matters in AI-heavy code review

AI increases PR throughput faster than human review capacity. The old model of reading every line of every PR does not scale. Better PR signals let teams shift review toward intent validation, risk management, evidence, and system integrity instead of raw diff inspection.

## Minimum required signals

Every PR should include:

- Purpose and outcome
- Scope
- Key changed files with one-line descriptions
- Risk level and hotspots
- Test commands scoped to the change
- Demo / spot-check walkthrough
- Assumptions and known gaps

## Optional machine-readable metadata

Teams that want stronger automation can add a structured metadata block to the PR body. This can later support routing, scoring, dashboards, or automated signal-vs-diff validation.
