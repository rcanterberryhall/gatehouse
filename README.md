# gatehouse

The entry point every new project or feature passes through.

Gatehouse packages a development process that has worked well with an AI
coding assistant (Claude Code) as the primary code writer: an engineer fills
out a **requirements sheet** — the kind of shall/should/may document you'd
see at the start of concept or contract design — and starts the AI session
with it in context. The sheet seeds the design dialogue; the process carries
the work through design spec, implementation plans, execution, and
verification, with the engineer at every gate.

## What's here

| File | What it is |
|---|---|
| [`INTAKE-TEMPLATE.md`](INTAKE-TEMPLATE.md) | The requirements sheet. Copy this to start anything new. |
| [`ROADMAP.md`](ROADMAP.md) | The six-phase process from idea to verified, merged code. |
| [`STANDARDS.md`](STANDARDS.md) | Day-0 engineering standards every project inherits: README, docstrings + type hints, pinned format/lint gate, tests + CI, structured logging, self-documenting structure (per-package READMEs + a docstring-generated code map, enforced in CI), and branch hygiene (feature branch per feature, `--no-ff` merges, tagged milestones). |
| [`examples/`](examples/) | Filled-out sheets — one New Project, one Feature. |
| [`tools/`](tools/) | Reference implementation of the code-map generator (Python, stdlib-only) to vendor into new projects, plus a theoretical C++ mapping. |
| [`skill/`](skill/) | Optional Claude Code skill: auto-finds the requirements sheet at project start and close-out, and holds the Phase-5 verification gate. |

## Quickstart

1. **Copy the template.** Name it `YYYY-MM-DD-<topic>-requirements.md` and
   keep it wherever your team keeps design documents.
2. **Fill it out.** Requirements as *"The system shall / should / may ..."*
   with an ID and a verification method each. Constraints, standards,
   interfaces. Push genuine unknowns into §10 Open Questions — that section
   is what the AI design dialogue will chew on.
3. **Start the session.** Open your AI assistant in the target workspace and
   say:

   > Read `<path-to-your-sheet>` and brainstorm the design from it.

   The sheet's built-in *Note to the design assistant* tells the AI how to
   consume it: treat *shall* and constraints as fixed, probe §10, flag
   contradictions, and cite requirement IDs in the design spec it produces.
4. **Follow the roadmap.** [`ROADMAP.md`](ROADMAP.md) takes it from there —
   design spec → plans → execution → verification against the sheet.

## Why a requirements sheet?

Starting an AI session with a blank prompt means the first hour is the
assistant interviewing you for facts you already knew. Starting it with a
requirements sheet inverts that: the invariants (requirements, constraints,
standards, compatibility) are stated once with IDs, and the dialogue spends
its depth on the genuinely open design questions. The IDs then give you
traceability for free — every design decision, plan, and verification step
points back at a numbered requirement, the same way commissioning paperwork
traces back to a URS.

## License

[MIT](LICENSE) — copy the template, the process, and the tooling freely.
