---
name: gatehouse
description: Use when starting, designing, planning, or finishing (wrapping up, merging, closing out) work on a project or feature — especially one that has, or should have, a requirements sheet (*-requirements.md) in the team's design-docs location or was started through the gatehouse intake process.
---

# gatehouse

## Overview

The gatehouse repo (github.com/rcanterberryhall/gatehouse — your team will
have a clone or fork) is the single source of truth for the project-intake
process (requirements sheet → brainstorm → spec → plans → execution →
verification). This skill does not restate it. Its one job: make sure the
requirements sheet gets **found** at the two moments it is routinely missed.

## The rule: look for the sheet first

At BOTH of these moments, before anything else:

1. **Starting** design/planning work on an app or feature
2. **Finishing** it (user says wrap up / merge / close out / done)

check the team's design-docs location for `*requirements*.md` — e.g. a
sibling `<app>-plans/` directory, the repo's `docs/`, wherever this team
keeps design documents.

- **Sheet exists** → read it fully and follow its "Note to the design
  assistant". Never re-ask what it answers; never draft a new sheet when
  one exists.
- **No sheet, new project** → offer the gatehouse `INTAKE-TEMPLATE.md`
  before brainstorming.
- Process phases and gates: gatehouse `ROADMAP.md`. Day-0 standards:
  gatehouse `STANDARDS.md` (the sheet's STD-001 row pulls these in).

## Close-out gate (Phase 5)

Sheet-traced work is not done at green tests. Before merge: walk the
sheet's §4/§5 requirement tables row by row, verify each *shall* by its
declared method (T/D/I/A), report per-ID results, and document any *should*
deviations. **Demonstration and Inspection rows are structurally invisible
to a passing unit-test suite.**

| Excuse | Reality |
|---|---|
| "Tests are green — wrap it up" | The suite cannot verify D/I rows. The walk takes minutes. |
| "The user didn't mention a sheet" | Users never re-mention it at close-out. Finding it is your job. |
| "I'll draft a fresh requirements sheet" | Check the design-docs location first — a duplicate discards the engineer's recorded answers. |
