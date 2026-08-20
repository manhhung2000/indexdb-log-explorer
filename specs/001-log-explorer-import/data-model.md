# Phase 1 Data Model: Log Explorer — Import & Search

> `/speckit-plan` Phase 1 output — regenerate via the command, don't hand-edit.

Spec doesn't fix an exact log line format, so this defines the smallest
shape needed for import → search/filter/paginate. Parse what's recognizable,
always keep the original line — never drop a line for not matching a format.

## LogEntry (IndexedDB object store `entries`)

| Field | Type | Notes |
|---|---|---|
| `id` | `number` | `autoIncrement: true`, also the import-order sort key |
| `raw` | `string` | Original line, always present |
| `timestamp` | `number \| null` | Parsed epoch-ms, `null` if unrecognized |
| `level` | `'ERROR'\|'WARN'\|'INFO'\|'DEBUG'\|null` | Parsed from `[LEVEL]` token |
| `message` | `string` | Remainder after timestamp/level stripped, or full `raw` |

```text
Database: "log-explorer" (version 1)
Object store "entries" — keyPath: "id", autoIncrement: true
  Index "by_timestamp" → keyPath: "timestamp"
  Index "by_level"     → keyPath: "level"
Object store "meta" — no keyPath (out-of-line key)
  Key "fileName" → value: string
```

Free-text search (`message`/`raw`) is a cursor scan, not index-backed —
IndexedDB has no substring index. Fine at this project's scale.

Entries are immutable: written in batches during import, read via cursor
during search, and the whole store is cleared at the start of the next
import (new import replaces old data).

## Meta (IndexedDB object store `meta`)

Stores the name of the currently-imported file, so the UI can show it again
after a reload without re-reading the original `File` object (browsers don't
persist `File` handles across reloads). Written once the worker starts a new
import (same transaction batch as the first `clearEntries`), read on app
mount alongside the `entries` count check.

| Key | Value type | Notes |
|---|---|---|
| `"fileName"` | `string` | Name of the last successfully-started import's source file |

## ImportRun (in-memory only, not persisted)

| Field | Type | Notes |
|---|---|---|
| `status` | `'idle'\|'importing'\|'done'\|'error'` | Drives ImportPanel UI |
| `linesProcessed` | `number` | Running total from `progress` messages |
| `errorMessage` | `string \| null` | From a worker `error` message |
