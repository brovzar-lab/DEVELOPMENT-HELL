# DEVELOPMENT-HELL

The Development module of **LEMON AI CENTER** — a command center for the entire
development slate at Lemon Studios. Every project from idea to market-ready screenplay,
embedded and queryable by the brain, pushed forward proactively so nothing goes stale,
with the existing skill library (lemon-coverage, story-ninja, dev-exec, film-finance,
co-writer, chivo, …) as the verbs.

This repo currently contains the **launch kit**: the product spec and the two prompts
that get the module built against the real infrastructure instead of guesses.

## Launch sequence

### ① Recon — ✅ DONE (both parts)
Part 1 ran on the Hermes VPS (embedded in `prompts/MASTER-BUILD-PROMPT.md`); Part 2
ran on the Mac (`prompts/RECON-REPORT-MAC`) and supersedes Part 1 where they conflict.
Headlines: **LEMON-AI-CENTER exists** (ceo.billyrovzar.com, Railway,
`brovzar-lab/lemon-ai-center`); its brain is FlexSearch over the Obsidian vault (no
embeddings — we add the vector layer); KNOWN_FACTS doesn't exist (replaced per D2);
the six named skills need consolidation (D6). Binding resolutions:
`docs/ARCHITECTURE-DECISIONS.md`.

### ② Build — Fable 5 ← YOU ARE HERE
Open a Fable 5 (Claude Code) session with this repo **and**
`brovzar-lab/lemon-ai-center` (locally on the Mac, or remotely via `add_repo`). Paste
`prompts/MASTER-BUILD-PROMPT.md` (below the divider) as the opening prompt. Nine
milestones, working software at every step.

### ③ Onboard the slate
The module's wizard creates the canonical `DEVELOPMENT/` folder
(`docs/FOLDER-STRUCTURE.md`) and you move projects in. From then on the folder is the
source of truth and the module watches it.

## Repo map

| File | What it is |
|---|---|
| `docs/PRODUCT-SPEC.md` | The PRD — binding for the build |
| `docs/FOLDER-STRUCTURE.md` | Canonical DEVELOPMENT folder + `project.yaml` schema |
| `prompts/HERMES-RECON.md` | The recon mission for Hermes (step ①) |
| `prompts/MASTER-BUILD-PROMPT.md` | The Fable 5 build prompt (step ②) |

## Principles

- **A module, not another app** — shares AI CENTER's brain, KNOWN_FACTS, and auth.
  One source of truth, no split-brain.
- **Queryable brain, not a status board** — the material itself is embedded, so the
  slate answers questions like "which two projects are secretly the same movie."
- **Skills are the verbs** — select a project, fire a skill, the result lands in the
  project's record.
- **Proactive by default** — morning briefing, staleness engine, pre-drafted nudges.
  Nothing dies quietly in development hell.
