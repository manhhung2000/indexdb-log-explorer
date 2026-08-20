---

description: "Task list for Log Explorer — Import & Search"
---

# Tasks: Log Explorer — Import & Search

**Input**: Design documents from `/specs/001-log-explorer-import/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [research.md](./research.md), [contracts/worker-messages.md](./contracts/worker-messages.md), [quickstart.md](./quickstart.md)

**Tests**: Included. plan.md's Project Structure explicitly names three test files
(`tests/unit/parseLine.test.ts`, `entries.test.ts`, `worker-messages.test.ts`) using
Vitest + `fake-indexeddb`, so those are treated as requested deliverables, not
optional extras. The real Worker + real IndexedDB path is validated manually via
[quickstart.md](./quickstart.md) instead (jsdom/Node can't run a real Worker).

**Organization**: Tasks are grouped by user story, derived from spec.md's Key
Decisions → **Build order**: import → search/filter/paginate → persistence across
reload. Each stage is independently shippable/demoable, matching spec.md's stated
MVP (import alone).

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependency on an incomplete task)
- **[Story]**: US1 = Import, US2 = Search/Filter/Paginate, US3 = Persistence
- File paths use the `@/` alias convention from plan.md, written as real repo-relative paths below

## Path Conventions

Single front-end project, feature-based (per plan.md): `src/features/log-explorer/`,
tests in `tests/unit/`. No backend, no `src/shared/` content yet (stays an empty
placeholder per plan.md until a second feature needs it).

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Scaffold the Vite + React + TS project from scratch — nothing exists yet (no `package.json`).

- [X] T001 Scaffold Vite + React + TS project at repo root (`package.json`, `tsconfig.json`, `tsconfig.app.json`, `index.html`, `src/main.tsx`, `src/App.tsx`, `src/index.css`) via the React+TS Vite template, Node 24 engines
- [ ] T002 Add project dependencies to `package.json`: `@tailwindcss/vite` + `tailwindcss` v4, `vitest` + `fake-indexeddb` (dev), `typescript-eslint` + `eslint-plugin-react-hooks` + `eslint-plugin-react-refresh` (dev)
- [ ] T003 Configure `vite.config.ts`: add `@tailwindcss/vite` plugin, `"@" -> "./src"` in `resolve.alias`, and a Vitest `test` config block (per research.md decisions)
- [ ] T004 [P] Configure `tsconfig.json`/`tsconfig.app.json`: `strict: true`, `paths: { "@/*": ["./src/*"] }`
- [ ] T005 [P] Configure `eslint.config.js` as a flat config (`typescript-eslint` + `react-hooks` + `react-refresh`, matching Vite's official `react-ts` template set)
- [ ] T006 [P] Replace `src/index.css` with the Tailwind v4 entry (`@import "tailwindcss";`) and confirm it's imported from `src/main.tsx`
- [ ] T007 Create `src/features/log-explorer/{components,hooks,worker,db}/` directory skeleton, an empty `src/shared/` placeholder, and clear the Vite starter boilerplate out of `src/App.tsx`

**Checkpoint**: `npm run dev` serves a blank app; `npm run lint` and `npm run build` both pass on the empty scaffold.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core types and DB schema that every user story depends on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T008 [P] Define shared types in `src/features/log-explorer/types.ts`: `LogEntry`, `ImportRun`, `MainToWorkerMessage`, `WorkerToMainMessage` (per [data-model.md](./data-model.md) and [contracts/worker-messages.md](./contracts/worker-messages.md))
- [ ] T009 Implement `src/features/log-explorer/db/database.ts` — `openDatabase()` opening `"log-explorer"` v1, creating the `entries` object store (`keyPath: "id"`, `autoIncrement: true`) with indexes `by_timestamp` and `by_level`, plus a `meta` object store (no keyPath, out-of-line key) on `upgradeneeded`; also export `getFileName(db)`/`setFileName(db, name)` helpers reading/writing the `"fileName"` key in `meta` (per data-model.md)

**Checkpoint**: Foundation ready — user story implementation can now begin.

---

## Phase 3: User Story 1 - Import a log file without blocking the UI (Priority: P1) 🎯 MVP

**Goal**: Selecting a large log file parses and writes it into IndexedDB from a Web
Worker, batch by batch, with progress reported back to the main thread — the page
stays responsive throughout.

**Independent Test**: Run `npm run dev`, open the app, select a large `sample.log`
(per quickstart.md). The page stays scrollable/clickable while a progress indicator
advances. When it reports done, DevTools → Application → IndexedDB →
`log-explorer` → `entries` count matches the file's line count.

### Implementation for User Story 1

- [ ] T010 [P] [US1] Implement `src/features/log-explorer/parseLine.ts` — pure function parsing `timestamp`/`level`/`message` out of a raw line, always keeping `raw`; never drops an unrecognized line (returns `timestamp: null`, `level: null` instead)
- [ ] T011 [US1] Unit test `tests/unit/parseLine.test.ts` — recognized `[LEVEL]`/timestamp lines, and unrecognized lines that still keep `raw` (depends on T010)
- [ ] T012 [US1] Implement write-path helpers in `src/features/log-explorer/db/entries.ts`: `addEntriesBatch(db, entries)` and `clearEntries(db)`, both driven off transaction `oncomplete` (depends on T009)
- [ ] T013 [US1] Unit test `tests/unit/entries.test.ts` (write path) — `addEntriesBatch`/`clearEntries` against `fake-indexeddb` (depends on T012)
- [ ] T014 [P] [US1] Unit test `tests/unit/worker-messages.test.ts` — asserts the `MainToWorkerMessage`/`WorkerToMainMessage` shapes from contracts/worker-messages.md (depends on T008)
- [ ] T015 [US1] Implement `src/features/log-explorer/worker/log-import.worker.ts` — on `'start'`: open DB, `clearEntries`, `setFileName(db, file.name)`, read `file.stream()` and decode/split into lines, parse via `parseLine`, batch per `batchSize` into `addEntriesBatch` transactions, `postMessage({type:'progress', batchCount, totalProcessed})` after each batch's `oncomplete`, then `'done'`; on error, call `clearEntries(db)` to remove any batches already written for this import BEFORE `postMessage({type:'error', message})` (as a plain string) — leaves the store empty rather than partial, per spec.md Clarification Q3 (depends on T010, T012, T008, T009)
- [ ] T016 [US1] Implement `src/features/log-explorer/hooks/useLogImport.ts` — spawns the worker, posts `'start'`, tracks `ImportRun` state (`idle`/`importing`/`done`/`error`, `linesProcessed`, `errorMessage`) off `progress`/`done`/`error` messages (depends on T015)
- [ ] T017 [P] [US1] Implement `src/features/log-explorer/components/ProgressBar.tsx` — renders processed-line count / status from `ImportRun`
- [ ] T018 [US1] Implement `src/features/log-explorer/components/ImportPanel.tsx` — file picker that calls `useLogImport`, renders `ProgressBar` plus status/error text; file input/import trigger MUST be `disabled` while `ImportRun.status === 'importing'`, re-enabled on `'done'`/`'error'` (per spec.md Clarification Q2 — prevents two Worker instances writing concurrently) (depends on T016, T017)
- [ ] T019 [US1] Wire `ImportPanel` into `src/App.tsx` (depends on T018, T007)

**Checkpoint**: User Story 1 is fully functional — import a file, watch non-blocking progress, confirm entries land durably in IndexedDB.

---

## Phase 4: User Story 2 - Search, filter, and paginate imported entries (Priority: P2)

**Goal**: Filter entries by level, free-text search `message`/`raw`, and page through
results, all via cursor + keyrange IndexedDB queries (never `getAll()` + JS filter).

**Independent Test**: With entries already in IndexedDB (from US1), apply a level
filter and see only matching entries with a correct pagination count; free-text
search finds a known substring from the sample file.

### Implementation for User Story 2

- [ ] T020 [US2] Implement read-path helpers in `src/features/log-explorer/db/entries.ts`: `queryEntries({ level?, searchText?, page, pageSize })` (cursor + keyrange via `by_level`/`by_timestamp`, linear scan only for the free-text substring check) and `countEntries(filters)` (depends on T012, same file)
- [ ] T021 [US2] Extend `tests/unit/entries.test.ts` with read-path cases: level filter, free-text substring match, pagination boundaries (depends on T020, T013, same file)
- [ ] T022 [US2] Implement `src/features/log-explorer/hooks/useLogSearch.ts` — holds filter/page state, calls `queryEntries`/`countEntries` (depends on T020)
- [ ] T023 [P] [US2] Implement `src/features/log-explorer/components/LogTable.tsx` — renders a page of `LogEntry` rows
- [ ] T024 [P] [US2] Implement `src/features/log-explorer/components/SearchFilterBar.tsx` — level dropdown + free-text input
- [ ] T025 [P] [US2] Implement `src/features/log-explorer/components/Pagination.tsx` — page controls driven by `countEntries` total
- [ ] T026 [US2] Wire `LogTable` + `SearchFilterBar` + `Pagination` into `src/App.tsx` via `useLogSearch`, shown once import is `done` (depends on T022, T023, T024, T025, T019)

**Checkpoint**: User Stories 1 AND 2 both work — import, then search/filter/paginate the result.

---

## Phase 5: User Story 3 - Persist across reload; new import replaces old data (Priority: P3)

**Goal**: Entries survive a hard refresh without re-importing, and starting a new
import clears out the previous data first (spec.md Key Decisions: "new import
replaces old data").

**Independent Test**: Import a file, hard-refresh the page — entries and count are
still there without re-importing (per quickstart.md scenario 2). Import a second
file — only its entries remain afterward (scenario 4).

### Implementation for User Story 3

- [ ] T027 [US3] In `src/App.tsx`, on mount check for existing entries (via `database.ts` + `countEntries`) and show the search/table view directly instead of `ImportPanel` when entries are already present, reading the file name via `getFileName(db)` to display in the top bar (depends on T009, T020, T026)
- [ ] T028 [US3] Verify `log-import.worker.ts`'s `clearEntries` call runs to completion before the first `addEntriesBatch` write of a new import, so a second import fully replaces the first (depends on T015, T012)

**Checkpoint**: All three user stories work independently — reload-persistence and replace-on-reimport both confirmed.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final validation across the whole feature

- [ ] T029 [P] Run `npm run lint` and fix any violations across `src/`
- [ ] T030 Run `npm run build && npm run preview`, then manually validate [quickstart.md](./quickstart.md) scenarios 1–4 against the static build output

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Setup — BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational only
- **User Story 2 (Phase 4)**: Depends on Foundational; `entries.ts`/`entries.test.ts` build on top of US1's write-path (T012/T013); `App.tsx` wiring builds on US1's (T019)
- **User Story 3 (Phase 5)**: Depends on US1 (worker's clear-before-write) and US2 (query helpers, `App.tsx` view) — it's a thin verification/wiring layer on top of both
- **Polish (Phase 6)**: Depends on all user stories being complete

### Within Each User Story

- Foundational types/DB schema before story-specific DB helpers
- DB helpers before hooks
- Hooks before components that consume them
- Components before `App.tsx` wiring
- Story complete and checkpointed before moving to the next priority

### Parallel Opportunities

- All Setup tasks marked `[P]` (T004, T005, T006) can run together once T001–T003 land
- T008 (types) can run in parallel with Setup's tail end, but must finish before T009
- Within US1: T010 (parseLine) and T014 (worker-message test) are parallel; T017 (ProgressBar) is parallel with the DB/worker/hook chain
- Within US2: T023, T024, T025 (LogTable, SearchFilterBar, Pagination) are parallel with each other
- T029 (lint) can run in parallel with the final manual validation pass (T030 is a different activity but touches build output, not source — sequence them if working solo)

---

## Parallel Example: User Story 1

```bash
# Once Foundational (T008, T009) is done, these can start together:
Task: "Implement parseLine.ts pure function"                 # T010
Task: "Unit test worker-messages.test.ts contract shapes"     # T014

# Once useLogImport (T016) exists, ProgressBar can already be in progress:
Task: "Implement ProgressBar.tsx component"                   # T017
```

## Parallel Example: User Story 2

```bash
# Once useLogSearch (T022) exists, all three components can be built together:
Task: "Implement LogTable.tsx"          # T023
Task: "Implement SearchFilterBar.tsx"   # T024
Task: "Implement Pagination.tsx"        # T025
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (types + DB schema)
3. Complete Phase 3: User Story 1 (import, non-blocking, durable)
4. **STOP and VALIDATE**: run quickstart.md scenario 1 + 2 manually
5. Demo: select a large log file, watch it import without freezing the page

### Incremental Delivery

1. Setup + Foundational → foundation ready
2. Add User Story 1 → validate independently → demo (MVP!)
3. Add User Story 2 → validate independently → demo (search/filter/paginate)
4. Add User Story 3 → validate independently → demo (reload persistence, replace-on-reimport)
5. Polish (lint, build/preview, full quickstart pass)

---

## Notes

- `[P]` tasks = different files, no dependency on an incomplete task
- `[Story]` label maps each task to US1/US2/US3 for traceability
- `db/entries.ts` and `tests/unit/entries.test.ts` are touched by both US1 (write path) and US2 (read path) — same file, so those task pairs are sequential, not parallel, even though they sit in different story phases
- Commit after each task or logical group
- Stop at any checkpoint to validate a story independently before moving on
- No IndexedDB wrapper library, no state-management library, no `cancel` message in v1 — matches research.md's explicit decisions and the constitution's Simplicity principle
