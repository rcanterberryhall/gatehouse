# skill

A Claude Code skill companion to the gatehouse process. **Optional** — the
process works with plain markdown in any assistant; this adds automatic
triggering for Claude Code users.

## What it does (and deliberately doesn't)

Documents hold knowledge; skills hold triggers. The repo's ROADMAP.md,
STANDARDS.md, and the sheet's built-in "Note to the design assistant"
already tell an assistant everything it needs — *once the sheet is in
context*. Baseline testing (pressure scenarios with and without the skill)
showed exactly one recurring failure: **the assistant never goes looking
for an existing requirements sheet** — at project start it drafts a
duplicate and re-asks answered questions; at close-out it merges on green
tests without ever walking the requirement tables.

So the skill enforces exactly that: at start and at close-out, find the
sheet; if found, follow it — including the Phase-5 shall-by-shall
verification walk before any merge. Everything else stays canonical in
this repo, so the skill can't drift from the docs.

## Install

Copy into your personal Claude Code skills directory:

```bash
cp -r skill/gatehouse ~/.claude/skills/
```

It loads at your next session start.

## Customize

The skill refers to "the team's design-docs location" generically. If your
team has a fixed convention (e.g. a sibling `<app>-plans/` directory),
edit your installed copy to name it explicitly — a concrete path makes the
lookup more reliable than a description.
