# Provider Review Strategies

Per-provider commands for fetching PR context and submitting reviews.

## Control file

Shared with ver-pr-writer. Read `.ver-pr.yml` from the repo root:

```yaml
provider: github  # github | gitlab | bitbucket
```

**Detection order:**
1. Read `.ver-pr.yml` — if `provider` is set, use it.
2. Run `git remote get-url origin` and match: `github.com` → `github`, `gitlab.com` → `gitlab`, `bitbucket.org` → `bitbucket`.
3. If ambiguous, ask the user and write `.ver-pr.yml`.

After writing: *"Saved provider as `<provider>` to `.ver-pr.yml`. Commit this file to share it with your team."*

---

## Fetching PR context

### GitHub

```bash
# View PR description and metadata
gh pr view <number> --json number,title,body,headRefName,baseRefName,url

# Get the diff
gh pr diff <number>

# For current branch (no PR number)
git diff origin/HEAD...HEAD
git log origin/HEAD...HEAD --oneline
```

### GitLab

```bash
# View MR details
glab mr view <id> --output json

# Get the diff (use base/head from MR details)
git fetch origin
git diff origin/<base>...origin/<head>

# For current branch
git diff origin/HEAD...HEAD
git log origin/HEAD...HEAD --oneline
```

### Bitbucket

```bash
# CLI
bb pr view <id>

# REST API fallback
curl -s -u "<user>:<app-password>" \
  "https://api.bitbucket.org/2.0/repositories/<workspace>/<slug>/pullrequests/<id>"

# Diff via REST API
curl -s -u "<user>:<app-password>" \
  "https://api.bitbucket.org/2.0/repositories/<workspace>/<slug>/pullrequests/<id>/diff"
```

---

## Submitting reviews

### GitHub

**Approve:**
```bash
gh pr review <number> --approve --body-file /tmp/review-body.md
```

**Request changes:**
```bash
gh pr review <number> --request-changes --body-file /tmp/review-body.md
```

**Comment:**
```bash
gh pr review <number> --comment --body-file /tmp/review-body.md
```

If no PR number is known, omit it — `gh` will use the current branch's PR.

### GitLab

GitLab CLI supports approve and notes; "request changes" is expressed as a note with the PR left in open/draft state.

**Approve:**
```bash
glab mr approve <id>
```

If adding a comment alongside approval:
```bash
glab mr approve <id>
glab mr note <id> --message "$(cat /tmp/review-body.md)"
```

**Request changes (note + re-draft to prevent accidental merge):**
```bash
glab mr note <id> --message "$(cat /tmp/review-body.md)"
# Optionally re-mark as draft to signal it is not merge-ready:
glab mr update <id> --draft
```

**Comment only:**
```bash
glab mr note <id> --message "$(cat /tmp/review-body.md)"
```

Note: GitLab calls these "Merge Requests" (MRs). Use that term in any user-facing messages when the provider is `gitlab`.

### Bitbucket

Bitbucket supports approve, "needs work" (request changes), and comments.

**Approve:**
```bash
bb pr approve <id>
# or REST API:
curl -s -X POST -u "<user>:<app-password>" \
  "https://api.bitbucket.org/2.0/repositories/<workspace>/<slug>/pullrequests/<id>/approve"
```

**Request changes (needs work):**
```bash
# REST API — mark as "needs work":
curl -s -X PUT -u "<user>:<app-password>" \
  "https://api.bitbucket.org/2.0/repositories/<workspace>/<slug>/pullrequests/<id>/request-changes"

# Then add a comment with the details:
curl -s -X POST -u "<user>:<app-password>" \
  -H "Content-Type: application/json" \
  -d "{\"content\": {\"raw\": \"$(cat /tmp/review-body.md | sed 's/"/\\"/g')\"}}" \
  "https://api.bitbucket.org/2.0/repositories/<workspace>/<slug>/pullrequests/<id>/comments"
```

If `request-changes` endpoint is unavailable (plan limitation), fall back to comment only with `**Requesting changes.**` prefix.

**Comment only:**
```bash
curl -s -X POST -u "<user>:<app-password>" \
  -H "Content-Type: application/json" \
  -d "{\"content\": {\"raw\": \"$(cat /tmp/review-body.md | sed 's/"/\\"/g')\"}}" \
  "https://api.bitbucket.org/2.0/repositories/<workspace>/<slug>/pullrequests/<id>/comments"
```
