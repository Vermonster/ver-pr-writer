---
name: ver-pr-review
description: >
  Review pull requests in two phases: an agent phase (implementation review,
  refactor check, docs sync, issue linkage, technical position) and a human
  phase (local test guide, human judgment). Combines both into a terse final
  review submitted to GitHub, GitLab, or Bitbucket. Approval requires both
  agent and human to approve.
metadata:
  version: "1.0.0"
---

# Ver PR Review

Review PRs in two phases. Reach a grounded technical position, then guide the human reviewer to theirs. Combine both into a terse, actionable final review.

Do not rubber-stamp.
Do not write boilerplate.
Do not approve if there are unresolved blocking issues.
Do not request changes without specifying exactly what needs to change.

## Provider detection

Read `.ver-pr.yml` from the repo root. See `reference/provider-review-strategies.md` for detection logic, commands, and control file format. The detection and save behavior is identical to ver-pr-writer.

## Reference files

- `reference/provider-review-strategies.md` — per-provider commands for fetching PR context and submitting reviews

## Phase 1: Agent Review

### Step 1: Acquire the diff and PR context

Get the PR/MR diff and description. Use the provider-specific commands from `reference/provider-review-strategies.md`.

If no PR number is given, use the current branch:
```bash
git diff origin/HEAD...HEAD
git log origin/HEAD...HEAD --oneline
```

Read the PR description as stated intent — use it as context for comparison, not as ground truth. The diff is the source of truth.

### Step 2: Review the implementation

Read the diff. Identify findings and classify each as blocking or non-blocking.

**Blocking (must fix before merge):**
- Bugs or logic errors: incorrect conditions, off-by-one, missing null/error handling
- Security issues: injection, auth bypass, exposed secrets, unsafe input handling
- Correctness vs stated intent: implementation does not match the PR's stated purpose
- Missing or broken tests for changed behavior
- Public API or schema changes undocumented or breaking

**Non-blocking (suggestions worth noting):**
- Performance concerns without evidence of impact
- Style or naming that reduces clarity
- Scope creep (changes outside stated scope — flag but do not block)
- Missing test coverage for edge cases (flag if risk is real)

Do not flag formatting, whitespace, or cosmetic issues unless they obscure logic.

### Step 3: Check refactor availability

If a local `refactor` skill is available, invoke it on the files changed in the diff. Incorporate any blocking findings (e.g., dead code hiding a bug, deeply tangled logic making correctness unverifiable) into the agent position.

Do not run refactor on files outside the diff. Do not surface pure style refactor suggestions as blocking.

### Step 4: Check documentation sync

For each significant behavior change in the diff, verify:
- Inline comments or docstrings reflect the new behavior
- README, API docs, or relevant docs files were updated if behavior is externally visible
- If the repo uses specs (RSpec, Cucumber, Gherkin, OpenAPI, AsyncAPI, etc.), check that specs cover the changed behavior and are not contradicted by the implementation

Mark out-of-sync public API docs or specs as **blocking**. Mark out-of-sync internal docs as **non-blocking**.

### Step 5: Check issue/ticket linkage

Look for a linked issue or ticket in:
- The PR/MR description (GitHub `closes #N`, GitLab `closes #N`, Jira/Linear/Shortcut/Asana URLs)
- Commit messages (`git log origin/HEAD...HEAD`)
- The branch name (issue numbers, ticket IDs)

If no linkage is found, note it as a non-blocking comment: "No linked issue found — consider adding one for traceability."

### Step 6: Reach an agent technical position

Based on steps 2–5, decide:

- **Approve** — no blocking issues; implementation matches intent; docs in sync; tests adequate
- **Request changes** — one or more blocking issues found; enumerate them specifically
- **Comment** — no blocking issues, but non-trivial observations worth sharing

Record internally:
- Position: `approve` | `request_changes` | `comment`
- Blocking items (numbered list, specific and actionable — file and line where helpful)
- Non-blocking notes (brief, clearly flagged as optional)

---

## Phase 2: Human Review

### Step 7: Build the local test guide

From the actual implementation (not the PR description), write a concrete walkthrough a reviewer can follow in under 5 minutes.

**Format:**
1. Setup: checkout branch, any prerequisite (seed, env var, feature flag, dev server start command)
2. Happy path: numbered steps to observe the primary behavior change
3. Edge case(s): one or two scenarios that probe the riskiest parts of the diff
4. What to observe: specific UI state, API response field, log line, or DB value — not generic "it should work"

For UI changes: name the exact route and what to click or observe.
For API/server changes: provide a `curl` or sample request with the expected response.

Do not write "run the tests" — Phase 1 covers automated tests. This is a live behavioral walkthrough.

### Step 8: Present to human and collect feedback

Display in order:
1. **Agent position** — state it clearly (Approve / Request changes / Comment)
2. **Blocking items** — numbered list if any, each specific and actionable
3. **Non-blocking notes** — brief, clearly flagged as optional
4. **Local test guide** — from Step 7
5. **Judgment questions** — design tradeoffs, threshold choices, naming decisions the agent cannot resolve (omit if none)

Then ask:

> **Your review position:**
> - Approve
> - Request changes — (what specifically needs to change?)
> - Comment only

Wait for the human's response before proceeding.

---

## Phase 3: Final Review

### Step 9: Combine positions

| Agent | Human | Final |
|---|---|---|
| Approve | Approve | **Approve** |
| Approve | Request changes | **Request changes** |
| Approve | Comment | **Comment** |
| Request changes | Any | **Request changes** |
| Comment | Approve | **Comment** |
| Comment | Comment | **Comment** |
| Comment | Request changes | **Request changes** |

Approval requires both agent and human to approve.

### Step 10: Draft the final review text

Keep it terse. Every sentence must carry information. Do not summarize what the PR does. Do not restate the PR description. No filler.

**Approving:**
```
Approved.

[Optional: one or two inline non-blocking notes, if worth surfacing]
```

**Requesting changes:**
```
Requesting changes.

**Must fix:**
1. [Specific issue — `file.ts:line` if relevant — what needs to change and why]
2. ...

**Optional:**
- [Non-blocking suggestion]
```

**Commenting:**
```
[One focused paragraph or a short list. No position taken.]
```

If the human provided specific requested changes, incorporate them verbatim alongside the agent's blocking items. Deduplicate overlapping items.

### Step 11: Confirm and submit

Show the full proposed review to the user and ask: *"Submit this review?"*

On confirmation, submit using the provider-specific command from `reference/provider-review-strategies.md`.

---

## Safeguards

- Never approve without the human reviewer's explicit confirmation.
- Never submit a review without showing the full text first.
- Never request changes without specifying what needs to change.
- Do not comment on style, formatting, or trivial matters unless they introduce bugs.
- Do not block on non-blocking items.
