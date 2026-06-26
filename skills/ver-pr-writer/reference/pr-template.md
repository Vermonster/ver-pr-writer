# <!-- PR title: concise imperative phrase -->

## Intent

**Purpose:** Describe the problem this PR solves.
**Outcome:** Describe what becomes true after merge.
**Type:** Bug fix | Feature | Refactor | Cleanup | Infra | Docs | Test-only — **Risk:** Low | Medium | High — **Scope:** Narrow | Bounded | Cross-cutting
**Linked:** Issue / ticket / design doc URL

---

## Technical

**What changed**
- `path/to/file.ts` — what it does now
- `path/to/file.test.ts` — what cases were added or changed

**Risk**
- Risk level and reason
- Key hotspots or edge cases

**Test it**
```bash
# exact command scoped to changed files
```
Manual steps if needed; note what to observe.

<!-- If operations notes apply (migrations, config, rollout, rollback), add:
**Operations**
- ...
-->

---

## Human Review

**Start here:** `path/to/key-file.ts` — why it's the most signal-dense file.
**Skim:** low-signal files (formatting, generated, lockfile).
**Non-goals:** what was intentionally left unchanged.

**Demo / spot check**
1. Start: `...`
2. Navigate to: `...`
3. Do: click / fill / submit / call
4. Expect: what you should see or what should not appear

<!-- For UI changes, list each affected route and the specific action + expected result:
- `/path/to/page` — click X → expect Y
- `/path/to/other` — submit form with Z → expect validation error / success state

For API changes, provide a curl example:
curl -X POST ...
-->

**Questions for reviewers**
- ...

**Uncertainty**
- Assumptions, known gaps, or areas where reviewer judgment matters more than correctness.

