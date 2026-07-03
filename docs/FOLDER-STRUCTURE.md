# The canonical DEVELOPMENT folder

The single source of truth for all material on the slate. Designed so a scanner can
parse it **deterministically**; the brain only guesses when a file breaks convention
(and then asks for one-click confirmation instead of silently filing it).

Default location: `~/DEVELOPMENT/` (configurable in the module's settings; stored in
KNOWN_FACTS so every tool agrees).

```
DEVELOPMENT/
├── LA-CASA-DEL-FUEGO/              ← one folder per project, slug in CAPS-KEBAB
│   ├── project.yaml                ← metadata (schema below) — required
│   ├── 01-idea/                    ← loglines, pitch one-pagers, voice memos, refs
│   ├── 02-treatment/               ← treatments & synopses (file naming below)
│   ├── 03-outline/                 ← outlines, beat sheets, bibles (TV)
│   ├── 04-drafts/                  ← screenplays, versioned (naming below)
│   ├── coverage/                   ← skill outputs: coverage, dev notes, budgets
│   ├── notes/                      ← Billy's and readers' notes
│   └── correspondence/             ← relevant emails/messages, exported
├── _external/                      ← other people's material (firewalled)
│   └── SUBMISSION-TITLE/           ← same structure inside
│       └── project.yaml            ← must carry origin: external
├── _archive/                       ← dead projects, moved whole (status: dead)
└── _inbox/                         ← unfiled drops; the module proposes filing
```

## File naming

**Drafts** (`04-drafts/`):

```
<SLUG>_v<NN>_<YYYY-MM-DD>[_<label>].<ext>
LA-CASA-DEL-FUEGO_v03_2026-07-01_polish-pass.fdx
LA-CASA-DEL-FUEGO_v03_2026-07-01_polish-pass.pdf     ← same version may exist in both
```

- `v01`, `v02`… strictly increasing; the scanner takes the highest as current.
- Accepted extensions: `.fdx` `.fountain` `.pdf` `.docx` `.md` `.txt`
- TV: pilot drafts use the same scheme; episodes add `_ep<NN>` after the version.

**Treatments/outlines** (`02-treatment/`, `03-outline/`):

```
<SLUG>_<treatment|synopsis|outline|bible>_v<NN>_<YYYY-MM-DD>.<ext>
```

**Coverage** (`coverage/`, written by the module itself):

```
<SLUG>_<skill>_<YYYY-MM-DD>.md          e.g. LA-CASA-DEL-FUEGO_lemon-coverage_2026-07-03.md
```

Anything that doesn't parse goes to the project's staleness record as "unfiled
material present" and shows up in the briefing until confirmed.

## `project.yaml` schema

```yaml
title: La Casa del Fuego            # display title
slug: LA-CASA-DEL-FUEGO             # = folder name; immutable
format: film                        # film | series
stage: rewrites                     # film:   idea|concept|treatment|outline|draft1|rewrites|polish|market-ready
                                    # series: idea|concept|bible|pilot-outline|pilot-draft|rewrites|season-arc|market-ready
origin: internal                    # internal | external
status: active                      # active | paused | dead
priority: A                         # A | B | C
language: es                        # es | en | both

logline: >
  One line. The scanner surfaces this everywhere.

writers:
  - name: María González
    contact: maria@example.com      # email enables Gmail-draft nudges
    language: es                    # nudges drafted in this language

waiting_on:                         # omit when waiting on nothing
  who: María González
  what: draft 4 with act-two notes addressed
  since: 2026-06-19

targets:                            # buyers/platforms in mind
  - Apple TV+
  - Netflix LatAm

deadlines:
  - date: 2026-08-15
    what: submission window for Morelia lab

staleness_days: 10                  # optional per-project override of stage default

notes: >
  Free text the brain reads for context.
```

Only `title`, `slug`, `format`, `stage`, `origin`, and `status` are required; the
module's onboarding wizard writes the file so nobody edits YAML by hand unless they
want to.

## Rules the module enforces

1. `slug` == folder name, always. Renames go through the module (it updates both).
2. `_external/` projects: `origin: external` is mandatory and cannot be changed in
   place — moving material from external to internal is an explicit, logged action.
3. The module never deletes; "dead" projects move to `_archive/`.
4. Every file event (add/replace) updates the project's `last_touched`, which drives
   the staleness engine.
