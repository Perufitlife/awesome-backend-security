# Awesome Backend Security Auditors [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> Curated, keyless **security auditors** for the modern backend stack — BaaS platforms, headless CMSs, GraphQL engines, workflow runners and local LLM servers. Every tool here runs locally and **confirms each leak with an active anonymous probe** instead of just inferring it from config.

The single most common production breach in this stack is boring and universal: a backend left readable by the **public / anonymous role**. Supabase ships RLS-disabled tables, Firebase ships `allow read: if true`, Strapi/Directus/Payload leave the Public role on `find`, Hasura sets `unauthorized-role: public`, Ollama binds `0.0.0.0` with no auth. The tools below find that — and prove it — one command at a time.

## Contents

- [Why active-probe](#why-active-probe)
- [BaaS & Databases](#baas--databases)
- [Headless CMS](#headless-cms)
- [GraphQL](#graphql)
- [Workflow & Automation](#workflow--automation)
- [Local LLM / AI](#local-llm--ai)
- [Secrets & Exposure](#secrets--exposure)
- [MCP servers](#mcp-servers)
- [Reference reading](#reference-reading)

## 🔍 Free audit

Not sure if your backend is exposed? **[Open a free-audit request](https://github.com/Perufitlife/awesome-backend-security/issues/new?template=free-audit.yml)** with your URL and I'll run the matching auditor and post the findings + exact fixes back — free, read-only, nothing downloaded or changed. If you'd rather have the fixes done for you, there's a [$99 fixed-scope audit](https://perufitlife.github.io/supabase-security-skill/).

## Guides

- [How to tell if your backend is leaking data (and fix it)](guides/find-backend-data-leaks.md) — a platform-by-platform checklist with a one-line command to confirm each leak.
- [Is your Ollama server exposed?](guides/is-your-ollama-server-exposed.md) — 175,000+ instances run with no auth. Check yours in one command.
- [Supabase is locking down public table access on Oct 30, 2026 — are you ready?](guides/supabase-public-schema-deadline-checklist.md) — pre-deadline checklist to find and fix anon-exposed tables.
- [Can opening a repo in Claude Code run code or steal your API key?](guides/claude-code-untrusted-repo-security.md) — the `.claude/` config attack surface (CVE-2025-59536) and how to scan a repo before you open it.

## Why active-probe

A linter that reads your rules file tells you what *might* be exposed. An **active probe** sends the exact unauthenticated request an attacker would and shows you the bytes that actually come back. Every tool in this list is:

- **Keyless** where possible — point it at a URL, no admin token needed for the public-exposure checks.
- **Local-first** — your data and credentials never leave your machine.
- **Zero-dependency, MIT** — auditable in one file, free forever.

## BaaS & Databases

- [supabase-security](https://github.com/Perufitlife/supabase-security-skill) — RLS-disabled tables, anon grants, public buckets and `SECURITY DEFINER` functions; active anon-key probe confirms each leak. [npm](https://www.npmjs.com/package/supabase-security)
- [firebase-security](https://github.com/Perufitlife/firebase-security-skill) — the infamous `match /{document=**} { allow read, write: if true; }`, expired test-mode rules and auth-without-ownership in `firestore.rules`. [npm](https://www.npmjs.com/package/firebase-security)
- [pocketbase-security](https://github.com/Perufitlife/pocketbase-security-skill) — empty API rules (fully public), `@request.auth.id != ""` over-permissive rules, dangerous `true` literals. [npm](https://www.npmjs.com/package/pocketbase-security)
- [appwrite-security](https://github.com/Perufitlife/appwrite-security-skill) — `any` role grants, document-security misconfig and over-permissive collection permissions. [npm](https://www.npmjs.com/package/appwrite-security)
- [nhost-security](https://github.com/Perufitlife/nhost-security-skill) — Hasura/Nhost anonymous role with open SELECT, missing row filters, public introspection. [npm](https://www.npmjs.com/package/nhost-security)
- [convex-security](https://github.com/Perufitlife/convex-security) — public queries/mutations reachable without auth on a Convex deployment's HTTP API, CORS reflection and metadata leaks. [npm](https://www.npmjs.com/package/convex-security)

## Headless CMS

- [strapi-security](https://github.com/Perufitlife/strapi-security) — public-role read exposure, CORS reflection, `/api/users` enumeration, GraphQL introspection and the relational-populate admin oracle (CVE-2026-27886 class). [npm](https://www.npmjs.com/package/strapi-security)
- [directus-security](https://github.com/Perufitlife/directus-security) — public-role data exposure, search-param field enumeration (CVE-2025-30352), unauth version/schema leak (CVE-2025-53887) and GraphQL introspection. [npm](https://www.npmjs.com/package/directus-security)
- [payload-security](https://github.com/Perufitlife/payload-security) — collections readable without auth, field-level leaks (`apiKey`/`email`/`hash`/`salt`), user enumeration and open first-user registration. [npm](https://www.npmjs.com/package/payload-security)

## GraphQL

- [hasura-security](https://github.com/Perufitlife/hasura-security) — open introspection without the admin secret, the anonymous `public` unauthorized role leaking tables/rows, an unauthenticated console and a missing admin secret. [npm](https://www.npmjs.com/package/hasura-security)

## Workflow & Automation

- [n8n-security](https://github.com/Perufitlife/n8n-security) — unauthenticated `/rest/settings` config+version leak, open owner-setup takeover, version vs known critical CVEs (CVE-2026-21858 "Ni8mare", CVSS 10.0) and no-auth editor/REST API. [npm](https://www.npmjs.com/package/n8n-security)

## Local LLM / AI

- [ollama-security](https://github.com/Perufitlife/ollama-security) — a publicly bound, unauthenticated Ollama API (175k+ found exposed) proven via anonymous probes of `/api/tags`, `/api/ps`, `/api/version` and CORS reflection — without downloading a model or running a workload. [npm](https://www.npmjs.com/package/ollama-security)
- [dotclaude-security](https://github.com/Perufitlife/dotclaude-security) — scans a repo's `.claude/` config (hooks, MCP servers, env, permissions) for the RCE (CVE-2025-59536) and API-key-exfiltration (CVE-2026-21852) footguns that fire when you open an untrusted repo. [npm](https://www.npmjs.com/package/dotclaude-security)

## Secrets & Exposure

- [dotenv-exposure-check](https://github.com/Perufitlife/dotenv-exposure-check) — probes a live URL for accidentally-served secret artifacts (`.env`, `.git/`, `.js.map` source maps, `.DS_Store`, backups) and confirms each by fetching and fingerprinting the bytes. [npm](https://www.npmjs.com/package/dotenv-exposure-check)

## MCP servers

- [web-exposure-mcp](https://github.com/Perufitlife/web-exposure-mcp) — an MCP server that points an AI agent at a deployed URL and confirms which secret files are actually being served, by fetching the bytes and fingerprinting content (not trusting status codes). [npm](https://www.npmjs.com/package/web-exposure-mcp)

## Reference reading

- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
- [Supabase: securing your data with RLS](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Firebase Security Rules](https://firebase.google.com/docs/rules)
- [Strapi: Users & Permissions](https://docs.strapi.io/dev-docs/plugins/users-permissions)
- [Hasura: the `HASURA_GRAPHQL_UNAUTHORIZED_ROLE` footgun](https://github.com/hasura/graphql-engine/issues/5501)
- [Ollama has no authentication by default (CNVD-2025-04094)](https://github.com/ollama/ollama/issues/849)

## Contributing

Found a tool that fits — keyless, local-first, actively probes to confirm? PRs welcome. Keep entries one line, alphabetical within a section, with a `[npm]` link where published.

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

To the extent possible under law, the contributors have waived all copyright and related rights to this work.
