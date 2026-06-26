# ver-pr-writer + ver-pr-review

Two agent skills for the full PR lifecycle: writing signal-rich PR descriptions and reviewing PRs with grounded technical judgment.

Helps both humans and agents answer: what changed, why, where the risk is, how it was validated, what deserves attention, and what still needs judgment.

## Why PR signals?

If your team is using AI coding agents, you've probably noticed something: **the bottleneck isn't writing code anymore. It's reviewing it**. See [MOTIVATION.md](MOTIVATION.md) for the full analysis of why structured PR signals matter in agentic SDLC workflows.

## Install

Both skills are installed from the same repository:

```bash
npx skills add Vermonster/ver-pr-skills
```

Or install to a specific agent:

```bash
npx skills add Vermonster/ver-pr-skills -a github-copilot
```

## ver-pr-writer

When activated, the skill:

1. Inspects the git diff, changed files, commit messages, tests, and other available context
2. Classifies the change type, scope, and risk
3. Generates a structured PR description with three sections: **Intent** (purpose, outcome, type/risk/scope), **Technical** (changed files, risk, test commands, operations), and **Human Review** (where to start, demo walkthrough, questions, uncertainty)
4. Validates the generated signals against the actual diff and fixes mismatches

### Usage

#### Draft mode (default)

Generates the PR description and displays it without touching your remote. Use this to review and iterate before submitting.

> "Write a PR description for this branch"
>
> "Draft PR signals for my changes"
>
> "Generate a PR description"

#### Submit mode

Generates the PR description, then creates or updates the pull request on your remote (GitHub, GitLab, or Bitbucket). The skill checks for an existing PR on the current branch and either creates a new one or updates the body.

> "Submit a PR for this branch"
>
> "Create a PR with signals"
>
> "Update my PR description"

Submit mode uses the GitHub Pull Request tool when available, and falls back to `gh` CLI. New PRs are created as drafts by default.

The skill will always confirm before creating a new PR, and show a summary before updating an existing one.

**Note:** If your changes include UI updates, the skill will prompt you to add screenshots or a short screencast after the PR is created.

## ver-pr-review

`ver-pr-review` runs a structured two-phase review process and combines both positions into a terse final review that you confirm before it's submitted.

### Phase 1: Agent review

The agent inspects the diff automatically before asking you anything:

- **Implementation review** — reads the diff for bugs, logic errors, security issues, and correctness vs. the stated PR intent
- **Refactor check** — invokes a local refactor skill on changed files if one is available
- **Docs/spec sync** — checks that README, API docs, and specs (RSpec, OpenAPI, Gherkin, etc.) reflect the new behavior
- **Issue linkage** — looks for a linked issue or ticket in the description, commits, or branch name

It then reaches an internal technical position: **approve**, **request changes**, or **comment**.

### Phase 2: Human review

The agent presents its findings and builds a local test guide from the actual implementation — not from the PR description. You get:

1. The agent's position and any blocking items (specific, actionable, with file references)
2. A numbered walkthrough to reproduce and observe the change locally
3. Any judgment questions the agent can't resolve (design tradeoffs, thresholds, naming)

Then it asks for your position. You can approve, request specific changes, or comment.

### Final review

Positions are combined: **approval requires both agent and human to approve.** The skill drafts a terse review for your confirmation:

**If approving:**
```
Approved.
```

**If requesting changes:**
```
Requesting changes.

**Must fix:**
1. `lib/auth.ts:47` — token is compared with `==` instead of a constant-time check; use `crypto.timingSafeEqual`
2. No test covers the expired-token path added in `app/api/session.ts`

**Optional:**
- `README.md` — auth flow diagram is outdated
```

The review is submitted to GitHub, GitLab, or Bitbucket on your confirmation.

### Usage

> "Review PR #42"
>
> "Review the open PR on this branch"
>
> "Do a code review"

## What's included

### ver-pr-writer

| File | Purpose |
|------|---------|
| `skills/ver-pr-writer/SKILL.md` | Agent instructions for generating and validating PR descriptions |
| `skills/ver-pr-writer/reference/template.md` | Output template the agent follows |
| `skills/ver-pr-writer/reference/provider-strategies.md` | Per-provider submit strategies (GitHub, GitLab, Bitbucket) |
| `skills/ver-pr-writer/reference/pr-signals.md` | Team-facing explanation of PR signals |
| `skills/ver-pr-writer/reference/pr-template.md` | PR template teams can adopt |
| `skills/ver-pr-writer/reference/pr-signal-check.yml` | CI workflow to enforce required sections |

### ver-pr-review

| File | Purpose |
|------|---------|
| `skills/ver-pr-review/SKILL.md` | Agent instructions for the two-phase PR review process |
| `skills/ver-pr-review/reference/provider-review-strategies.md` | Per-provider commands for fetching PR context and submitting reviews |

## Git provider

The skill works with **GitHub, GitLab, and Bitbucket**. Both `ver-pr-writer` and `ver-pr-review` share a single control file at the repo root:

```yaml
# .ver-pr.yml
provider: github  # github | gitlab | bitbucket
```

**Setting the provider:**

You can create this file manually, or let the skill do it for you. On the first submit, the skill will:
1. Try to auto-detect the provider from your git remote URL
2. Confirm the detected provider with you
3. Write `.ver-pr.yml` automatically

You can also set it explicitly:

```bash
echo "provider: gitlab" > .ver-pr.yml
```

Commit `.ver-pr.yml` to share the provider setting with your team. If the file is absent, each person will be asked once.

**Provider-specific behavior:**
- **GitHub** — uses `gh` CLI or the `github-pull-request_create_pull_request` MCP tool; creates draft PRs by default
- **GitLab** — uses `glab` CLI; creates draft MRs; uses "Merge Request" terminology
- **Bitbucket** — uses `bb` CLI or falls back to the Bitbucket REST API; no native draft support

## Adopting PR signals in your project

To enforce the signal structure on your team's PRs:

1. Copy `reference/pr-template.md` to the appropriate location for your provider:
   - **GitHub:** `.github/pull_request_template.md`
   - **GitLab:** `.gitlab/merge_request_templates/Default.md`
   - **Bitbucket:** no native template support; use the description as a starting point
2. Copy `reference/pr-signal-check.yml` to `.github/workflows/pr-signal-check.yml` (GitHub Actions) or adapt it to your CI system
3. Add `.ver-pr.yml` to the repo root with your provider set
4. Install the skill so your coding agents generate signal-compliant PR descriptions

## Goals

- Avoid rubber-stamp review
- Help humans and agents focus on what matters
- Shift review from raw diff reading toward intent, risk, evidence, and deployability
- Create a shared signal contract for PR authors, reviewers, and tooling

## License

MIT
