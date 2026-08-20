# Contract: Main Thread ↔ Import Worker `postMessage` Protocol

> `/speckit-plan` Phase 1 output — regenerate via the command, don't hand-edit.

Only "interface" this project exposes (no HTTP API, no CLI). Types live in
`src/features/log-explorer/types.ts`, shared by both sides.

```ts
// Main thread → Worker (sent once, when a file is confirmed)
type MainToWorkerMessage = {
  type: 'start';
  file: File;
  batchSize: number; // e.g. 500 lines per IndexedDB transaction
};

// Worker → Main thread
type WorkerToMainMessage =
  | { type: 'progress'; batchCount: number; totalProcessed: number }
  | { type: 'done'; totalProcessed: number }
  | { type: 'error'; message: string };
```

- `progress` fires after each batch's transaction `oncomplete` — progress
  shown to the user always reflects durable data, not just parsed data.
- Sequencing: one `start` → zero-or-more `progress` → exactly one `done` or
  `error`. No `cancel` in v1 (not required by spec).
- Errors are reported as plain strings (`error.message`), not raw
  `Error`/`DOMException` objects — those don't reliably structured-clone.
