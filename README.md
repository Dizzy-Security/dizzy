# DizzySecurity — Agent Skill & CLI

This repository contains everything needed for clients to use their own AI agents (Claude Code, Cursor, Copilot, etc.) with the DizzySecurity platform.

It provides:
- **`dizzy-scan` CLI** — authenticate, query security issues, and reproduce sandbox attack paths from the terminal
- **`skill.md`** — a Claude Code skill that teaches agents the two core workflows
- **`AGENTS.md`** — full reference for any AI agent: install, auth, all commands, API paths, and agentic workflow

## Quickstart

### 1. Install the CLI

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

**Windows (amd64)** — download `dizzy-scan-windows-amd64.exe` from the [releases page](https://github.com/Dizzy-Security/dizzy/releases/latest).

### 2. Log in

```bash
dizzy-scan login
```

Opens your browser to `https://platform.dizzysecurity.com/auth/cli`. After authenticating, your API token is saved to `~/.dizzy/token` automatically.

### 3. Fetch issues

```bash
dizzy-scan issues
dizzy-scan issues --severity CRITICAL --status open --json
```

### 4. Reproduce a sandbox attack

```bash
dizzy-scan sandbox paths https://github.com/your-org/your-repo
dizzy-scan sandbox rerun https://github.com/your-org/your-repo <path-id>
```

## Using with Claude Code

Add the skill to your project by copying `skill.md` into `.claude/skills/dizzy.md`, then invoke it with `/dizzy` inside Claude Code. The agent will:
1. Check auth and prompt login if needed
2. Fetch and triage your open security issues
3. Reproduce any issue by triggering the sandbox attack rerun

See `AGENTS.md` for the full command reference and API paths.

## Files

| File | Purpose |
|------|---------|
| `AGENTS.md` | Full agent reference — install, auth, all commands, API paths |
| `skill.md` | Claude Code skill definition |
| `CLAUDE.md` | Claude Code project instructions (loaded automatically in this directory) |
