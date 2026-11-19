---
name: dizzy
description: Query DizzySecurity issues using the dizzy-scan CLI
---

You have access to the `dizzy-scan` CLI for interacting with DizzySecurity.

## Authentication

Check if the user is logged in:

```bash
# Mac / Linux
cat ~/.dizzy/token

# Windows (PowerShell)
type "$env:USERPROFILE\.dizzy\token"
```

If the file doesn't exist or is empty, run `dizzy-scan login` and tell the user to authenticate in the browser. The browser will open `https://platform.dizzysecurity.com/auth/cli` | after authenticating, the token is saved automatically.

Token location: `~/.dizzy/token` (Mac/Linux) | `%USERPROFILE%\.dizzy\token` (Windows)

## Workflow 1: Fetch and triage issues

Run:

```bash
dizzy-scan issues --json
```

This calls `GET /api/v1/data/issues` (auth: `Authorization: Bearer dzsk_...`) and returns a JSON array.

Present findings as a severity-sorted table:

| Severity | Title | Status | Source | Repo |
|----------|-------|--------|--------|------|

Highlight CRITICAL and HIGH open issues. End with a summary line: "X critical, Y high open issues."

**Issue JSON fields:** `issue_id`, `title`, `severity` (CRITICAL|HIGH|MEDIUM|LOW), `status` (open|in_progress|resolved|closed), `category`, `source` (ci_scan|sandbox), `repo_url`, `description`, `locations`, `first_seen_at`, `last_seen_at`

**Extra fields when present:**
- `pr_url` — remediation PR link if one exists.

## Key API paths

| Purpose | Method | Path |
|---------|--------|------|
| List issues | GET | `/api/v1/data/issues` |
| List sandboxes | GET | `/api/v1/sandbox` |
| Full attack | POST | `/api/v1/sandbox/{repo_url}/attack` |

Base URL: `https://api.dizzysecurity.com`  
Auth: `Authorization: Bearer <token from ~/.dizzy/token>`
