# DizzySecurity Agent Skill

This skill lets AI agents query security issues and reproduce sandbox attacks using the `dizzy-scan` CLI.

## Install

**Mac (Apple Silicon)**
```bash
curl -sSL https://github.com/hagay3/dizzy-releases/releases/latest/download/dizzy-scan-darwin-arm64 \
  -o /usr/local/bin/dizzy-scan && chmod +x /usr/local/bin/dizzy-scan
```

**Linux (amd64)**
```bash
curl -sSL https://github.com/hagay3/dizzy-releases/releases/latest/download/dizzy-scan-linux-amd64 \
  -o /usr/local/bin/dizzy-scan && chmod +x /usr/local/bin/dizzy-scan
```

**Windows (amd64)** — download `dizzy-scan-windows-amd64.exe` from the [releases page](https://github.com/hagay3/dizzy-releases/releases/latest).

## Authenticate

```bash
dizzy-scan login
```

Opens the browser. Authenticate with your DizzySecurity account. The token is saved to `~/.dizzy/token` automatically. You only need to do this once.

To verify authentication:
```bash
cat ~/.dizzy/token
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

## List sandboxes

```bash
dizzy-scan sandbox list
```

## List sandbox attack paths

```bash
dizzy-scan sandbox paths <repo-url>
```

Returns JSON array of attack paths. Key fields: `path_id`, `title`, `severity`, `category`, `entry_point`, `steps`, `rerun_status`, `status`

## Re-run (reproduce) a specific attack path

```bash
dizzy-scan sandbox rerun <repo-url> <path-id>
```

Triggers the attack via Claude on the live sandbox.

Response: `{"status": "started|starting", "path_id": "..."}`
- `started` — sandbox is running, attack agent has started
- `starting` — sandbox is auto-starting first, attack will follow

The sandbox must be enabled for your company. `path_id` comes from `sandbox paths`.

## Trigger a full attack

```bash
dizzy-scan sandbox attack <repo-url>
```

## Agent workflow: triage and reproduce

1. Check auth: `cat ~/.dizzy/token` — if missing, run `dizzy-scan login`
2. List issues: `dizzy-scan issues --json` → identify CRITICAL/HIGH open issues
3. For each issue: `dizzy-scan sandbox paths <repo_url>` → find matching attack path by `title`/`category`
4. Reproduce: `dizzy-scan sandbox rerun <repo_url> <path_id>`
5. Report back: tell the user "Attack rerun triggered for [title] — results will appear in the platform"

## API reference (for direct HTTP calls)

**Base URL:** `https://api.dizzysecurity.com`  
**Auth header:** `Authorization: Bearer dzsk_...`

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/data/issues` | List all issues for the company |
| GET | `/api/v1/sandbox` | List sandboxes |
| GET | `/api/v1/sandbox/{repo_url}/attack-paths` | List attack paths |
| POST | `/api/v1/sandbox/{repo_url}/attack-paths/{path_id}/rerun` | Re-run a specific attack path |
| POST | `/api/v1/sandbox/{repo_url}/attack` | Trigger a full attack |

All sandbox endpoints require `sandbox_enabled` for the company.
