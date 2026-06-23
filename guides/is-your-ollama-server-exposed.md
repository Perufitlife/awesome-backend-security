---
layout: default
title: "Is your Ollama server exposed? How to check in one command (175,000+ are)"
description: "Ollama ships with no authentication and binds to all interfaces by default. Over 175,000 instances are publicly reachable. Here's how to check yours in one command, and how to lock it down."
---

# Is your Ollama server exposed?

> Ollama has **no authentication by default** and, in many setups, listens on `0.0.0.0:11434` — reachable by anyone who finds the IP. Security researchers have catalogued **175,000+ publicly exposed instances**. An open Ollama means anyone can list and pull your models, burn your GPU on their prompts, and — chained with other bugs — has been used as a foothold for worse.

If you run Ollama on a VPS, a cloud box, or behind a port-forward, this two-minute check is worth doing.

## Check it in one command

```bash
npx ollama-security --url http://your-host:11434
```

This sends a handful of **read-only, anonymous** requests — exactly what any visitor could send — to confirm whether your instance answers without auth. It never downloads a model or runs a real workload (the `/api/pull` and `/api/generate` checks use empty payloads, so a `200`/`400` just proves the endpoint is open). You get back which of these are reachable:

- `/api/version` — server version (and whether it's a vulnerable one)
- `/api/tags` — your full list of installed models
- `/api/ps` — what's currently loaded in memory
- `/api/generate` / `/api/pull` — whether anyone can run inference or pull/push models on your box
- CORS reflection — whether a malicious website can talk to your instance from a victim's browser

If it comes back with confirmed findings, your instance is open to the internet.

## Why "no auth by default" bites people

Ollama is built to be a local dev tool, so it assumes a trusted network. The moment you:

- run it on a cloud VM to share with a team,
- set `OLLAMA_HOST=0.0.0.0` to reach it from another machine, or
- forward port `11434` through your router,

…it becomes reachable by the whole internet with **zero** authentication ([CNVD-2025-04094](https://github.com/ollama/ollama/issues/849)). There's no login to get past — the API just answers.

## How to lock it down

1. **Don't bind to all interfaces.** Keep `OLLAMA_HOST=127.0.0.1` and reach it over an SSH tunnel or a private network instead of the public internet.
2. **Put a reverse proxy with auth in front.** Nginx/Caddy with basic-auth or an API key, so the raw `:11434` port is never public.
3. **Firewall the port.** Allow `11434` only from the specific IPs that need it.
4. **Use a VPN / private network** (Tailscale, WireGuard) for team access instead of exposing the port.
5. **Re-check after every change** — run the command above again to confirm the port is no longer answering anonymously.

## Doing this across a fleet?

`ollama-security` prints JSON, so you can loop it over a list of hosts and gate a CI/monitoring job on the result. It's MIT, zero-dependency, and runs entirely on your machine.

→ [ollama-security on GitHub](https://github.com/Perufitlife/ollama-security) · [npm](https://www.npmjs.com/package/ollama-security)

---

Want it checked for you? [Open a free audit request](https://github.com/Perufitlife/awesome-backend-security/issues/new?template=free-audit.yml) with your URL and I'll run it and send the report back — free.

← Back to [Awesome Backend Security Auditors](../)
