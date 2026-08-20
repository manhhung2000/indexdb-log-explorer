# Feature: Log Explorer — Import & Search

**Feature Branch**: `001-log-explorer-import`
**Ticket**: —
**Created**: 2026-08-20
**Status**: Draft

## Goal

Front-end-only tool to import a large log file into the browser, then
search/filter/paginate it, without freezing the UI and without losing data
on reload. Purpose: learn IndexedDB (storage) + Web Worker (non-blocking
processing) via a real use case.

## Clarifications

### Session 2026-08-20

- Q: What input format should be recognized so `parseLine` can extract `timestamp`/`level`? → A: An ISO-8601 timestamp at the start of the line, followed by a `[LEVEL]` token, with the remainder treated as `message` (matches the sample generator in quickstart.md)
- Q: What should happen if the user picks a new log file while another import is still in progress? → A: Disable the file picker/import trigger until the current import reaches `done` or `error`
- Q: If an import fails partway through, should the batches already written stay, or be cleared automatically? → B: Automatically clear the written data on error, leaving the store empty — avoids an indistinguishable partial-data state after a reload

## Key Decisions

- **Recognized log line format**: an ISO-8601 timestamp at the start of the
  line, followed by a `[LEVEL]` token, with the remainder treated as
  `message` (e.g. `2026-08-20T10:00:05.000Z [INFO] sample log line`, per
  quickstart.md's sample generator). A line not matching this shape still
  keeps its full `raw` text with `timestamp: null` and `level: null` — never
  dropped (per data-model.md).
- **Build order**: import (non-blocking) → search/filter/paginate →
  persistence across reload. Each step independently shippable/demoable —
  this is what `/speckit-tasks` uses to phase the work (MVP = import step
  alone).
- **One import at a time**: the file picker/import trigger MUST be disabled
  while an import is in progress (`status: 'importing'`), re-enabled only on
  `'done'` or `'error'`. Prevents two Worker instances writing to the same
  IndexedDB store concurrently — v1's worker protocol has no `'cancel'`
  message (per contracts/worker-messages.md).
- **Error mid-import clears partial data**: if the worker reports `'error'`,
  it MUST clear any batches already written for that import before posting
  the error, leaving the `entries` store empty rather than a partial
  dataset. Keeps the invariant "store is always either empty or one
  complete import" — important for reload persistence (a reload after a
  failed import must not silently show truncated data as if it were
  complete).
- **New import replaces old data**, not accumulated/merged across multiple
  files. Why: keeps v1 scope small (Constitution: Simplicity). Revisit as a
  separate feature if multi-file browsing turns out to matter.
- **Heavy work (parsing, writing) runs in a Web Worker**, never blocks the
  main thread — this is non-negotiable, it's the point of the project
  (Constitution: Worker-First).
- **Storage is IndexedDB**, not localStorage — the other point of the
  project (Constitution: IndexedDB as system of record).
