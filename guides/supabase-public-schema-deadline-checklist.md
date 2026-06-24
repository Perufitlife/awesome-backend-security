---
layout: default
title: "Supabase is locking down public table access on Oct 30, 2026 — are you ready? (checklist)"
description: "On Oct 30, 2026 Supabase enforces the no-auto-expose default for all existing projects. Here's a pre-deadline checklist to find which of your tables are leaking to anon today, and fix them before it breaks (or before someone scrapes them)."
---

# Supabase is locking down public table access — are you ready?

> Two dates matter. **May 30, 2026:** new Supabase projects no longer auto-expose `public`-schema tables to the Data API. **October 30, 2026:** that becomes the enforced default for **all existing projects**. If you've been on Supabase for a while, this is both a security win and a potential breakage — and a great moment to find what's been silently exposed.

If your app has been running since before mid-2026, you almost certainly have some mix of:

- Tables granted CRUD to the `anon` role by default (because that *was* the default).
- One or two tables where RLS was never enabled.
- `SECURITY DEFINER` functions that are technically callable by `anon`.

Until Oct 30 those are reachable by anyone who pulls your anon key out of the JS bundle. After Oct 30, some of them stop working — and you want to know *which* before your users hit errors.

## The 5-minute check

```bash
SUPABASE_ACCESS_TOKEN=sbp_xxx npx supabase-security YOUR_PROJECT_REF --html report.html
```

You get an HTML report listing every table with RLS disabled + anon grants, every public storage bucket, and every `SECURITY DEFINER` function callable by `anon` — each with copy-paste fix SQL. The active probe uses your anon key to **confirm** what's actually reachable, so you triage facts, not guesses. (Read access on the token is enough.)

## Pre-deadline checklist

1. **List every table reachable by `anon` today.** That's your exposure surface and your breakage surface. The report above gives it in one pass.
2. **For each: is it *meant* to be public?** A `public_stats` table might be intentional; a `users` or `orders` table is not. Decide per table — the tool can't know your intent.
3. **Enable RLS on anything that isn't deliberately open**, and write owner-scoped policies (`auth.uid() = user_id`).
4. **Re-grant intentionally-public reads explicitly** via a policy, so they keep working after Oct 30 instead of silently breaking.
5. **Check storage buckets.** Public buckets leak files (PII docs, payment proofs) regardless of table RLS.
6. **Check `SECURITY DEFINER` functions.** They run as the definer and can bypass RLS — make sure `anon` can't call the sensitive ones.
7. **Re-run the audit** until the report is clean, then keep it in CI so new tables don't reintroduce the gap.

## Why do it now, not on Oct 29

Two reasons. First, the security one: every day before the deadline, the exposed tables are scrapeable by anyone — the deadline doesn't change that, it just changes the default going forward. Second, the operational one: if you wait, you'll be debugging "why did half my API start 401-ing" under pressure. Doing the audit now turns a deadline into a checklist.

→ [supabase-security on GitHub](https://github.com/Perufitlife/supabase-security-skill) · [npm](https://www.npmjs.com/package/supabase-security)

---

Want it checked for you before the deadline? [Open a free audit request](https://github.com/Perufitlife/awesome-backend-security/issues/new?template=free-audit.yml) with your project and I'll run it and send the report back — free.

← Back to [Awesome Backend Security Auditors](../)
