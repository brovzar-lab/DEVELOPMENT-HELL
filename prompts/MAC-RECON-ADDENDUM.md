# MAC RECON ADDENDUM — the half the VPS couldn't see

> **Why this exists:** The first recon (`prompts/HERMES-RECON.md`) was executed by
> Hermes **on its Linux VPS** (`/opt/data`, 187.124.251.98). It verified the Hermes
> ecosystem thoroughly, but it physically could not see `/Users/quantumcode/CODE` —
> so "LEMON AI CENTER does not exist" is only proven for the VPS. This addendum runs
> **on the Mac** and covers only the gaps.
>
> **How to use:** open Claude Code on the Mac (terminal: `cd /Users/quantumcode/CODE
> && claude`) and paste everything below the line. Append its output to
> `RECON-REPORT.md` as "Part 2 — Mac".

---

You are completing a reconnaissance mission. Part 1 already verified the Hermes agent
ecosystem on a remote VPS. Your job is ONLY the Mac side. Same rules: every claim
tagged `✅ VERIFIED` (show the command + output snippet) or `⚠️ ASSUMED`; exact paths;
read-only; a thing not existing is a finding, not a failure. Output:
`RECON-REPORT-MAC.md`.

## Gap 1 — Does LEMON AI CENTER exist on this Mac?

Part 1 found no such app on the VPS. Search here: `ls /Users/quantumcode/CODE`, then
look for anything named like LEMON AI CENTER (also LEMON-AI-CENTER, AICENTER,
lemon-center…). If it exists: full Target-1 treatment — stack, run it (`dev`), module
pattern with a real code example, its brain (model, API layer, key location), any
KNOWN_FACTS file/store (exact path, format, real redacted example), auth, database,
any existing embeddings/vector search. If it does NOT exist, say so flatly — that
decision forks the whole architecture.

## Gap 2 — The six apps (full Target-2 treatment, now with real access)

For QUANTUMSTORY, BESTSCREEN, FRACTAL STORY ROOM, WR-AI-TERS ROOM AG,
LEMON-SCREENPLAY-DASHBOARD, AI-DETECTOR — exact folder name, GitHub repo (check
`git remote -v` in each), stack, launch command (verify at least that deps install /
it starts), where data lives (Firebase project ids from config files — Part 1 saw a
`service-account.json` referenced in LEMON-SCREENPLAY-DASHBOARD), file formats in/out,
integration surface (API/CLI/exported functions/watched folders), auth/keys (paths
only, no secrets). Also list any OTHER apps in CODE that look development-relevant —
Part 1 saw a "MATADERO-RW" mentioned.

## Gap 3 — Skill-name discrepancy

Billy's list: **co-writer, story-ninja, lemon-coverage, dev-exec, film-finance,
chivo**. Part 1's verified inventory (60+ skills on the VPS) contains none of these
names — closest matches were comedy-screenwriting, ip-scouting, jen-grisanti,
logline-crafter, production-templates. Search the Mac for skills too
(`~/.claude/skills`, project `.claude/` dirs, anywhere in CODE). Report: do Billy's
six named skills exist anywhere (Mac or renamed on the VPS), or are they aspirational?

## Gap 4 — Mac tooling and cloud auth

Which CLIs are installed AND logged in on the Mac: `claude`, `firebase`, `vercel`,
`railway`, `gcloud`, `gh` (versions; auth status via `firebase projects:list`,
`vercel whoami`, `railway whoami`, `gcloud config list`, `gh auth status`). This is
where the Firebase/Vercel/Railway/Hostinger/Google deployment map (Target 4) can
actually be verified — build the app→host→URL→repo table from what these CLIs return
plus each app's config files.

## Gap 5 — Can the Mac reach Hermes?

Part 1 verified Hermes's gateway on the VPS and Paperclip at
https://paperclip.billyrovzar.com. From the Mac: is there any client, API endpoint,
or token that lets a local process talk to the Hermes gateway (check for API docs,
ports, reverse proxies, Telegram-only)? This decides whether a Mac-side dashboard can
use Hermes as its brain or needs its own LLM layer.
