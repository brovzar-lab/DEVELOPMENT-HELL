# MASTER BUILD PROMPT — DEVELOPMENT-HELL

> **How to use this file:** Run a Fable 5 session (Claude Code) with access to BOTH
> this repo and the LEMON AI CENTER codebase (locally on the Mac, or remotely after
> `add_repo`-ing the repos named in the recon report). Fill the `<RECON-REPORT>` slot,
> then paste everything below the line as the opening prompt.

---

You are building **DEVELOPMENT-HELL**: the Development module of LEMON AI CENTER, a
command center for Billy Rovzar, producer at Lemon Studios. It tracks every project on
his development slate, embeds all the material so the whole slate is queryable, fires
his existing skill library at projects, and proactively keeps projects from going
stale. This is a power tool meant to be an unfair advantage — build it like one.

## Ground truth — read these first, in this order

1. `docs/PRODUCT-SPEC.md` — the full PRD. It is binding: pipeline stages, metadata
   schema, staleness thresholds, briefing format, external-material firewall,
   acceptance-test queries.
2. `docs/FOLDER-STRUCTURE.md` — the canonical DEVELOPMENT folder and `project.yaml`
   schema the scanner must parse.
3. The recon report below — the verified map of the real infrastructure. **Where the
   spec and the recon conflict, the recon wins**; flag the conflict and amend the spec
   in the same commit.

If either doc is missing from your working tree, stop and say so — do not rebuild the
spec from memory.

## Recon report

<RECON-REPORT>
PASTE THE FULL RECON-REPORT.md PRODUCED BY prompts/HERMES-RECON.md HERE.
If it is saved as a file, replace this block with its path and read it.
</RECON-REPORT>

## Architecture constraints (non-negotiable)

- **A module inside LEMON AI CENTER** — follow AI CENTER's existing module/section
  pattern exactly as the recon documents it. Match its stack, its routing, its styling.
  No parallel app, no second server unless the recon shows AI CENTER cannot host a
  background worker (then: one small companion daemon, nothing more).
- **One brain.** Extend AI CENTER's existing LLM layer; do not add a second AI stack.
- **One source of truth.** Read and write KNOWN_FACTS in the exact format the recon
  documents. Slate-level facts (folder location, project list, stage changes) belong
  in KNOWN_FACTS so every other tool sees them.
- **Same auth.** Sit behind whatever AI CENTER already uses.
- **Skills, not reimplementations.** Coverage, dev notes, budgets, rewrites are the
  existing skills (lemon-coverage, dev-exec, film-finance, story-ninja, co-writer,
  chivo, …) invoked programmatically the way the recon verified. Never reimplement a
  skill's job inline.

## Build order — one milestone per commit, working software at every step

1. **Module shell** — DEVELOPMENT-HELL mounted in AI CENTER's nav, behind its auth,
   rendering an empty slate view. Prove the module pattern works before anything else.
2. **Onboarding wizard + scanner** — wizard creates the `DEVELOPMENT/` folder per the
   structure doc (location saved to KNOWN_FACTS); filesystem watcher + deterministic
   parser populate the slate; unparseable files land in a confirm queue.
3. **Slate board** — the visual pipeline: projects as cards in stage columns (film and
   series lanes), showing priority, staleness heat, waiting-on, external badge.
4. **Ingestion + slate index** — extract text (PDF/FDX/Fountain/docx), scene-aware
   chunking, embeddings in the store the recon recommends. Re-index on file change.
5. **Query chat** — the brain over the slate index + metadata. It must pass the four
   acceptance queries in the spec §3 verbatim. External firewall (spec §7) enforced in
   retrieval from day one.
6. **Skills dispatch** — project × skill firing surface; results land in `coverage/`
   and the index; runs logged.
7. **Briefing engine** — the five-section morning briefing on module open; staleness
   thresholds per spec §5.
8. **Nudges** — drafted per recipient language, in-app copy button; Gmail drafts when
   the contact has email (never auto-send).
9. **App integrations** — stage→app routing per spec §6, wired per app at the depth
   the recon verified (data-store read > API > file-drop > deep link).

## Quality bar

- Zero-config start: after the wizard, everything works with no manual file editing.
- No mock data anywhere; the empty state is a designed onboarding, not lorem ipsum.
- Briefing generation and ingestion run in the background — the UI never blocks on
  the brain.
- Bilingual reality: parsing, embedding, and chat must handle Spanish and English
  material equally; UI stays English.
- Every destructive-looking action (refile, archive, external→internal) is explicit,
  confirmed, and logged.
- At each milestone: run it, drive the real flow end-to-end, and show what you saw
  before moving on.

Start with milestone 1. Before writing code, restate in ten lines how AI CENTER's
module pattern, brain layer, and KNOWN_FACTS work according to the recon — if you
can't, the recon is insufficient and you should say exactly what's missing instead of
guessing.
