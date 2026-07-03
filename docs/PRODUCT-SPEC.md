# DEVELOPMENT-HELL — Product Spec

The Development module of **LEMON AI CENTER**. A development executive's command center:
every project on the slate, readable and queryable by the brain, pushed forward
proactively so nothing goes stale, with the existing skill library as the verbs.

Single user: Billy Rovzar, producer at Lemon Studios. UI and briefings in English;
material is bilingual (Spanish/English); outbound nudges drafted in the recipient's
language.

## 1. Architecture principle

**A module, not a standalone app.** DEVELOPMENT-HELL lives inside LEMON AI CENTER and
shares:

- **The brain** — the same LLM layer AI CENTER already uses (extended with the slate
  index and skills dispatch)
- **KNOWN_FACTS** — one source of truth; the module reads it for context and writes
  slate facts back into it
- **Auth** — whatever AI CENTER uses today; no second login

The exact module contract (routing, data layer, brain API) is filled in from
`RECON-REPORT.md` (produced by `prompts/HERMES-RECON.md`). Nothing in this spec may
contradict the recon; where they conflict, the recon wins and this spec gets amended.

## 2. The slate and its source of truth

The canonical `DEVELOPMENT/` folder (spec: `docs/FOLDER-STRUCTURE.md`) is the source of
truth for material. One folder per project, `project.yaml` for metadata, versioned
drafts. The module:

1. **Watches** the folder (filesystem watcher) — new files, new versions, edits.
2. **Parses deterministically** what the naming convention allows (project, stage,
   draft version, date).
3. **Falls back to the brain** for messy or ambiguous files ("this PDF looks like
   draft 3 of LA CASA, dated March — file it?" → one-click confirm).

### Project metadata (`project.yaml` — full schema in FOLDER-STRUCTURE.md)

| Field | Notes |
|---|---|
| `title`, `slug` | slug = folder name |
| `format` | `film` \| `series` |
| `stage` | see pipeline below |
| `origin` | `internal` \| `external` — external is firewalled (see §7) |
| `writers[]` | name, contact, language |
| `waiting_on` | person + since-date — powers the Waiting On briefing section |
| `priority` | `A` \| `B` \| `C` |
| `targets[]` | buyer/platform targets |
| `language` | `es` \| `en` \| `both` |
| `deadlines[]` | date + what — brain flags these ahead of staleness thresholds |
| `status` | `active` \| `paused` \| `dead` |

### Pipeline stages

- **Film:** `idea → concept → treatment → outline → draft1 → rewrites → polish → market-ready`
- **Series:** `idea → concept → bible → pilot-outline → pilot-draft → rewrites → season-arc → market-ready`

## 3. The slate index (the unfair advantage)

Not a status board — a queryable brain over the whole slate.

**Ingestion pipeline:** file appears → extract text (PDF, FDX, Fountain, docx, md/txt)
→ chunk (scene/section-aware for screenplays) → embed → store vectors + structured
metadata. Re-ingest on new versions; keep prior versions queryable for "what changed"
questions. Embedding store: whatever the recon says AI CENTER can host (pgvector /
Chroma / local files) — decided at build time.

**Queries it must answer** (acceptance tests, verbatim):

- "Which of my projects has the weakest second act?"
- "Find me something with a female lead in her 40s I can send to Apple."
- "What have I not touched in 60 days that has a finished draft?"
- "Which two projects are secretly the same movie?"

That means: semantic search across full text, structural analysis of screenplays
(act breaks, character demographics), structured filters (dates, stage, status), and
cross-project comparison — combined in one chat surface.

## 4. Skills are the verbs

The skill library (co-writer, story-ninja, lemon-coverage, dev-exec, film-finance,
chivo, + full inventory from recon) is fireable at any project:

- **Fire:** select project (and optionally a specific draft) → pick skill → run.
  Invocation mechanism comes from recon (Target 3: programmatic skill invocation).
- **Land:** results (coverage, dev notes, budget estimates, rewrite passes) are saved
  into the project's record — `coverage/` folder + indexed into the slate index — never
  lost in a chat scroll.
- **Learn:** every run is logged (skill, project, date, accepted/ignored) so the brain
  learns which skills Billy actually uses per stage and suggests them proactively
  ("draft 2 just landed — run lemon-coverage?").

## 5. Proactive engine

### Morning briefing (renders on module open; generated fresh, cached for the day)

1. **What Moved** — files/stages changed since last visit
2. **Going Stale** — staleness engine output, worst first
3. **Waiting On** — people owing things, with days elapsed
4. **Suggested Nudges** — pre-drafted messages (see below)
5. **Today's Pushes** — the brain's 1–3 concrete recommendations ("LA CASA has notes
   ready and the writer is free — send them today")

### Staleness engine

Days-since-last-touch per project, thresholds by stage (per-project override in
`project.yaml`):

| State | Default threshold |
|---|---|
| `idea` / `concept` | 30 days |
| `treatment` / `outline` / `bible` | 21 days |
| active drafting/rewrites | 7 days |
| out to writer | 14 days |
| out to buyer/platform | 10 days |
| `paused` | excluded (but resurfaced monthly: "still paused on purpose?") |

The brain applies judgment on top: an approaching `deadline` flags a project early
regardless of thresholds.

### Nudges

- Drafted in the recipient's language, in Billy's voice (voice profile learned from
  edits he makes to drafts — stored as brain memory).
- **Delivery:** always in-app with a copy button; when the recipient contact is an
  email address, also placed as a **Gmail draft** (never auto-sent).

## 6. Stage → app routing (deep integration)

The module knows which tool fits the moment, deep-links or hands files to it, and
watches for what comes back. Wiring per app comes from recon Target 2 §6.

| Moment | App |
|---|---|
| Brainstorming an idea | WR-AI-TERS ROOM AG |
| Idea → screenplay development | QUANTUMSTORY |
| Writing pages | BESTSCREEN |
| Rewrites / screenplay analysis | FRACTAL STORY ROOM |
| Script arrives from outside | LEMON-SCREENPLAY-DASHBOARD |
| Cleaning AI-flavored text | AI-DETECTOR |

Integration depth per app is whatever the recon reports as available, in order of
preference: read its data store directly → call its API → file-drop handoff → deep link.

## 7. External material firewall

`origin: external` projects are readable by the brain but:

- Marked visibly everywhere in the UI.
- **Never cross-pollinated:** brainstorms, rewrites, and skill runs on internal
  projects must not draw ideas from external material (enforced by excluding external
  chunks from retrieval on internal-project tasks).
- Slate-wide *status* queries include them; slate-wide *creative* queries exclude them
  unless explicitly asked.

## 8. Non-goals (v1)

- Multi-user / permissions
- Auto-sending any message
- Production/post tracking (development only, through market-ready)
- Mobile app (responsive web is enough)
