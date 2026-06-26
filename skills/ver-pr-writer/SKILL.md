---
name: ver-pr-writer
description: >
  Generate and validate reviewer-focused pull request descriptions using explicit
  PR signals. Analyzes diffs to surface intent, risk, evidence, and review focus
  so reviewers spend time on what matters. Two modes: draft (display only) and
  submit (create or update the PR on GitHub, GitLab, or Bitbucket). Use when
  preparing a PR description, updating one from a diff, or checking whether
  stated signals match actual changes.
metadata:
  version: "1.2.0"
---

# Ver PR Writer

Write PR descriptions that expose high-signal review metadata.

A good PR description helps both humans and agents answer: what changed, why, where the risk is, how it was validated, what deserves attention, and what still needs judgment.

Do not narrate the diff line-by-line.
Do not write vague AI-style summaries.
Do not invent intent, evidence, or risk mitigation.

## Modes

**Draft mode is the default. When in doubt, draft.**

This skill has two modes. Determine which mode to use from context:

- If the user says "draft", "generate", "prepare", or "write" a PR description, use **draft mode**.
- If the user says "submit", "create", "open", "push", or "update" a PR, use **submit mode**.
- If ambiguous or unclear, **always default to draft mode**. Display the description and ask: *"Would you like me to submit this?"* Do not proceed to submit without an explicit yes.

### Draft mode (default)

Generate the PR description and display it to the user. Do not create or modify any PR.

Always end draft output with: *"Ready to submit? Say 'submit' to create or update the PR."*

### Provider detection

Before any submit action, resolve the git provider. Read `reference/provider-strategies.md` for the full control file format, auto-detection logic, and per-provider commands.

**Resolution order:**
1. Read `.ver-pr-writer.yml` from the repo root. If `provider` is set, use it.
2. If the file is absent or `provider` is missing, run `git remote get-url origin` and match against known hostnames (github.com → `github`, gitlab.com → `gitlab`, bitbucket.org → `bitbucket`).
3. If auto-detection succeeds, confirm with the user: *"Detected `<provider>` from your remote. Save this to `.ver-pr-writer.yml`?"* If yes, write the file.
4. If auto-detection is ambiguous, ask: *"Which provider are you using?"* (GitHub / GitLab / Bitbucket), then write `.ver-pr-writer.yml`.

After writing, inform the user: *"Saved provider as `<provider>` to `.ver-pr-writer.yml`. Commit this file to share it with your team."*

### Submit mode

**Always confirm with the user before taking any action on the remote.** Show the generated description and ask for explicit approval before creating or updating a PR. If the user has not confirmed, stop and wait.

Resolve the provider (see above), then follow the provider-specific strategy in `reference/provider-strategies.md`.

#### Detecting an existing PR

Check whether a PR already exists for the current branch using the provider-specific command from `reference/provider-strategies.md`. If a PR exists, update its body. If not, create a new one.

#### Submit mode safeguards

- **Always confirm with the user before creating or updating a PR.** Show the full description and wait for explicit approval. Never proceed on implied consent.
- When updating, show a diff summary of what will change before applying.
- Never force-push, delete branches, or merge as part of this skill.
- If auth fails for the detected provider, instruct the user to authenticate and stop.

#### Post-submit: visual evidence prompt

After successfully creating or updating a PR that includes UI changes (flagged in Step 1), remind the user:

> This PR includes UI changes. Consider adding screenshots or a short screencast to the Human Review section to help reviewers assess the visual impact.

Do not block on this. It is a recommendation, not a requirement.

## Reference files

This skill includes companion files in the `reference/` directory:

- `reference/template.md` — the output template to follow when generating a PR description
- `reference/provider-strategies.md` — per-provider submit strategies (GitHub, GitLab, Bitbucket), control file format, and auto-detection logic
- `reference/pr-signals.md` — team-facing explanation of what PR signals are and why they matter
- `reference/pr-template.md` — PR template teams can adopt
- `reference/pr-signal-check.yml` — CI workflow that checks required sections are present

Read `reference/template.md` before generating output. Read `reference/provider-strategies.md` before any submit action. Use the other files when the user asks about adopting PR signals in their project.

## Agent responsibilities

Do two jobs every time (both modes):

### Job 1: Generate the PR description
Infer and write the strongest grounded PR description from the available evidence, following the output template.

### Job 2: Validate the PR signals
Compare the claimed signals against the actual diff. Note mismatches internally and fix them before finalizing.

Mismatch examples:
- Claims low risk but changes auth and persistence
- Claims refactor only but behavior changes
- Claims validation performed but no relevant tests changed
- Claims narrow scope but touches multiple subsystems
- Omits rollout concerns when config or migrations changed

If author-provided signals are weak, incomplete, or misleading, improve them from the code. Be explicit and grounded.

## Inputs to inspect

Before drafting, inspect as many of these as are available:
- git diff (primary source of truth)
- changed file list
- commit messages
- branch name
- test files
- migration files
- schema or API changes
- config or deployment changes
- docs or README updates
- linked issue or ticket context
- CI changes

## Workflow

### Step 1: Classify the change
Determine:
- change type: bug fix, feature, refactor, cleanup, infra, docs, test-only
- scope: narrow, bounded subsystem, or cross-cutting
- external vs internal behavior change
- whether the change includes UI updates (see UI change detection below)

Do not rely on branch name alone.

#### UI change detection

Scan the diff for signals that the change affects user-visible interface:
- changes to templates, views, components, layouts, or pages
- changes to CSS, SCSS, Tailwind classes, or style files
- changes to frontend assets (images, icons, fonts)
- changes to form fields, buttons, modals, or navigation
- changes to error messages, labels, or user-facing copy
- changes to email templates or PDF rendering

If any of these are present, flag the PR as having UI changes. This flag is used in the Technical section (test it / evidence) and after submit.

### Step 2: Draft the Intent header
Write a single compact block:
- Title: a concise imperative phrase (e.g. "Extract shared input validator", "Add rate limiting to search API") — used as both the H1 and the PR title
- Purpose: one sentence on the problem being solved
- Outcome: one sentence on what becomes true after merge
- Type / Risk / Scope on one line
- Linked context if any

### Step 3: Build the Technical section

**What changed:** List only signal-bearing files with one-line descriptions. Group test files together. Do not list generated, formatting, or lockfile changes — call them out in Human Review as "skim".

**Risk:** Assign low / medium / high with a short rationale. Name specific hotspots and edge cases. Use the risk heuristics below.

#### Risk heuristics

**Low:** docs only, formatting only, isolated refactor with unchanged behavior and stable tests, narrow copy or presentation change, internal cleanup with no contract changes.

**Medium:** targeted logic change, new validation path, bounded behavior change, non-breaking endpoint evolution, moderate refactor around live logic.

**High:** auth or security changes, migrations, concurrency or caching correctness, config or deployment changes, public contract changes, multi-system coordination, broad deletion or rewrite of core logic.

**Test it:** Provide exact commands scoped to changed files (e.g., `npm test path/to/spec`). Label prerequisites (db running, seed data, feature flag). Add manual steps only when automated tests don't cover the observable behavior. For UI changes, add a placeholder for screenshots/screencast.

**Operations** (include only if relevant): migration sequencing, feature flags, rollback plan, contract or dependency changes.

### Step 4: Build the Human Review section

**Start here:** Name the single most signal-dense file and say why in one sentence.

**Skim:** Name low-signal files (formatting, generated, lockfiles, mechanical renames).

**Non-goals:** Call out what was intentionally not changed.

**Demo / spot check:** Write a concrete walkthrough a reviewer can follow in under 5 minutes to observe the change working end-to-end. This is separate from the automated test commands in Technical.

Format:
1. Start command (dev server, seed, feature flag if needed)
2. Navigate to the exact route or trigger point
3. Action: what to click, fill, submit, or call
4. Observe: specific UI state, response field, log line, or DB value that confirms it worked
5. (Optional) Negative case: what should now be rejected or look different

For UI changes, list every affected route/component as its own item:
- `/path/to/page` — action → expected result
- `/path/to/other` — action → expected result

For API/server changes, provide a `curl` or sample request with the expected response.

Do not write a generic "run `npm run dev`" placeholder. If the change is too abstract to demo, name the integration test or log line that confirms the behavior.

**Questions for reviewers:** Ask only real questions — design tradeoffs, threshold choices, naming decisions — that need human judgment. Skip if there are none.

**Uncertainty:** State assumptions, known gaps, and places where reviewer judgment matters more than correctness. Do not hide uncertainty.


### Step 9: Validate before finalizing

Check the PR description against the diff for consistency:

**Intent checks:**
- Does the stated purpose match the changed code?
- Is the claimed scope believable?
- Are non-goals consistent with untouched areas?

**Risk checks:**
- Does the risk level match the systems touched?
- Are key risk areas missing?
- Are review hotspots pointing to the real complexity?

**Evidence checks:**
- If behavior changed, is there evidence of validation?
- If API or schema changed, are examples or contract notes included?
- If migration or config changed, are rollout notes present?

**Focus checks:**
- Are reviewers told where to spend attention?
- Are generated or mechanical areas de-emphasized?

**Operations checks:**
- If deploy or runtime behavior may change, are monitoring and rollback addressed?

If a mismatch exists, revise the PR description to reflect the code truthfully.

## Output

Use the template from `reference/template.md`. Produce markdown with three top-level blocks:

1. **Intent** — one compact header: purpose, outcome, type/risk/scope on one line, linked context
2. **Technical** — what changed (named files with one-line descriptions), risk (level + hotspots + edge cases), how to test (exact commands + manual steps), and operations notes if relevant (migrations, rollout, rollback)
3. **Human Review** — where to start, what to skim, non-goals, a concrete demo/spot-check walkthrough, explicit questions for reviewers, and uncertainty/judgment calls

Use concise markdown with real file, module, endpoint, or workflow names. Omit any subsection that has nothing meaningful to say. Do not repeat information between sections.

## Style guidance

**Terse over verbose.** Each bullet or sentence should carry new information. Cut any sentence a reviewer would skip.
- Good: "prevents invalid requests from reaching downstream processing"
- Bad: "introduces pre-persistence input quality enforcement semantics"

**Prefer plain English.** Use language a busy reviewer can scan in under 10 seconds per section.

**Be concrete.** Name files, modules, endpoints, commands, workflows, flags, and migrations.

**Two-section discipline.**
- **Technical** answers: what changed, what could break, how to verify with automated tests.
- **Human Review** answers: where to look, how to demo it locally, what to judge, what's uncertain.
- Do not blur the boundary. Risk facts go in Technical. Reviewer guidance and demo steps go in Human Review.

**Avoid these failure modes:**
- narrating every changed file — name only the signal-bearing ones
- saying "misc fixes" or "minor cleanup" when behavior changed
- claiming improved performance without evidence
- claiming tests passed unless known
- claiming safety without naming risk
- using filler that could apply to any PR

## Final checks

Before returning output, confirm the result:
- clearly states the purpose
- separates major changes from noise
- identifies real risk
- includes automated test commands scoped to changed files
- includes a concrete demo walkthrough (not a generic placeholder)
- targets reviewer attention to the right files
- covers operational concerns when relevant
- surfaces uncertainty honestly
- uses no emoji
- is grounded in the actual diff
