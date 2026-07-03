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

**Next:** Milestone 2 — onboarding wizard + scanner (folder per
`docs/FOLDER-STRUCTURE.md`; location saved to Firestore slate config +
surfaced via the vault note, per D2's KNOWN_FACTS replacement).
