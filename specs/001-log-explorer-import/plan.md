# Implementation Plan: Log Explorer — Import & Search

**Branch**: `001-log-explorer-import` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-log-explorer-import/spec.md`

## Summary

Front-end-only log explorer. A large log file is read and parsed line-by-line
inside a dedicated Web Worker, which batches parsed entries and writes them
into IndexedDB via the native `indexedDB` API, reporting progress per batch
back to the main thread over `postMessage`. The UI (React + TS) shows import
progress, then lets the user search/filter/paginate the persisted entries —
which survive a page reload because they live in IndexedDB, not memory.

## Technical Context

**Language/Version**: TypeScript 5.x (strict), Node.js 24+ (tooling only)

**Primary Dependencies**: React 19, Vite 6.x, native `indexedDB` + `Worker` APIs (no wrapper lib, no state-management lib). Tailwind CSS v4 (`@tailwindcss/vite`), ESLint flat config (`typescript-eslint` + `react-hooks` + `react-refresh`), `@/*` → `src/*` import alias

**Storage**: IndexedDB — one database, two object stores: `entries` and `meta` (holds the imported file's name, so it can be shown again after a reload) (see [data-model.md](./data-model.md))

**Testing**: Vitest + `fake-indexeddb` for unit-testable logic; manual browser validation via [quickstart.md](./quickstart.md) for the real Worker+IndexedDB path

**Target Platform**: Modern evergreen browsers, via Vite dev server or static build

**Project Type**: Single-page web application (front-end only)

**Performance Goals**: Import never blocks the main thread; batches sized so progress reports at least every ~1-2s

**Constraints**: 100% client-side; parsing/writing only in the Worker; IndexedDB only for bulk data; strict TS, no unexplained `any`; no speculative abstraction

**Scale/Scope**: Single-user, single-tab, one imported file at a time (new import replaces old data)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Check | Status |
|---|---|---|
| I. Front-End Only | No backend anywhere; Vite dev server + static build only | PASS |
| II. Learning-First Scope | Every piece serves import/parse/store/search | PASS |
| III. Worker-First | Parsing + batch writes run only in `log-import.worker.ts` | PASS |
| IV. IndexedDB as System of Record | Entries in IndexedDB; no localStorage for bulk data | PASS |
| V. Strict TypeScript | Strict mode; worker/IndexedDB payloads fully typed | PASS |
| VI. Simplicity (YAGNI) | No IndexedDB wrapper, no state-mgmt lib, no plugin system | PASS |

No violations, before or after Phase 1 design. Complexity Tracking not needed.

## Project Structure

### Documentation (this feature)

```text
specs/001-log-explorer-import/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md         # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
│   └── worker-messages.md
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created here)
```

### Source Code (repository root)

```text
index.html
vite.config.ts        # @tailwindcss/vite plugin + "@" -> "src" resolve.alias
tsconfig.json         # strict: true; paths: { "@/*": ["./src/*"] }
eslint.config.js       # flat config: typescript-eslint + react-hooks + react-refresh
package.json

src/
├── main.tsx
├── App.tsx
├── index.css           # Tailwind entry (@import "tailwindcss")
├── features/
│   └── log-explorer/
│       ├── components/       # ImportPanel, ProgressBar, LogTable, SearchFilterBar, Pagination
│       ├── hooks/             # useLogImport, useLogSearch
│       ├── worker/log-import.worker.ts   # file read + parse + batch write
│       ├── db/
│       │   ├── database.ts    # open/upgrade, store + index setup
│       │   └── entries.ts     # typed cursor-based query helpers
│       ├── types.ts            # LogEntry, worker message types
│       └── parseLine.ts        # pure line-parsing logic
└── shared/               # empty placeholder — only populate if a 2nd feature needs it

tests/unit/
├── parseLine.test.ts
├── entries.test.ts        # uses fake-indexeddb
└── worker-messages.test.ts
```

Imports use the `@/` alias (`@/features/log-explorer/types`), never deep
relative paths.

**Structure Decision**: Single front-end project, feature-based
(`src/features/log-explorer/`), no backend/frontend split (no backend
exists). `src/shared/` stays empty until a second feature needs it.

## Complexity Tracking

*No violations — table not needed.*
