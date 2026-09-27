# DizzySecurity Agent Skill | Project Context

This directory is the public-facing client repository for the DizzySecurity platform. It is published separately so clients can use their own AI agents (Claude Code, Cursor, Copilot, etc.) to interact with DizzySecurity via the `dizzy-scan` CLI.

## What this repo is for

Clients copy this directory (or clone it) into their own projects to give their AI agent:
- A ready-to-use `dizzy-scan` CLI for querying issues and triggering sandbox reproductions
- `skill.md` | a Claude Code skill for the two core workflows (triage issues, reproduce in sandbox)
- `AGENTS.md` | full reference documentation any agent can read

## Using the skill

`skill.md` is a Claude Code skill — it is loaded automatically by the agent when the task involves DizzySecurity (e.g. "show my security issues", "reproduce this attack"). It is **not** a slash command; users do not type `/dizzy`.

When the skill is active, the agent should:
1. Check `~/.dizzy/token` | if missing or empty, run `dizzy-scan login`
2. For issue triage: `dizzy-scan issues --json`
3. For reproduction: `path_id` is embedded in `sandbox_data` on each sandbox issue — use it directly with `dizzy-scan sandbox rerun <repo> <path-id>`

Full workflow is in `skill.md`. Full command reference is in `AGENTS.md`.

## API

Base URL: `https://api.dizzysecurity.com`  
Auth: `Authorization: Bearer <token>` where token is the `dzsk_...` value from `~/.dizzy/token`

## CLI binary source

The `dizzy-scan` CLI is built from source and published at [github.com/Dizzy-Security/dizzy/releases](https://github.com/Dizzy-Security/dizzy/releases). The binary is a single static Go executable | no dependencies to install.
