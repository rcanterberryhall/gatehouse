# Changelog

All notable changes to gatehouse. Newest first; grouped by version/tag once
tags exist (per `STANDARDS.md` DOC-009 / VCS-003). Until then, by date.

## [Unreleased]

## [0.1] - 2026-07-05

### Added
- **Branch-hygiene standards family** (`STANDARDS.md` VCS-001…007): one feature
  branch per feature, no direct commits to the default branch, `--no-ff`
  merges, lightweight `v0.x` milestone tags, verify the checked-out branch
  before merging, isolate parallel work, and make the merge-vs-PR integration
  route an explicit finish-time decision. VCS-007 adds: plan execution begins
  from a clean working tree.
- **Plan-execution preflight** (`ROADMAP.md` Phase 4): start from a clean tree
  and check each plan for staleness against the current design and code before
  writing.
- **Changelog standard** (`STANDARDS.md` DOC-009): every ongoing project keeps
  a `CHANGELOG.md`. This file is gatehouse eating its own dog food.

## 2026-07-05 — initial release

### Added
- `INTAKE-TEMPLATE.md` — URS-style requirements sheet (shall/should/may, IDs,
  T/D/I/A verification methods) to seed the design dialogue.
- `ROADMAP.md` — the six-phase process (Intake → Brainstorm → Design spec →
  Plans → Execution → Verification & close-out) with gates.
- `STANDARDS.md` — day-0 standards families DOC / CODE / TST / LOG.
- `tools/gen_code_map.py` — stdlib-only code-map generator (DOC-007/008
  reference implementation) with adoption notes.
- `examples/` — one filled New-Project sheet and one Feature sheet.
- `skill/` — optional Claude Code skill companion that finds the requirements
  sheet at project start and close-out.
- CI: pinned ruff gate and a generator fixture test. MIT license.
