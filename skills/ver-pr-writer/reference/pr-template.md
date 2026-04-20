# PR signals

## Intent

### Change type
Select one or more:
- [ ] Bug fix
- [ ] Feature
- [ ] Refactor
- [ ] Cleanup
- [ ] Test-only
- [ ] Docs-only
- [ ] CI / build / tooling
- [ ] Infrastructure / deployment

### Purpose
Describe the problem this PR solves in plain English.

### Outcome
Describe the expected user-visible or system-visible result.

### Scope
Select one:
- [ ] Narrow / local
- [ ] Medium / bounded subsystem
- [ ] Cross-cutting / architectural

### Linked context
Reference issue, ticket, design note, or other source of truth.

---

## Mental model

### How to think about this change
Explain the change at a high level.
- Where does the behavior begin?
- Where does the key logic live?
- What is intentionally unchanged?

### Main flow / affected boundaries
List the most important modules, services, entry points, or boundaries involved.
- 
- 
- 

### Non-goals
Call out related things that were intentionally not changed.
- 
- 
- 

---

## What changed

### Main changes
List the meaningful behavioral or structural changes.
- 
- 
- 

### Supporting changes
List secondary changes like refactors, test updates, docs, or cleanup.
- 
- 
- 

### Noise / mechanical changes
Call out changes reviewers can skim.
- [ ] Formatting only
- [ ] Renames / moves
- [ ] Generated files
- [ ] Lockfile / dependency metadata
- [ ] Other: 

---

## Risk signals

### Risk level
Select one:
- [ ] Low
- [ ] Medium
- [ ] High

### Risk areas
Check all that apply:
- [ ] Business logic
- [ ] Data model
- [ ] Migration
- [ ] API contract
- [ ] UI behavior
- [ ] Performance
- [ ] Auth / security
- [ ] Concurrency / async behavior
- [ ] Infrastructure / deployment
- [ ] Observability / logging
- [ ] Backward compatibility
- [ ] External integrations
- [ ] Other: 

### Blast radius
Describe what systems, users, or workflows could be affected.

### Key review hotspots
Point reviewers to the places that deserve the most attention.
- 
- 
- 

### Edge cases / failure modes / caveats
Describe subtle scenarios, tradeoffs, or known limitations.
- 
- 
- 

---

## Evidence / validation

### Validation performed
Check all that apply:
- [ ] Unit tests added or updated
- [ ] Integration tests added or updated
- [ ] End-to-end tests added or updated
- [ ] Manual verification performed
- [ ] Lint / static analysis / typecheck passed
- [ ] Existing test suite still passes
- [ ] Benchmark / perf check performed
- [ ] Not yet fully validated

### What was tested
Describe what you actually validated.

### How to test locally
Provide exact steps when possible.

```bash
# example
pnpm install
pnpm test
pnpm dev
```

1. 
2. 
3. 

### Expected behavior
Describe what a reviewer should observe.

### Evidence artifacts
Add screenshots, sample requests/responses, logs, traces, before/after examples, or links as needed.

---

## Review targeting

### Best way to review this PR
Suggest the shortest useful review order.

1. 
2. 
3. 

### Suggested review focus
What matters most in this PR?
- 
- 
- 

### Questions for reviewers
What specifically would you like feedback on?
- 
- 

---

## Operations

### Migration / rollout notes
Describe sequencing, feature flags, schema changes, config changes, or deployment concerns.

### Monitoring / rollback
How would you detect issues, and how would you revert or mitigate them?

### Dependency / contract changes
Note dependency updates, public API changes, schema changes, or downstream impacts.

---

## Uncertainty / judgment calls

### Assumptions
List important assumptions this PR relies on.
- 
- 

### Known gaps
What is not fully validated or still incomplete?
- 
- 

### Human judgment needed
Call out areas where reviewer judgment matters more than mechanical correctness.
- 
- 

---

## Checklist

- [ ] Purpose is stated in plain English
- [ ] Mental model is included
- [ ] Main changes are separated from noise
- [ ] Risk level and risk areas are identified
- [ ] Review hotspots are named
- [ ] Testing / validation steps are included
- [ ] Rollout or migration notes are included if relevant
- [ ] Non-goals and caveats are explicit
- [ ] No unrelated changes are mixed in
