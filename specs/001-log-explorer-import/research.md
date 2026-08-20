# Phase 0 Research: Log Explorer — Import & Search

> `/speckit-plan` Phase 0 output — regenerate via the command, don't hand-edit.

No unresolved `NEEDS CLARIFICATION` markers — the user's instructions fixed
the stack directly. Decisions below, one line each.

- **File reading**: `file.stream()` decoded inside the worker, never on the
  main thread — avoids loading the whole file into memory before parsing.
- **Worker protocol**: fixed message set (`start` / `progress` / `done` /
  `error`), no Comlink — simplest typed contract for one worker doing one
  job (see [contracts/worker-messages.md](./contracts/worker-messages.md)).
- **IndexedDB writes**: batch of N lines per `readwrite` transaction,
  reported after `oncomplete` — balances progress granularity vs
  transaction overhead; store is cleared before each new import.
- **IndexedDB access**: native API only, no `idb`/Dexie — explicit ask,
  the whole point is learning raw transaction/store/index/cursor mechanics.
- **Queries**: cursor + keyrange via index, never `getAll()` + JS filter —
  idiomatic IndexedDB pattern, scales as data grows.
- **Styling**: Tailwind CSS v4 via `@tailwindcss/vite` — fastest way to
  translate the Claude Design mockups (sent separately) without adding a
  component library.
- **Linting**: ESLint flat config (`typescript-eslint` + `react-hooks` +
  `react-refresh`) — same set Vite's official `react-ts` template uses.
- **Import alias**: `@/*` → `src/*` in both `tsconfig.json` and
  `vite.config.ts` — avoids deep relative imports in nested feature folders.
- **Testing**: Vitest + `fake-indexeddb` for parser/query logic; real
  Worker + real IndexedDB validated manually via
  [quickstart.md](./quickstart.md) — jsdom/Node can't run a real Worker.
