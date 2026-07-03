# HERMES RECON MISSION — Infrastructure Verification for the DEVELOPMENT-HELL Module

> **How to use this file:** Paste everything below the line into Hermes (or a Claude Code
> session on the Mac with access to `/Users/quantumcode/CODE`). The output it produces —
> `RECON-REPORT.md` — is the input for building the Development module of LEMON AI CENTER.

---

You are running a reconnaissance and verification mission on this machine and on the
cloud services connected to it. Your findings will be used by another engineering agent
to build **DEVELOPMENT-HELL**, a Development module that plugs into LEMON AI CENTER,
shares its brain and KNOWN_FACTS, embeds the entire development slate for semantic
queries, and fires the existing skill library at projects.

That agent cannot see this machine. It will trust your report completely. Therefore:

## Rules of engagement

1. **Verify, don't summarize.** Do not report what a README says. Run the command, open
   the file, query the database, hit the endpoint. Every factual claim in your report
   must be tagged:
   - `✅ VERIFIED` — you executed something and saw it (include the command/query and a
     snippet of its output)
   - `⚠️ ASSUMED` — you inferred it but could not confirm (say why not)
2. **Exact paths, exact names.** Absolute file paths, exact collection names, exact env
   var names (values redacted for secrets — report `FIREBASE_KEY=<present>`), exact
   URLs, exact repo names.
3. **Read-only mission.** Do not modify, migrate, or clean anything. If you find
   something broken or contradictory, report it — don't fix it.
4. **If something doesn't exist, that's a finding.** "There is no KNOWN_FACTS file; the
   closest equivalent is X" is more valuable than silence.
5. Deployments live across **Firebase, Vercel, Railway, Hostinger VPS, and Google
   (Cloud/Drive)**. Check CLI auth state where available (`firebase projects:list`,
   `vercel ls`, `railway status`, `gcloud config list`) and report which CLIs are
   installed and logged in.

## Target 1 — LEMON AI CENTER (priority one, be exhaustive)

The module will live inside this app, so its architecture decides everything.

- Local path and GitHub repo. Deployed where (URL, host)?
- Stack: framework, language, package manager, build/run commands (verify `dev` runs).
- **Module architecture:** how are sections/modules added today? Routing structure,
  a modules directory, a plugin registry? Show the pattern with a real example from the
  code.
- **The brain:** which model/API does it call (Anthropic? which model id?), through what
  layer (raw API, Agent SDK, LangChain, custom)? Where is the system prompt? Is there
  streaming? Where do API keys live?
- **KNOWN_FACTS:** exact location(s), format (file? Firestore collection? table?),
  schema/shape (paste a real, redacted example), and every code path that reads or
  writes it.
- **Auth:** how does Billy log in (Firebase Auth? none? basic)? What would a new module
  need to do to sit behind the same auth?
- **Data layer:** which database(s), which Firebase project id(s), collection names,
  any existing "projects" or "slate"-like data.
- **Embeddings/search:** does any vector or semantic search already exist anywhere in
  the system? If yes, what store (pgvector, Firestore ext, local files, Chroma…)?

## Target 2 — The six development apps

For each of: **QUANTUMSTORY**, **BESTSCREEN**, **FRACTAL STORY ROOM**,
**WR-AI-TERS ROOM AG**, **LEMON-SCREENPLAY-DASHBOARD**, **AI-DETECTOR** — report:

1. Local path in `/Users/quantumcode/CODE` (exact folder name) and GitHub repo.
2. Deployed URL and host (Firebase/Vercel/Railway/Hostinger/Google) — verify the URL
   responds.
3. Stack and how it's launched locally.
4. **Where its data lives**: DB + project/collection/table names, or file directories.
   What does a "project"/"script"/"session" record look like? Paste one redacted example.
5. **File formats** it consumes and produces (PDF, FDX, Fountain, docx, JSON…) and
   where output files land on disk or in cloud storage.
6. **Integration surface**: any HTTP API, CLI, exported functions, MCP server, webhook,
   or watched folder another app could use to (a) read its data, (b) send it work,
   (c) get results back. If none exists, say what the least-invasive option would be
   (e.g. "reads Firestore collection X directly" or "drop file in folder Y").
7. Auth/keys it needs.

## Target 3 — Hermes itself + the skill library

- What is Hermes, concretely: which runtime (Claude Code? custom agent?), where do its
  persona/config files live, what memory does it read/write, does it read KNOWN_FACTS?
- **Full skill inventory**: list every skill (expected among them: co-writer,
  story-ninja, lemon-coverage, dev-exec, film-finance, chivo — find the rest). For each:
  exact path, format (SKILL.md? script? prompt file?), what it does in one line, and
  what inputs it expects (a script file? a logline? a folder?).
- **Programmatic invocation:** how can another process fire a skill non-interactively?
  (e.g. `claude -p "/lemon-coverage <file>"`, an SDK call, a queue?) Verify by actually
  invoking one harmless skill non-interactively and showing the output.

## Target 4 — Deployment & account map

- Table: app → host → URL → GitHub repo → Firebase/GCP project id → who owns the account.
- Which CLIs are installed and authenticated on this Mac (firebase, vercel, railway,
  gcloud, gh, claude). Versions.
- Where service-account files / `.env` files live for each app (paths only, no secrets).
- Anything that looks like it would break if a new module started reading the same data
  (rate limits, quotas, free-tier limits worth knowing).

## Output format — write `RECON-REPORT.md`

```
# RECON REPORT — <date>
## 0. Executive summary (10 lines max: the shape of the empire, biggest surprises, biggest risks)
## 1. LEMON AI CENTER
## 2. Six apps (one subsection each)
## 3. Hermes + skill library (with full skill inventory table)
## 4. Deployment & account map (table)
## 5. Contradictions, dead code, and things that are not what they claim to be
## 6. Recommendations: the three cheapest integration paths for a Development module,
##    ranked, with the evidence for each
```

Every claim tagged `✅ VERIFIED` or `⚠️ ASSUMED`. Verified claims show the command and
a short output snippet. Be ruthless about the difference — a wrong "verified" claim
will send the build down a dead end.
