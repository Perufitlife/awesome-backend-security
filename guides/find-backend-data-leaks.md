---
layout: default
title: "How to tell if your backend is leaking data (and fix it) — 2026 checklist"
description: "A practical, platform-by-platform checklist to find public/anonymous data exposure in Supabase, Firebase, Strapi, Directus, Payload, Hasura, Convex, n8n and Ollama — with a one-line command to confirm each leak."
---

# How to tell if your backend is leaking data (and fix it)

> A practical 2026 checklist. The single most common production data breach in modern app stacks isn't a clever exploit — it's a backend left **readable by the anonymous/public role**. This guide shows you how to check each major platform in one command, and how to close the hole.

Almost every breach in this category shares one root cause: a default that exposes data to unauthenticated callers, left on in production. Below, for each platform, is **what to check, how to confirm it for real (an anonymous request — exactly what an attacker sends), and how to fix it.**

Every tool linked here is open-source (MIT), zero-dependency, runs locally so your credentials never leave your machine, and **confirms the leak with an active probe** instead of guessing from config.

---

## The universal test

Before anything platform-specific, ask: *if I open my app's API URL in a private browser window with no login, what comes back?* If you get rows of data, that data is public. Every tool below automates exactly that question and proves the answer.

---

## Supabase

**The footgun:** tables with Row Level Security (RLS) disabled, or `anon` role granted CRUD by default. Anyone who pulls the anon key out of your JS bundle can read those tables.

**Confirm it:**
```bash
npx supabase-security <project-ref> --html report.html
```
It lists RLS-disabled tables, public storage buckets and `SECURITY DEFINER` functions callable by `anon`, and confirms each with the anon key. Fix: enable RLS and write owner-scoped policies. → [supabase-security](https://github.com/Perufitlife/supabase-security-skill)

## Firebase

**The footgun:** `match /{document=**} { allow read, write: if true; }` left over from `firebase init`, or test-mode rules that are wide open until an expiry date.

**Confirm it:**
```bash
npx firebase-security firestore.rules --project-id your-project
```
→ [firebase-security](https://github.com/Perufitlife/firebase-security-skill)

## Strapi

**The footgun:** the Users & Permissions **Public** role with `find`/`findOne` enabled — your whole content API is readable. Plus CORS reflection and `/api/users` enumeration.

**Confirm it:**
```bash
npx strapi-security --url https://your-strapi.example.com
```
→ [strapi-security](https://github.com/Perufitlife/strapi-security)

## Directus

**The footgun:** public-role read on collections, plus the 2025 CVE cluster — search-param field enumeration leaking emails and password hashes (CVE-2025-30352) and an unauthenticated version/schema leak.

**Confirm it:**
```bash
npx directus-security --url https://your-directus.example.com
```
→ [directus-security](https://github.com/Perufitlife/directus-security)

## Payload CMS

**The footgun:** collections readable without auth, field-level leaks (`apiKey`, `email`, `hash`, `salt`), and open first-user registration.

**Confirm it:**
```bash
npx payload-security --url https://your-payload.example.com
```
→ [payload-security](https://github.com/Perufitlife/payload-security)

## Hasura

**The footgun:** `HASURA_GRAPHQL_UNAUTHORIZED_ROLE=public` plus open introspection — an unauthenticated client can download your whole schema and query tables.

**Confirm it:**
```bash
npx hasura-security --url https://your-hasura.example.com
```
→ [hasura-security](https://github.com/Perufitlife/hasura-security)

## Convex

**The footgun:** public queries/mutations reachable on the HTTP API without auth, returning real rows.

**Confirm it:**
```bash
npx convex-security --url https://your-deployment.convex.cloud
```
→ [convex-security](https://github.com/Perufitlife/convex-security)

## n8n (self-hosted)

**The footgun:** an unauthenticated `/rest/settings` config+version leak, open owner-setup takeover, and running a version vulnerable to CVE-2026-21858 ("Ni8mare", CVSS 10.0).

**Confirm it:**
```bash
npx n8n-security --url https://your-n8n.example.com
```
→ [n8n-security](https://github.com/Perufitlife/n8n-security)

## Ollama (local LLM)

**The footgun:** Ollama binds with **no authentication by default**. Over 175,000 instances have been found publicly exposed — open to model theft and free compute abuse.

**Confirm it:**
```bash
npx ollama-security --url http://your-host:11434
```
→ [ollama-security](https://github.com/Perufitlife/ollama-security)

---

## Bonus: things that leak outside your database

- **Served secret files** — `.env`, `.git/`, JS source maps and backups accidentally deployed to your web root. `npx dotenv-exposure-check --url https://your-site.com` confirms each by fetching the bytes. → [dotenv-exposure-check](https://github.com/Perufitlife/dotenv-exposure-check)
- **Your `.claude/` config** — hooks and MCP definitions in an untrusted repo can run code or exfiltrate API keys when opened (CVE-2025-59536, CVE-2026-21852). `npx dotclaude-security --dir ./repo` flags the dangerous lines. → [dotclaude-security](https://github.com/Perufitlife/dotclaude-security)

---

## Make it a habit

Run the relevant check **after every deploy** — exposure creeps back in when someone adds a collection or flips a default. Each tool prints JSON you can gate a CI job on.

Want it done for you? A [fixed-scope audit ($99, 24h)](https://perufitlife.github.io/supabase-security-skill/) verifies every finding live and sends a written report with the exact fixes.

← Back to [Awesome Backend Security Auditors](../)
