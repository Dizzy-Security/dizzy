# DizzySecurity Agent Skill

This skill lets AI agents query security issues and reproduce sandbox attacks using the `dizzy-scan` CLI.

## Install

**Mac (Apple Silicon)**
```bash
curl -sSL https://github.com/Dizzy-Security/dizzy/releases/latest/download/dizzy-scan-darwin-arm64 \
  -o ~/bin/dizzy-scan && chmod +x ~/bin/dizzy-scan
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

**JSON fields:** `issue_id`, `title`, `severity` (CRITICAL|HIGH|MEDIUM|LOW), `status` (open|resolved|closed), `category`, `source` (ci_scan|sandbox), `repo_url`, `description`, `locations`, `first_seen_at`, `last_seen_at`

**Extra fields when present:**
- `deps` — on dependency summary issues: array of `{name, version, severity}` for each vulnerable package
- `sandbox_data` — on sandbox issues: array of matching attack paths with `path_id`, `title`, `severity`, `category`, `entry_point`, `steps`, `rerun_status`. Use `path_id` directly with `sandbox rerun` — no separate `paths` lookup needed
- `pr_url` — link to the remediation PR if one exists

## List sandboxes

```bash
dizzy-scan sandbox list
```

## Re-run (reproduce) a specific attack path

```bash
dizzy-scan sandbox rerun <repo-url> <path-id>
```

Triggers the attack via Claude on the live sandbox.

Response: `{"status": "started|starting", "path_id": "..."}`
- `started` | sandbox is running, attack agent has started
- `starting` | sandbox is auto-starting first, attack will follow

The sandbox must be enabled for your company. `path_id` comes from `sandbox paths`.

## Trigger a full attack

```bash
dizzy-scan sandbox attack <repo-url>
```

## Agent workflow: triage and reproduce

1. Check auth: Mac/Linux — `cat ~/.dizzy/token` | Windows — `type "%USERPROFILE%\.dizzy\token"` | if missing or empty, run `dizzy-scan login`
2. List issues: `dizzy-scan issues --json` → identify CRITICAL/HIGH open issues
3. Sandbox issues already include `sandbox_data[].path_id` — no separate lookup needed
4. Reproduce: `dizzy-scan sandbox rerun <repo_url> <path_id>`
5. Report back: tell the user "Attack rerun triggered for [title] | results will appear in the platform"

## API reference (for direct HTTP calls)

**Base URL:** `https://api.dizzysecurity.com`  
**Auth header:** `Authorization: Bearer dzsk_...`

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/data/issues` | List all issues for the company |
| GET | `/api/v1/sandbox` | List sandboxes |
| GET | `/api/v1/sandbox/{repo_url}/attack-paths` | List attack paths (embedded in issues JSON as `sandbox_data`) |
| POST | `/api/v1/sandbox/{repo_url}/attack-paths/{path_id}/rerun` | Re-run a specific attack path |
| POST | `/api/v1/sandbox/{repo_url}/attack` | Trigger a full attack |

