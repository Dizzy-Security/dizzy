---
name: dizzy
description: Query DizzySecurity issues and reproduce sandbox attack paths using the dizzy-scan CLI
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

**Issue JSON fields:** `issue_id`, `title`, `severity` (CRITICAL|HIGH|MEDIUM|LOW), `status` (open|resolved|closed), `category`, `source` (ci_scan|sandbox), `repo_url`, `description`, `locations`, `first_seen_at`

**Extra fields when present:**
- `sandbox_data` — on sandbox issues: attack paths with `path_id`, `title`, `severity`, `entry_point`, `steps`, `rerun_status`. Use `path_id` directly — no separate lookup needed.
- `deps` — on dependency summary issues: `[{name, version, severity}]` per vulnerable package.
- `pr_url` — remediation PR link if one exists.

## Workflow 2: Reproduce an issue in the sandbox

### Step 1 | Get the path_id

Sandbox issues returned by `dizzy-scan issues --json` already include `sandbox_data[].path_id`. Use it directly — no separate `sandbox paths` call needed.

If multiple paths are in `sandbox_data`, match by `title` similarity or ask the user which one to reproduce.

### Step 2 | Trigger the rerun

```bash
dizzy-scan sandbox rerun <repo-url> <path-id>
```

This calls `POST /api/v1/sandbox/{repo_url}/attack-paths/{path_id}/rerun`.

Possible responses:
- `{"status": "started", "path_id": "..."}` | Claude attack agent is running on the server
- `{"status": "starting", "path_id": "..."}` | sandbox is auto-starting, attack will follow

Tell the user: "Attack rerun triggered for **[title]**. Results will appear in the DizzySecurity platform."

## Key API paths

| Purpose | Method | Path |
|---------|--------|------|
| List issues | GET | `/api/v1/data/issues` |
| List sandboxes | GET | `/api/v1/sandbox` |
| List attack paths | GET | `/api/v1/sandbox/{repo_url}/attack-paths` |
| Rerun attack path | POST | `/api/v1/sandbox/{repo_url}/attack-paths/{path_id}/rerun` |
| Full attack | POST | `/api/v1/sandbox/{repo_url}/attack` |

Base URL: `https://api.dizzysecurity.com`  
Auth: `Authorization: Bearer <token from ~/.dizzy/token>`
