# CLAUDE.md

Bridge file for Claude Code in this repository. Read this first when starting a session here.

## Project

**IndexDB — Local Log Explorer.** A learning/demo project, 100% front-end (no
backend), built to understand IndexedDB + Web Worker working together. Stack:
React + TypeScript + Vite, Node 24. Developed using Spec-Driven Development
via Spec Kit.

## Governance

Project principles live in [`.specify/memory/constitution.md`](.specify/memory/constitution.md).
Every plan, task, and line of code MUST comply with it. Key rules to keep in
mind at all times:

- Front-end only — never add a backend/server.
- Heavy/blocking work (parsing, indexing) MUST run in a Web Worker, never on
  the main thread.
- IndexedDB is the system of record for bulk data — no localStorage for bulk
  data.
- Strict TypeScript, no unexplained `any`.
- Simplicity/YAGNI — no speculative abstraction for a small scoped project.

## Where specs live

Each feature has its own folder under `specs/<NNN-feature-name>/`:

```
specs/001-log-import/
├── spec.md    # what & why (from /speckit-specify)
├── plan.md    # technical how (from /speckit-plan)
└── tasks.md   # work breakdown (from /speckit-tasks)
```

`spec.md` is the source of truth for that feature. If generated code doesn't
match intent, check whether the spec is wrong before patching code directly.

## Workflow

This project follows the Spec Kit flow. Run these in order, one fresh
session (`/clear`) per step:

1. `/speckit-constitution` — project rules (already done, edit only when
   principles change)
2. `/speckit-specify` — describe WHAT/WHY for a feature (no tech stack here)
3. `/speckit-plan` — technical HOW (tech stack, architecture)
4. `/speckit-tasks` — dependency-ordered task breakdown
5. Optional quality gate before implementing:
   `/speckit-clarify` (resolve ambiguity) → `/speckit-analyze` (cross-check
   spec/plan/tasks) → `/speckit-checklist` (validation checklist)
6. `/speckit-implement` — execute tasks, write real code

Do not start implementing a feature without an approved spec + plan + tasks
for it.

## Notes for the agent

- This is a learning project: prefer clarity over cleverness, and prefer
  explaining the IndexedDB/Worker mechanics over hiding them behind
  abstractions.
- Skills are installed under `.claude/skills/speckit-*` (hyphenated command
  names: `/speckit-specify`, not `/speckit.specify`).
