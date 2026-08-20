# Quickstart: Validate Log Explorer — Import & Search

> `/speckit-plan` Phase 1 output — regenerate via the command, don't hand-edit.

Manual browser validation — the Worker + real IndexedDB path can't be fully
covered by unit tests (see [research.md](./research.md)).

## Setup

```bash
npm install
npm run dev
```

A sample log file, if none handy:

```bash
# PowerShell
1..300000 | ForEach-Object {
  "2026-08-20T10:00:$($_ % 60).000Z [INFO] sample log line $_"
} | Set-Content sample.log
```

## Scenarios

1. **Import doesn't block the UI** — select `sample.log`; page stays
   responsive (scroll/click work) while progress advances incrementally.
2. **Persists across reload** — after `done`, check DevTools → Application
   → IndexedDB → `log-explorer` → `entries` count matches the file; hard
   refresh; entries are still there without re-importing.
3. **Search/filter/paginate** — level filter shows only matching entries
   with correct pagination count; free-text search finds a known substring.
4. **New import replaces old** — import a second file; only its entries
   remain afterward.

## Build check

```bash
npm run build
npm run preview
```

Scenarios 1–4 still pass against the static build output.
