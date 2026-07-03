# DEVELOPMENT-HELL

The Development module of **LEMON AI CENTER** — a command center for the entire
development slate at Lemon Studios. Every project from idea to market-ready screenplay,
embedded and queryable by the brain, pushed forward proactively so nothing goes stale,
with the existing skill library (lemon-coverage, story-ninja, dev-exec, film-finance,
co-writer, chivo, …) as the verbs.

This repo currently contains the **launch kit**: the product spec and the two prompts
that get the module built against the real infrastructure instead of guesses.

## Launch sequence

### ① Recon — run once, on the Mac
Paste `prompts/HERMES-RECON.md` into Hermes (or Claude Code at
`/Users/quantumcode/CODE`). It verifies — not summarizes — LEMON AI CENTER's module
architecture, brain, and KNOWN_FACTS; the six development apps and their data; the
skill library and how to fire it programmatically; and the full deployment map across
Firebase / Vercel / Railway / Hostinger / Google. Output: `RECON-REPORT.md`.

### ② Build — Fable 5
Open a Fable 5 (Claude Code) session with this repo **and** the LEMON AI CENTER
codebase available (locally, or remotely after adding the repos named in the recon
report). Paste the recon report into `prompts/MASTER-BUILD-PROMPT.md` and run it.
It builds in nine milestones, working software at every step.

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
