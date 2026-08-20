<!--
Sync Impact Report
- Version change: (none) → 1.0.0 (initial ratification)
- Modified principles: n/a (new document)
- Added sections: Core Principles (I–VI), Technology Constraints, Development Workflow, Governance
- Removed sections: n/a
- Follow-up TODOs: none
-->
# IndexDB (Local Log Explorer) Constitution

## Core Principles

### I. Front-End Only
The project MUST remain 100% client-side. No backend service, server-rendered
API, or remote database MAY be introduced. All data ingestion, processing,
and persistence happen entirely in the browser. Rationale: the project's
purpose is to learn and demonstrate browser-native storage and concurrency
primitives (IndexedDB, Web Worker); adding a server would dilute that goal
and introduce unrelated concerns.

### II. Learning-First Scope
Every feature MUST directly serve the Log Explorer use case (import large
log files, parse, store, search/filter/paginate). Features that do not
illustrate IndexedDB or Web Worker usage patterns MUST be rejected or
deferred, regardless of how easy they are to add. Rationale: this is a
scoped learning project; breadth of features is explicitly not a goal.

### III. Worker-First for Heavy Computation
Any CPU-intensive or potentially blocking operation (file parsing,
tokenization, batch writes, aggregation) MUST run inside a Web Worker, never
on the main thread. The main thread is reserved for rendering and user
interaction. Rationale: keeping the main thread responsive under large data
volumes is the core technical lesson of this project.

### IV. IndexedDB as the System of Record
Bulk or structured application data (log entries, derived indexes) MUST be
persisted in IndexedDB. `localStorage`/`sessionStorage` MUST NOT be used for
bulk data and MAY only be used for small UI preferences (e.g. theme).
Rationale: demonstrating correct IndexedDB usage (object stores, indexes,
cursors, transactions) is a primary objective.

### V. Strict TypeScript
The project MUST use TypeScript in strict mode. The `any` type MUST NOT be
used unless no safer alternative exists, and each such use MUST carry an
inline comment explaining why. Rationale: strict typing catches
IndexedDB/worker message-passing mistakes (a common source of bugs in this
domain) at compile time.

### VI. Simplicity (YAGNI)
Prefer duplication over premature abstraction. Do not introduce state
management libraries, generic plugin systems, or configurability beyond
what the current spec requires. Three similar lines of code are better than
a speculative abstraction. Rationale: this is a small, scoped project;
unnecessary abstraction obscures the IndexedDB/Worker lessons it exists to
teach.

## Technology Constraints

- Runtime: Node.js 24+ for tooling/build only (not shipped to the browser).
- Framework: React + TypeScript, bundled with Vite.
- Storage: IndexedDB (native API or a thin typed wrapper), no server-backed
  storage.
- Concurrency: native Web Worker API (or Comlink-style thin wrapper if
  adopted later via an explicit spec decision); no server-side workers.
- No external backend, no network calls required for core functionality.

## Development Workflow

This project follows Spec-Driven Development via Spec Kit. Every feature
MUST progress through: `/speckit-specify` → `/speckit-plan` →
`/speckit-tasks` → (optional `/speckit-clarify`, `/speckit-analyze`,
`/speckit-checklist`) → `/speckit-implement`. Implementation MUST NOT begin
without an approved spec and task breakdown. If a build result does not
match intent, the spec (not just the code) MUST be reconsidered as the
possible source of the mismatch before further prompting.

## Governance

This constitution supersedes ad hoc practices and prior undocumented
decisions. Amendments are made via `/speckit-constitution` and MUST include
an updated Sync Impact Report. Versioning follows semantic versioning:
MAJOR for backward-incompatible principle removal/redefinition, MINOR for
new or materially expanded principles/sections, PATCH for clarifications
and wording fixes. All specs and plans MUST be checked for compliance with
these principles before `/speckit-implement` is run; unjustified complexity
MUST be flagged during `/speckit-analyze` or plan review.

**Version**: 1.0.0 | **Ratified**: 2026-08-20 | **Last Amended**: 2026-08-20
