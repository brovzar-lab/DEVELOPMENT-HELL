# Build log — where each milestone landed

App code lives in `brovzar-lab/lemon-ai-center`, branch
`feature/development-hell-module`. One milestone per commit. This log records
the *implemented* module contract so later build sessions don't re-derive it.

## Milestone 1 — module shell (`d78e3d7`, 2026-07-02)

DEVELOPMENT-HELL mounted inside LEMON-AI-CENTER, behind its auth, rendering a
designed empty slate view. Verified end-to-end: typecheck, vitest, build, dev
boot, 401 on unauthenticated API, empty state in the browser, `g 0` shortcut.

**Where things landed (the module contract, as built):**

| Piece | Location |
|---|---|
| Slate API | `server/routes/slate.ts` → `app.use('/api/slate', slateRouter)`; router-level `requireAuth`; `{ data } / { error: {code,message,retryable} }` envelope |
| Firestore access (D2) | `server/lib/slate/index.ts` — collection `slate`, doc id == slug == folder name; admin SDK only, client never reads it directly |
| Slate types | `shared/types.ts` — `SlateProject` + stage/origin/status unions mirroring the `project.yaml` schema; `SLATE_FILM_STAGES` / `SLATE_SERIES_STAGES` |
| Model ids (D3) | `shared/models.ts` — `CLAUDE_MODELS.smart = 'claude-opus-4-8'`, `.fast = 'claude-haiku-4-5'`; every module AI call uses these |
| Store | `src/stores/useSlateStore.ts` — fetch via `apiFetch('/api/slate/projects')`; **no seeds** (empty state is designed onboarding, quality bar says no mock data); 401 → empty slate, not an error |
| View | `src/components/views/DevHellView.tsx`, rendered from `Dashboard.tsx` when `view === 'devhell'` |
| Nav | `ViewId 'devhell'` in `src/stores/useViewStore.ts`; tab "Dev Hell" in `WorkspaceTabs.tsx` (after Projects), shortcut `g 0`, count = active slate projects |
| Tests | `server/routes/slate.test.ts` (supertest + vi.mock of the lib, per `brain.test.ts` pattern) |

**Facts discovered during the build (recon deltas, none contradicting D1–D8):**

- The nav is not a router: views are a `ViewId` union switched by
  `useViewStore` (localStorage-persisted) and rendered from a ternary chain in
  `Dashboard.tsx`, gated by `VITE_NEW_DASHBOARD` + `VITE_OPS_VIEWS` env flags
  (both `true` in `.env`). New views need: union entry, tab, keyboard map
  entry, Dashboard branch.
- Unauthenticated, the app runs in demo mode (`useAuthStore.isDemo`); module
  views must degrade to their empty state there, not an error banner.
- Vite dev runs on **5175** (not 5173; 5173 is LEMON-AIVO's) proxying
  `/api`+`/auth` → Express `:3001`. Express reads `PORT` env — pin `PORT=3001`
  if a tool injects `PORT`.
- Styling per `DESIGN.md` in the repo root (Instrument system): fills +
  `shadow-card`, no hairline-only cards, one cobalt accent, Playfair display
  for the view title only.
- Pre-existing test failure on `main` (also on the feature branch, untouched
  by the module): `src/__tests__/Header.test.tsx` "Refresh all data" button.
  Not ours; flagged for a separate fix.

## Milestone 2 — onboarding wizard + scanner (`0349e2a`, 2026-07-02)

Wizard creates `DEVELOPMENT/` (+ `_external`, `_archive`, `_inbox`), saves the
location to Firestore `slate_config/settings` (D2), runs the first scan, starts
the watcher. Scanner populates `slate/*` deterministically; convention breaks
land in `slate_confirm` instead of silent filing. Verified live against real
Firestore (fixture slate: scan, firewall, archive→dead, confirm queue, vault
notes, watcher picked up a dropped draft; cleanup left Firestore virgin).

**Where things landed:**

| Piece | Location |
|---|---|
| Parser | `server/lib/slate/parser.ts` — project.yaml validation (slug==folder, stage↔format), draft `<SLUG>_v<NN>[_ep<NN>]_<YYYY-MM-DD>[_label].<ext>`, doc `<SLUG>_<treatment\|synopsis\|outline\|bible>_v<NN>_<date>`, coverage `<SLUG>_<skill>_<date>.md` |
| Scanner | `server/lib/slate/scanner.ts` — pure `scanDevelopmentFolder()` + Firestore sync (disk wins, replace semantics); `runSlateScan()` is the single writer of slate state |
| Rules as built | `_external/` placement forces `origin: external` + queues the inconsistency (firewall never weakens); `_archive/` forces `status: dead`; `_inbox/` + loose/bad-named files → `slate_confirm` (stable sha1-of-path ids); free-form folders: `01-idea`, `notes`, `correspondence`; `current_draft` = highest version; `last_touched` = max mtime |
| Config (D2) | `server/lib/slate/config.ts` → `slate_config/settings` `{ devFolderPath, onboardedAt, lastScanAt }` |
| Vault notes (D2) | `server/lib/slate/vaultNote.ts` → `<vault>/slate/<SLUG>.md`, content-diffed, module-owned folder only; picked up by FlexSearch brain + `seedFromVault` automatically |
| Watcher (D4) | `server/lib/slate/watcher.ts` — chokidar, 1.2s debounce + awaitWriteFinish, full rescan per event; `initSlateWatcher()` at boot (catch-up scan); off when folder unreachable (Railway) |
| Routes | `GET /api/slate/status\|projects\|confirm`, `POST /api/slate/onboard\|rescan` (csrfCheck = Origin allow-list; localhost passes in dev) |
| UI | `DevHellView`: wizard (default `~/DEVELOPMENT`, demo mode disables the action) → onboarded dashboard (folder chip, watcher badge, rescan, confirm queue, project rows with priority/vNN/waiting-on) |
| Deps added | `js-yaml` + `@types/js-yaml` (both in `dependencies` — Railway `npm ci` skips devDeps) |

**Deltas/notes:** filing *actions* on confirm-queue items (move/rename/one-click
brain proposals) are deliberately not in M2 — the queue clears itself when the
file is fixed on disk; actions belong with a later milestone. E2E note: real
Google auth can't run headless, so the authenticated flow was driven at the lib
layer against real Firestore + a scratch vault; the browser verified wizard
rendering and demo-mode degradation.

**Next:** Milestone 3 — the slate board (cards in stage columns, film/series
lanes, priority, staleness heat, waiting-on, external badge).
