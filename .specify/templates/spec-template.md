---

description: "Minimal feature specification — goal and final decisions only. No acceptance criteria, no data contracts, no code-shaped detail: that lives in plan.md/tasks.md and changes freely without touching this file."
---

# Feature: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`
**Ticket**: [optional link/ID, or "—"]
**Created**: [DATE]
**Status**: Draft
**Input**: User description: "$ARGUMENTS"

## Goal

<!--
  1 short paragraph: what to build, for whom, and why it matters.
  This is the north star — it should stay true even as implementation changes.
  No tech stack, no acceptance-test detail, no data shapes. That's downstream.
-->

[Goal paragraph]

## Key Decisions

<!--
  Only decisions that had more than one reasonable option and materially
  affect scope, UX, or architecture — the choices you don't want re-litigated
  every time a new request comes in. State the choice and the one-line why.

  Include a build-order line here if there's a meaningful priority/MVP
  sequencing (e.g. "Build order: A before B before C, each shippable alone")
  — that's a decision, not an implementation detail, and it's what
  /speckit-tasks uses to phase the work.

  When a new request comes in later: check it against Goal + these
  decisions. Conflicts with a decision → that decision must be revisited
  and this file updated. No conflict → it's just an implementation detail,
  handle it in plan.md/tasks.md without touching this file.
-->

- **[Decision]**: [Chosen option] — [one-line why]
