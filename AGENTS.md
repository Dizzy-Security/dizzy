# DizzySecurity Agent Skill

This skill lets AI agents query security issues and reproduce sandbox attacks using the `dizzy-scan` CLI.

## Install

**Mac**
```bash
brew tap dizzy-security/dizzy
brew install dizzy-scan
```

**Linux (amd64)**
```bash
curl -sSL https://github.com/Dizzy-Security/dizzy/releases/latest/download/dizzy-scan-linux-amd64 \
  -o ~/bin/dizzy-scan && chmod +x ~/bin/dizzy-scan
```

**Windows (amd64)**
```powershell
Invoke-WebRequest -Uri "https://github.com/Dizzy-Security/dizzy/releases/latest/download/dizzy-scan-windows-amd64.exe" `
  -OutFile "$env:LOCALAPPDATA\Microsoft\WindowsApps\dizzy-scan.exe"
```

## Authenticate

```bash
dizzy-scan login
```

Opens the browser. Authenticate with your DizzySecurity account. The token is saved automatically. You only need to do this once.

Token location:
- Mac / Linux: `~/.dizzy/token`
- Windows: `%USERPROFILE%\.dizzy\token`

To verify authentication:
```bash
# Mac / Linux
cat ~/.dizzy/token

# Windows (PowerShell)
type "$env:USERPROFILE\.dizzy\token"
```

## List security issues

```bash
dizzy-scan issues                          # all issues
dizzy-scan issues --severity CRITICAL      # filter by severity (CRITICAL, HIGH, MEDIUM, LOW)
dizzy-scan issues --status open            # filter by status (open, resolved, closed)
dizzy-scan issues --source sandbox         # filter by source (ci_scan, sandbox)
dizzy-scan issues --status open --json     # machine-readable JSON output
```

**JSON fields:** `issue_id`, `title`, `severity` (CRITICAL|HIGH|MEDIUM|LOW), `status` (open|in_progress|resolved|closed), `category`, `source` (ci_scan|sandbox), `repo_url`, `description`, `locations`, `first_seen_at`, `last_seen_at`

**Extra fields when present:**
- `pr_url` — link to the remediation PR if one exists

## List sandboxes

```bash
dizzy-scan sandbox list
```

## Trigger a full attack

```bash
dizzy-scan sandbox attack <repo-url>
```

## Agent workflow: triage issues

1. Check auth: Mac/Linux — `cat ~/.dizzy/token` | Windows — `type "%USERPROFILE%\.dizzy\token"` | if missing or empty, run `dizzy-scan login`
2. List issues: `dizzy-scan issues --json` → identify CRITICAL/HIGH open issues
3. Report back with a severity-sorted summary

## API reference (for direct HTTP calls)

**Base URL:** `https://api.dizzysecurity.com`  
**Auth header:** `Authorization: Bearer dzsk_...`

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/data/issues` | List all issues for the company |
| GET | `/api/v1/sandbox` | List sandboxes |
| POST | `/api/v1/sandbox/{repo_url}/attack` | Trigger a full attack |

