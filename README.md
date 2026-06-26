# ver-pr-writer

An agent skill for generating reviewer-focused pull request descriptions using explicit PR signals.

Helps both humans and agents answer: what changed, why, where the risk is, how it was validated, what deserves attention, and what still needs judgment.

## Why PR signals?

If your team is using AI coding agents, you've probably noticed something: **the bottleneck isn't writing code anymore. It's reviewing it**. See [MOTIVATION.md](MOTIVATION.md) for the full analysis of why structured PR signals matter in agentic SDLC workflows.

## Install

```bash
npx skills add Vermonster/ver-pr-writer
```

Or install to specific agents:

```bash
npx skills add Vermonster/ver-pr-writer -a github-copilot
npx skills add Vermonster/ver-pr-writer -a claude-code
npx skills add Vermonster/ver-pr-writer -a cursor
```

## What it does

When activated, the skill:

1. Inspects the git diff, changed files, commit messages, tests, and other available context
2. Classifies the change type, scope, and risk
3. Generates a structured PR description with three sections: **Intent** (purpose, outcome, type/risk/scope), **Technical** (changed files, risk, test commands, operations), and **Human Review** (where to start, demo walkthrough, questions, uncertainty)
4. Validates the generated signals against the actual diff and fixes mismatches

## Usage

The skill has two modes: **draft** and **submit**.

### Draft mode (default)

Generates the PR description and displays it without touching your remote. Use this to review and iterate before submitting.

> "Write a PR description for this branch"
>
> "Draft PR signals for my changes"
>
> "Generate a PR description"

### Submit mode

Generates the PR description, then creates or updates the pull request on your remote (GitHub, GitLab, or Bitbucket). The skill checks for an existing PR on the current branch and either creates a new one or updates the body.

> "Submit a PR for this branch"
>
> "Create a PR with signals"
>
> "Update my PR description"

Submit mode uses the GitHub Pull Request tool when available, and falls back to `gh` CLI. New PRs are created as drafts by default.

The skill will always confirm before creating a new PR, and show a summary before updating an existing one.

**Note:** If your changes include UI updates, the skill will prompt you to add screenshots or a short screencast after the PR is created.

## What's included

| File | Purpose |
|------|---------|
| `skills/ver-pr-writer/SKILL.md` | Agent instructions for generating and validating PR descriptions |
| `skills/ver-pr-writer/reference/template.md` | Output template the agent follows |
| `skills/ver-pr-writer/reference/provider-strategies.md` | Per-provider submit strategies (GitHub, GitLab, Bitbucket) |
| `skills/ver-pr-writer/reference/pr-signals.md` | Team-facing explanation of PR signals |
| `skills/ver-pr-writer/reference/pr-template.md` | PR template teams can adopt |
| `skills/ver-pr-writer/reference/pr-signal-check.yml` | CI workflow to enforce required sections |

## Git provider

The skill works with **GitHub, GitLab, and Bitbucket**. It stores your provider in a control file at the repo root:

```yaml
# .ver-pr-writer.yml
provider: github  # github | gitlab | bitbucket
```

**Setting the provider:**

You can create this file manually, or let the skill do it for you. On the first submit, the skill will:
1. Try to auto-detect the provider from your git remote URL
2. Confirm the detected provider with you
3. Write `.ver-pr-writer.yml` automatically

You can also set it explicitly:

```bash
echo "provider: gitlab" > .ver-pr-writer.yml
```

Commit `.ver-pr-writer.yml` to share the provider setting with your team. If the file is absent, each person will be asked once.

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
3. Add `.ver-pr-writer.yml` to the repo root with your provider set
4. Install the skill so your coding agents generate signal-compliant PR descriptions

## Goals

- Avoid rubber-stamp review
- Help humans and agents focus on what matters
- Shift review from raw diff reading toward intent, risk, evidence, and deployability
- Create a shared signal contract for PR authors, reviewers, and tooling

## License

MIT
