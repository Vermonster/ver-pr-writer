---
name: ver-pr-writer
description: >
  Generate and validate reviewer-focused pull request descriptions using explicit
  PR signals. Analyzes diffs to surface intent, risk, evidence, and review focus
  so reviewers spend time on what matters. Two modes: draft (display only) and
  submit (create or update the PR on GitHub). Use when preparing a PR description,
  updating one from a diff, or checking whether stated signals match actual changes.
metadata:
  version: "1.1.0"
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
- If ambiguous or unclear, **always default to draft mode**. Display the description and ask: *"Would you like me to submit this to GitHub?"* Do not proceed to submit without an explicit yes.

### Draft mode (default)

Generate the PR description and display it to the user. Do not create or modify any PR on GitHub.

Always end draft output with: *"Ready to submit? Say 'submit' to create or update the PR on GitHub."*

### Submit mode

**Always confirm with the user before taking any action on GitHub.** Show the generated description and ask for explicit approval before creating or updating a PR. If the user has not confirmed, stop and wait.

Generate the PR description, then create or update the pull request on GitHub.

#### Detecting an existing PR

Before creating a new PR, check whether one already exists for the current branch:
1. Run `gh pr view --json number,title,url` in the terminal
2. If a PR exists, update its body. If no PR exists, create a new one.

#### Submitting via GitHub Pull Request tool (preferred)

If the `github-pull-request_create_pull_request` tool is available, use it to create new PRs:
- Set `title` to a concise summary derived from the Intent section
- Set `body` to the full generated PR description
- Set `head` to the current branch name (from `git branch --show-current`)
- Set `draft` to `true` unless the user explicitly requests a non-draft PR

This tool cannot update existing PRs. Fall back to `gh` CLI for updates.

#### Submitting via gh CLI (fallback or update)

Use `gh` CLI when the GitHub Pull Request tool is unavailable, or when updating an existing PR.

**Create a new PR:**
```bash
gh pr create --title "<title>" --body "<body>" --draft
```

**Update an existing PR body:**
```bash
gh pr edit --body "<body>"
```

**Update title and body:**
```bash
gh pr edit --title "<title>" --body "<body>"
```

For long PR bodies, write to a temp file to avoid shell escaping issues:
```bash
gh pr create --title "<title>" --body-file /tmp/pr-body.md --draft
gh pr edit --body-file /tmp/pr-body.md
```

#### Submit mode safeguards

- **Always confirm with the user before creating or updating a PR.** Show the full description and wait for explicit approval. Never proceed on implied consent.
- When updating, show a diff summary of what will change before applying.
- Never force-push, delete branches, or merge as part of this skill.
- If `gh auth status` fails, instruct the user to run `gh auth login` and stop.

#### Post-submit: visual evidence prompt

After successfully creating or updating a PR that includes UI changes (flagged in Step 1), remind the user:

> This PR includes UI changes. Consider adding screenshots or a short screencast to the Evidence section to help reviewers assess the visual impact. You can edit the PR body directly on GitHub or run:
> ```bash
> gh pr view --web
> ```

Do not block on this. It is a recommendation, not a requirement.

## Reference files

This skill includes companion files in the `reference/` directory:

- `reference/template.md` — the output template to follow when generating a PR description
- `reference/pr-signals.md` — team-facing explanation of what PR signals are and why they matter
- `reference/pr-template.md` — GitHub PR template teams can adopt (`.github/pull_request_template.md`)
- `reference/pr-signal-check.yml` — CI workflow that checks required sections are present

Read `reference/template.md` before generating output. Use the other files when the user asks about adopting PR signals in their project.

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

If any of these are present, flag the PR as having UI changes. This flag is used in Step 6 (evidence) and after submit.

### Step 2: Infer intent
Write:
- Purpose: the problem being solved
- Outcome: the expected result

A good purpose explains why the change exists.
A good outcome explains what becomes true after merge.

### Step 3: Build the mental model
Explain:
- where the behavior starts
- where the core logic lives
- what boundaries are crossed
- what remains intentionally unchanged

Help a reviewer understand the architecture of the change in a few sentences.

### Step 4: Separate signal from noise
Group changes into:
- main changes (behavioral or structural)
- supporting changes (refactors, test updates, docs)
- mechanical or low-signal changes (formatting, generated code, renames, lockfiles, codemods)

Do not give equal weight to all files.

### Step 5: Assess risk
Assign:
- risk level: low, medium, or high
- risk areas
- blast radius
- review hotspots
- caveats and edge cases

Use real evidence from the diff.

#### Risk heuristics

**Low risk:**
docs only, formatting only, isolated refactor with unchanged behavior and stable tests, narrow copy or presentation change, internal cleanup with no contract changes.

**Medium risk:**
targeted logic change, new validation path, bounded behavior change, non-breaking endpoint evolution, moderate refactor around live logic.

**High risk:**
auth or security changes, migrations, concurrency or caching correctness, config or deployment changes, public contract changes, multi-system coordination, broad deletion or rewrite of core logic.

### Step 6: Extract evidence
Document:
- **Tests in this PR:** what test files changed, what cases were added or modified, and what behavior they assert
- **How to run those tests:** exact commands scoped to the changed files (e.g., `npm test path/to/spec`, `npm run test:int -- grep "feature name"`); label any prerequisites (db running, seed data, feature flag)
- what validation was performed beyond automated tests (manual checks, QA, exploratory)
- expected reviewer-observable behavior
- useful artifacts: screenshots, sample requests, logs, before/after behavior

Test execution and behavioral demo are different. This step covers the test suite. Step 7 covers the live walkthrough.

If the PR includes UI changes (from Step 1), note in the Evidence section that screenshots or a short screencast would strengthen the PR. Add a placeholder like:

> **Visual evidence needed:** This PR includes UI changes. Add before/after screenshots or a short screencast to this section after the PR is created.

### Step 7: Target review attention
Identify:
- best review order (fewest files, most signal first)
- files or modules to focus on
- changes that can be skimmed
- explicit questions for reviewer judgment

The goal is to reduce review effort while increasing review quality.

#### How to demo locally

A demo is a live behavioral walkthrough — not a test run. It lets reviewers observe the change working end-to-end in a real environment. It is separate from Step 6's test commands, which exercise the automated suite.

For every PR, write a concrete demo walkthrough that lets a reviewer reproduce and experience the change in under 5 minutes. Frame the demo around the mental model and purpose — not just the mechanics.

**Format:**
1. One sentence that anchors the demo to the PR's purpose ("This demo exercises the new X behavior introduced to solve Y")
2. Any prerequisite setup (seed data, feature flags, env vars, dev server) — distinct from test fixtures
3. Numbered UI steps or `curl`/HTTP examples that trigger the changed behavior
4. What to observe — specific UI state, API response field, log line, or DB value that confirms the change worked
5. (Optional) A negative case: a scenario that should now be rejected or behave differently — framed as what a user would see, not a test assertion

**When to include UI steps:** whenever the diff touches components, pages, forms, or navigation. Name the exact route and describe what to click or observe.

**When to include API examples:** whenever the diff touches server actions, API routes, or service logic with an external surface. Provide a `curl` command or sample request body.

**When both apply:** include both, UI first.

Do not write a generic "run `npm run dev`" placeholder. If the change is too abstract to demo, explain what integration test or log confirms the behavior instead.

### Step 8: Surface uncertainty
State:
- assumptions
- known gaps
- incomplete validation
- places needing human judgment

Do not hide uncertainty.

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

Use the template from `reference/template.md`. Produce markdown with these sections:

1. Intent
2. Mental model
3. What changed
4. Risk signals
5. Evidence / validation
6. Review targeting
7. Operations
8. Uncertainty / judgment calls

Use concise markdown with real file, module, endpoint, or workflow names.

## Style guidance

**Prefer plain English.** Use language a busy reviewer can scan quickly.
- Good: "prevents invalid requests from reaching downstream processing"
- Bad: "introduces pre-persistence input quality enforcement semantics"

**Be concrete.** Name files, modules, endpoints, commands, workflows, flags, and migrations.

**Be compact but dense.** Short to medium length, high information density.

**Avoid these failure modes:**
- narrating every changed file
- saying "misc fixes" or "minor cleanup" when behavior changed
- claiming improved performance without evidence
- claiming tests passed unless known
- claiming safety without naming risk
- using filler that could apply to any PR

## Final checks

Before returning output, confirm the result:
- clearly states the purpose
- gives a useful mental model
- separates major changes from noise
- identifies real risk
- includes realistic validation guidance
- targets reviewer attention
- covers operational concerns when relevant
- surfaces uncertainty honestly
- is grounded in the actual diff
