<p align="center">
  <a href="https://dizzysecurity.com" target="_blank">
    <img alt="Dizzy Security" src=".github/logo.png" width="200"/>
  </a>
</p>

<p align="center">
  <strong>Your friendly hacker | always trying to break into your app, always on your side</strong>
</p>

<p align="center">
  <a href="https://platform.dizzysecurity.com">Platform</a>
  ·
  <a href="https://dizzysecurity.com">Website</a>
  ·
  <a href="https://github.com/Dizzy-Security/dizzy/releases/latest">Download CLI</a>
  ·
  <a href="AGENTS.md">Agent Reference</a>
</p>

<br />

## What is this?

Dizzy is a friendly hacker that continuously tries to penetrate your app.

It runs attack simulations against your repos around the clock, finds real exploitable vulnerabilities, and shows you live proof-of-exploit in a sandbox. When it breaks in, it tells you exactly how and exactly how to fix it.

This repo connects your AI coding agent (Claude Code, Cursor, Copilot) directly to Dizzy. Your agent can pull the latest attack results, show you what got breached, and walk you through the fix, without leaving your editor.


Built for solo founders and small teams who ship fast and can't afford to get breached.

---

## Install

**Mac (Homebrew)**
```bash
brew install Dizzy-Security/tap/dizzy-scan
```

**Mac (Apple Silicon)**
```bash
curl -sSL https://github.com/Dizzy-Security/dizzy/releases/latest/download/dizzy-scan-darwin-arm64 \
  -o /usr/local/bin/dizzy-scan && chmod +x /usr/local/bin/dizzy-scan
```

**Linux (amd64)**
```bash
curl -sSL https://github.com/Dizzy-Security/dizzy/releases/latest/download/dizzy-scan-linux-amd64 \
  -o /usr/local/bin/dizzy-scan && chmod +x /usr/local/bin/dizzy-scan
```

**Windows (amd64)**
```powershell
Invoke-WebRequest -Uri "https://github.com/Dizzy-Security/dizzy/releases/latest/download/dizzy-scan-windows-amd64.exe" `
  -OutFile "$env:LOCALAPPDATA\Microsoft\WindowsApps\dizzy-scan.exe"
```

---

## Quickstart

### 1. Log in

```bash
dizzy-scan login
```

Opens your browser to `https://platform.dizzysecurity.com/auth/cli`. After authenticating, your API token is saved automatically. You only need to do this once.

| Platform | Token location |
|----------|---------------|
| Mac / Linux | `~/.dizzy/token` |
| Windows | `%USERPROFILE%\.dizzy\token` |

### 2. See what Dizzy broke into

```bash
dizzy-scan issues                              # all issues
dizzy-scan issues --severity CRITICAL          # filter by severity (CRITICAL | HIGH | MEDIUM | LOW)
dizzy-scan issues --status open --json         # machine-readable JSON
```

### 3. Watch the attack live

```bash
# List attack paths Dizzy has found for your repo
dizzy-scan sandbox paths https://github.com/your-org/your-repo

# Re-run a specific attack (triggers a live exploit in the sandbox | you watch it happen)
dizzy-scan sandbox rerun https://github.com/your-org/your-repo <path-id>
```

---

## Using with Claude Code

Copy `skill.md` into your project as `.claude/skills/dizzy.md`. Claude will automatically use it when you ask about security issues in your repo.

Your agent will:
1. Check auth | prompt `dizzy-scan login` if no token found
2. Pull the latest attacks Dizzy ran | severity-sorted, CRITICAL highlighted
3. Show live sandbox replays and walk you through the fix

---

## Using with any AI agent

Point your agent at `AGENTS.md`. It contains everything needed to operate the CLI autonomously:
- Install commands for each platform
- Auth setup
- All subcommands with flags and example output
- Full API path reference (for direct HTTP calls when the CLI is not available)

---

## API reference

All commands resolve to these backend endpoints. Agents can call them directly with `Authorization: Bearer <token>`.

**Base URL:** `https://api.dizzysecurity.com`

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/data/issues` | List all issues Dizzy found |
| GET | `/api/v1/sandbox` | List sandboxes |
| GET | `/api/v1/sandbox/{repo_url}/attack-paths` | List attack paths Dizzy has run |
| POST | `/api/v1/sandbox/{repo_url}/attack-paths/{path_id}/rerun` | Re-run a specific attack |
| POST | `/api/v1/sandbox/{repo_url}/attack` | Trigger a full attack run |

---

## Files

| File | Purpose |
|------|---------|
| `AGENTS.md` | Full agent reference | install, auth, commands, API paths |
| `skill.md` | Claude Code skill definition (`/dizzy`) |
| `CLAUDE.md` | Claude Code project context (auto-loaded in this directory) |
