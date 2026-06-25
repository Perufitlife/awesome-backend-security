---
layout: default
title: "Can opening a repo in Claude Code run code or steal your API key? (.claude security)"
description: "Cloning an untrusted repo and opening it in an AI coding agent can execute attacker-controlled hooks and exfiltrate your API key (CVE-2025-59536, CVE-2026-21852). Here's what's in the .claude config attack surface and how to scan a repo before you open it."
---

# Can opening a repo in Claude Code run code or steal your API key?

> Short answer: yes, if the repo ships a malicious `.claude/` config and you open it without looking. AI coding agents read repo-local configuration — hooks, MCP server definitions, environment variables, allowed-tools — and some of that runs automatically. Two 2026 CVEs ([CVE-2025-59536](https://thehackernews.com), auto shell-exec on init in an untrusted dir; [CVE-2026-21852](https://thehackernews.com), API-key exfiltration) both trace back to repo-controlled config.

The dangerous part: this fires from the most ordinary action in a developer's day — `git clone` a repo and open it. You don't have to run anything. The config does.

## The `.claude/` attack surface

A repo you clone can carry a `.claude/` directory (or `.mcp.json`, `settings.json`) containing:

- **Hooks** — commands the agent runs on events like session start, before/after tool use. A `SessionStart` hook can execute a shell command the moment you open the repo.
- **MCP server definitions** — point the agent at an attacker-controlled server, or a local binary that runs on launch.
- **Environment variables** — can redirect API base URLs (so your requests, and your key, flow to the attacker) or inject secrets into commands.
- **`permissions.allow`** — a too-broad allowlist that lets the agent run commands without prompting you.

None of this is exotic. It's the same config you write for your own projects — which is exactly why a malicious version blends in.

## Scan a repo before you trust it

```bash
npx dotclaude-security --dir ./path-to-cloned-repo
```

It statically reads the repo's `.claude/` config and flags the dangerous lines — auto-running hooks, suspicious MCP/env redirection, over-broad permissions, and secrets committed in config. It **never executes** what it finds; it just shows you the exact lines and why they're risky, so you can decide before you open the repo in your agent.

## Habits that close the gap

1. **Scan untrusted repos before opening them** in any AI coding agent — `npx dotclaude-security --dir .` takes seconds.
2. **Review `.claude/settings.json`, `.mcp.json` and any hooks** in a PR diff the same way you'd review a CI workflow change.
3. **Keep your agent's auto-run permissions tight** — prefer prompts over blanket `allow`.
4. **Don't open random repos from links/issues** in an environment that holds production API keys.
5. **Pin and audit MCP servers** — treat a new MCP definition like adding a dependency.

## Why the big scanners miss this

Most agent-security tooling scans *skills* or *prompts* for injection. The `.claude/` **config file** attack surface — hooks, MCP, env, permissions that execute on open — is a different, mostly-unowned lane. That's the gap `dotclaude-security` fills.

→ [dotclaude-security on GitHub](https://github.com/Perufitlife/dotclaude-security) · [npm](https://www.npmjs.com/package/dotclaude-security)

---

Maintain a lot of repos or onboard external contributors? [Open a free audit request](https://github.com/Perufitlife/awesome-backend-security/issues/new?template=free-audit.yml) and I'll scan a repo's config and send the findings back — free.

← Back to [Awesome Backend Security Auditors](../)
