# Provider Strategies

Per-provider CLI commands and detection logic for submit mode.

## Control file

The skill reads `.ver-pr-writer.yml` from the repo root to determine the provider.

```yaml
provider: github  # github | gitlab | bitbucket
```

If this file is absent or `provider` is unset, the skill auto-detects from the git remote and asks the user to confirm, then writes the file.

## Auto-detection

Run `git remote get-url origin` and match against known hostnames:

| Hostname in remote URL | Provider |
|---|---|
| `github.com` | `github` |
| `gitlab.com` or self-hosted GitLab | `gitlab` |
| `bitbucket.org` | `bitbucket` |

If detection is ambiguous or the remote URL is private/custom, ask the user explicitly.

## Saving the control file

After the user confirms or selects a provider, write `.ver-pr-writer.yml` to the repo root:

```bash
echo "provider: <github|gitlab|bitbucket>" > .ver-pr-writer.yml
```

Inform the user:
> Saved provider as `<provider>` to `.ver-pr-writer.yml`. Commit this file to share it with your team.

## GitHub

**Auth check:** `gh auth status`
If it fails, tell the user to run `gh auth login` and stop.

**Detect existing PR:**
```bash
gh pr view --json number,title,url
```

**Create (preferred — MCP tool):**
If `github-pull-request_create_pull_request` is available, use it:
- `title`: H1 from the PR description
- `body`: full PR description
- `head`: current branch (`git branch --show-current`)
- `draft`: `true` unless user requests otherwise

**Create (CLI fallback):**
```bash
gh pr create --title "<title>" --body-file /tmp/pr-body.md --draft
```

**Update existing:**
```bash
gh pr edit --title "<title>" --body-file /tmp/pr-body.md
```

**Open in browser:**
```bash
gh pr view --web
```

## GitLab

**Auth check:** `glab auth status`
If it fails, tell the user to run `glab auth login` and stop.

**Detect existing MR:**
```bash
glab mr view
```
If it exits non-zero, no MR exists for the current branch.

**Create:**
```bash
glab mr create --title "<title>" --description-file /tmp/pr-body.md --draft
```

**Update existing:**
Write body to a temp file, then:
```bash
glab mr update <id> --description "$(cat /tmp/pr-body.md)"
```
Get the MR ID from `glab mr view --output json | jq .iid`.

**Open in browser:**
```bash
glab mr view --web
```

Note: GitLab calls these "Merge Requests" (MRs), not pull requests. Use that term in any user-facing messages when the provider is `gitlab`.

## Bitbucket

**Auth check:** `bb auth status` (Bitbucket CLI) or verify `~/.config/bb/` exists.
If unavailable, fall back to REST API via `curl`.

**Detect existing PR (CLI):**
```bash
bb pr list --state OPEN
```

**Detect existing PR (REST API fallback):**
```bash
curl -s -u "<user>:<app-password>" \
  "https://api.bitbucket.org/2.0/repositories/<workspace>/<slug>/pullrequests?q=source.branch.name=\"$(git branch --show-current)\"&state=OPEN"
```

**Create (CLI):**
```bash
bb pr create --title "<title>" --body "$(cat /tmp/pr-body.md)"
```

**Create (REST API fallback):**
```bash
curl -s -X POST -u "<user>:<app-password>" \
  -H "Content-Type: application/json" \
  -d "{\"title\": \"<title>\", \"description\": \"$(cat /tmp/pr-body.md | jq -Rs .)\", \"source\": {\"branch\": {\"name\": \"$(git branch --show-current)\"}}, \"close_source_branch\": false}" \
  "https://api.bitbucket.org/2.0/repositories/<workspace>/<slug>/pullrequests"
```

**Update existing (REST API):**
```bash
curl -s -X PUT -u "<user>:<app-password>" \
  -H "Content-Type: application/json" \
  -d "{\"title\": \"<title>\", \"description\": \"$(cat /tmp/pr-body.md | jq -Rs .)\"}" \
  "https://api.bitbucket.org/2.0/repositories/<workspace>/<slug>/pullrequests/<pr-id>"
```

Note: Bitbucket calls these "Pull Requests". Draft PRs are not natively supported on Bitbucket Cloud; omit the draft flag.

**If neither CLI nor credentials are available:**
Provide the user with the direct URL to create a PR:
```
https://bitbucket.org/<workspace>/<slug>/pull-requests/new?source=<branch>
```
