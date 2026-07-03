# Architecture decisions — reconciling the spec with the recon

Recon sources: `prompts/MASTER-BUILD-PROMPT.md` (embedded VPS recon, Part 1) and
`prompts/RECON-REPORT-MAC` (Part 2). Rule from the spec: **where spec and recon
conflict, the recon wins.** These are the resolutions. The build prompt and spec are
read *through* this file.

## D1. The host app is real: LEMON-AI-CENTER

`/Users/quantumcode/CODE/LEMON-AI-CENTER` · repo `brovzar-lab/lemon-ai-center` ·
live at https://ceo.billyrovzar.com on Railway. Vite+React+TS+Zustand front, Express
(:3001) back, Firebase/Firestore, Google OAuth (allow-list), Anthropic SDK brain,
"Mission Control Engine" cron jobs. **DEVELOPMENT-HELL is built as a new domain inside
this app**: routes in `server/routes/`, libs in `server/lib/`, one Zustand store in
`src/stores/`, behind the existing `requireAuth` + `csrfCheck`. The VPS-recon
recommendation to build it as a Hermes skill is rejected — it was made without
visibility into the Mac.

## D2. "KNOWN_FACTS" does not exist — here is what replaces it

Verified: no KNOWN_FACTS file/store anywhere. The module's "one source of truth"
becomes two real substrates it writes to:

1. **Firestore** (AI CENTER's primary DB): new `slate/*` collections for project
   metadata, stage history, staleness state, skill-run logs.
2. **The Obsidian vault (`OBSIDIAN BRAIN`)**: the module writes/updates one markdown
   status note per project into the vault. AI CENTER's existing FlexSearch brain and
   `seedFromVault` engine job then see slate facts automatically — integration with the
   existing brain for free, using the app's own pattern.

The `corrections` loop is reused as-is for "the AI got this project's status wrong."

## D3. The brain is AI CENTER's Anthropic layer — Hermes is not the brain

- Every AI call goes through the existing `getAnthropicClient()` pattern
  (`server/lib/anthropic.ts`), with tools declared like `chatTools.ts`.
- Verified: there is **no wired path from the Mac (or Railway) to the VPS Hermes
  gateway** — no endpoint, no token. The local Mac `hermes` CLI (grok-4.3) exists but
  is unpaired and adds a second AI stack for no benefit. Hermes integration is
  **out of scope for v1**; revisit if the Quantum_Hermes templates ever get filled in.
- Note for the build: AI CENTER hardcodes previous-generation model IDs
  (`claude-sonnet-4-6`, `claude-haiku-4-5-*`). The module uses current models and a
  small shared model-id constant; refreshing the rest of the app is a separate,
  optional one-line-per-file cleanup.

## D4. The Mac-folder vs Railway-cloud gap → reuse the proven ingest pattern

The DEVELOPMENT/ folder lives on the Mac; AI CENTER runs on Railway. A cloud server
cannot watch a Mac folder. LEMON-SCREENPLAY-DASHBOARD already solved exactly this:
Storage upload trigger (`onScreenplayUploaded`) + Firestore `ingest-queue` + a polling
daemon. So:

- **Mac side:** a small watcher daemon (the only new process outside AI CENTER;
  pattern copied from LEMON-SCREENPLAY-DASHBOARD's `daemon.py`) watches `DEVELOPMENT/`,
  uploads new/changed files to Firebase Storage, and enqueues ingest jobs in Firestore.
- **Cloud side:** the module processes the queue — extract text, chunk, embed, index —
  and updates `slate/*`.
- When AI CENTER is run locally (`npm run dev`), the module MAY watch the folder
  directly (chokidar is already in the codebase for the vault watcher); the queue is
  the sync mechanism either way, so local and cloud sessions see the same slate.

## D5. Semantic search must be added — there is nothing to reuse

Verified: AI CENTER's brain is FlexSearch keyword search; no embeddings anywhere in
the fleet. The slate index therefore adds the fleet's first vector layer:

- Chunks + vectors stored in **Firestore with its vector/KNN support**; fall back to a
  simple stored-array + server-side cosine scan if the Firestore feature isn't
  available on the current plan (slate scale is thousands of chunks, not millions —
  a scan is fine).
- Embedding provider decided at build time from what's already paid for: Gemini
  embeddings (Gemini keys exist in the fleet) or Voyage — NOT a new heavyweight
  dependency. Anthropic has no embeddings API.
- FlexSearch stays for keyword search; the query chat uses both (hybrid retrieval).

## D6. The six named skills get consolidated, not assumed

Verified: co-writer, story-ninja, lemon-coverage, dev-exec exist only as
project-scoped/bundle files; film-finance and chivo don't exist. The real library is
~500 SKILL.md files (MASTER-LIBRARY, lemon-studio-skills, upload bundles). Therefore
milestone "Skills dispatch" begins with a **consolidation step**:

1. Create `LEMON-AI-CENTER/skills/` (or the location the build finds most idiomatic)
   as the canonical Lemon development skill set.
2. Copy in the four that exist (best version wins: `QUANTUMSTORY/.agent/skills/
   lemon-coverage`, `LEMON-SCREENPLAY-DASHBOARD/agent/skills/dev-exec`,
   `WR-AI-TERS ROOM AG/screenwriting skills/co-writer`, bundle `story-ninja`).
3. `film-finance` and `chivo` are created new (seeded from `finance-lead`/`cfo-advisor`
   in MASTER-LIBRARY and the matadero-cinematography vault notes respectively) — Billy
   reviews their SKILL.md before they go live.
4. The module's skill runner executes a skill = load SKILL.md as system context +
   project material into the existing Anthropic layer. No dependency on Claude Code
   being installed on the server.

## D7. App integration wiring (now known, per app)

| App | v1 integration (verified surface) |
|---|---|
| LEMON-SCREENPLAY-DASHBOARD | Read its Firestore (`lemon-screenplay-dashboard`) for incoming coverage/scripts; its ingest pattern is D4's template |
| AI-DETECTOR | Real HTTP API: `POST /api/detect`, `/api/humanize` — call directly (local service) |
| FRACTAL STORY ROOM | Firestore `fractal-story-room` read + deep link |
| QUANTUMSTORY | Firestore `screenpartner-16217665-33d08` read + deep link |
| WR-AI-TERS ROOM AG | Firestore `wr-ai-ters-room` read + deep link |
| BESTSCREEN | localStorage-first (no server data to read) → file-drop into DEVELOPMENT/ + deep link |

Known hazard: BESTSCREEN and WR-AI-TERS ROOM AG share the `wr-ai-ters-room` Firebase
project — never assume a collection belongs to the app its name suggests; check before
writing.

## D8. Deployment facts that gate the build

- GitHub org `brovzar-lab` (gh CLI authenticated); Firebase/gcloud/Railway all
  authenticated as billyrovzar@gmail.com on the Mac. **Vercel and Hostinger are not
  part of this stack** — ignore them.
- Build sessions need the `brovzar-lab/lemon-ai-center` repo (and this one). Remote
  Claude sessions: add it with `add_repo`.
- MATADERO-RW calls Anthropic **directly from the browser client** — flag to Billy;
  if that app ever deploys, the key leaks. Not this module's job to fix, but worth a
  ticket.
