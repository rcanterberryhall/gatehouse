# Requirements Sheet — [Project or Feature Name]

<!--
HOW TO USE THIS TEMPLATE
1. Copy this file. Name it YYYY-MM-DD-<topic>-requirements.md and keep it
   wherever your team keeps design documents.
2. Fill in every section. Write "None" rather than deleting a section — an
   explicit "None" is information; a missing section is a gap.
3. Use the requirement keywords with discipline (see Conventions at the end):
   shall = binding, should = strongly desired, may = optional.
   If you find yourself unsure whether something is a shall or a should, it
   probably belongs in §10 Open Questions instead.
4. Every "shall" gets a verification method (T/D/I/A). If you can't say how
   you'd verify it, it isn't a requirement yet — rewrite it or move it to §10.
5. Guidance appears in HTML comments like this one. Delete them as you fill
   in each section, or leave them — they render invisibly either way.
6. When done, start your AI design session with this sheet in context and ask
   it to brainstorm the design from it. See ROADMAP.md for the full process.
-->

| Field | Value |
|---|---|
| **Title** | |
| **Type** | New Project / Feature *(delete one)* |
| **Target system** | *(Feature only — the app/repo/subsystem this changes; write N/A for new projects)* |
| **Author / owner** | |
| **Date** | |
| **Revision** | A |
| **Status** | Draft |

<!-- Status lifecycle: Draft → Issued for Design → Implemented (rev letter bumps on change) -->

---

## 1. Purpose & Background

<!-- The problem in your own words: what hurts today, who feels it, what
happens if nothing is built. State the problem, not the solution — "engineers
re-type calibration due dates into a spreadsheet that silently goes stale" is
a purpose; "build a Flask app" is not. 3–10 sentences. -->

## 2. Scope

### 2.1 In scope

<!-- Bullet list of what this project/feature covers. -->

### 2.2 Out of scope

<!-- What adjacent work this explicitly does NOT cover (someone else's
problem, or a later phase). -->

### 2.3 Non-goals

<!-- Things a reasonable person might assume are goals but are deliberately
not — with a word on why. Non-goals prevent the design from drifting toward
features nobody asked for. -->

## 3. Definitions & References

<!-- Domain terms, abbreviations, and pointers to source material the
designer will need: existing docs, prior art, the system being replaced,
relevant tickets. -->

| Term / Ref | Meaning / Location |
|---|---|
| | |

## 4. Functional Requirements

<!-- What the system does, one requirement per row, phrased as
"The system shall/should/may ...". Keep each row atomic — if it contains
"and", it is probably two requirements. Rationale says WHY it's required
(one line). Verify: T=Test, D=Demonstration, I=Inspection, A=Analysis. -->

| ID | Requirement | Rationale | Verify |
|---|---|---|---|
| REQ-F-001 | The system shall ... | | T |
| REQ-F-002 | The system should ... | | D |
| REQ-F-003 | The system may ... | | — |

## 5. Non-Functional Requirements

<!-- How well it does it: performance, capacity, security, reliability,
usability, maintainability. Make these measurable where you can —
"shall return search results in under 2 s for a 10k-record dataset" beats
"shall be fast". -->

| ID | Requirement | Rationale | Verify |
|---|---|---|---|
| REQ-N-001 | The system shall ... | | T |

## 6. Constraints

<!-- Fixed facts the design must live within — not preferences. Mandated
technology stack, deployment environment, hardware limits, licensing,
timeline, budget, team skills. Use "will" for statements of fact about the
environment ("The host will remain air-gapped"). -->

| ID | Constraint |
|---|---|
| CON-001 | |

## 7. Applicable Standards & Codes

<!-- Anything the work must comply with or align to: regulatory standards,
industry codes, internal coding standards, style guides, security policies.
Cite the clause/section where it matters. STD-001 is pre-filled: the
gatehouse day-0 standards (documentation, type hints, tests/CI, logging)
apply to every project by default — remove the row only with a documented
reason. -->

| ID | Standard / Code | Applies to |
|---|---|---|
| STD-001 | gatehouse `STANDARDS.md` (day-0 engineering standards) | Whole project — DOC/CODE/TST/LOG requirements |
| STD-002 | | |

## 8. Interfaces & Compatibility

<!-- Every system this touches: upstream data sources, downstream consumers,
APIs it must call or expose, file formats it must read/write, auth systems,
the deployment target, browsers/clients that must be supported. For Feature
mode: which existing behaviors must NOT change. -->

| ID | Interface / Compatibility item | Direction / Notes |
|---|---|---|
| IFC-001 | | |

## 9. Acceptance Criteria

<!-- The demonstrable end state: "this is done when ...". A short list an
engineer could walk through at handover, SAT-style. May cite requirement IDs.
This is the contract for Phase 5 (Verification) in ROADMAP.md. -->

1.

## 10. Open Questions & Deferred Decisions

<!-- The most important section for the AI design session. List what you
genuinely don't know yet or deliberately want the design dialogue to work
out: architecture choices, trade-offs you can see both sides of, unknowns
that need investigation. Include options you've already considered. A rich
§10 makes the brainstorming session sharp; an empty one makes it generic. -->

| ID | Question | Options considered / notes |
|---|---|---|
| OQ-001 | | |

---

## Conventions

**Requirement keywords** (per RFC 2119 / IEEE 29148 usage):

- **shall** — binding requirement. Must be implemented and verified before
  acceptance.
- **should** — strongly desired. Deviation is allowed but must be documented
  with justification in the design spec.
- **may** — optional. Implement if cheap or natural; omission needs no
  justification.
- **will** — statement of fact about the environment or an external
  commitment; not a requirement on the system and not verified.

**Verification methods** (declare one per shall/should requirement):

- **T — Test:** exercised by an automated or scripted manual test case.
- **D — Demonstration:** observed working in normal operation, no
  instrumentation.
- **I — Inspection:** confirmed by examining code, configuration, or documents.
- **A — Analysis:** confirmed by calculation, modeling, or reasoned review.

## Note to the design assistant

If you are an AI assistant reading this sheet at the start of a design
session:

- Treat **shall** clauses and §6 Constraints as fixed. Do not propose designs
  that violate them; if two of them conflict, say so explicitly and ask.
- Do not re-ask what this sheet already answers. Focus the design dialogue on
  **§10 Open Questions**, on gaps you notice, and on genuine design forks.
- Where the sheet is silent, ask — don't assume.
- In the design spec you produce, cite requirement IDs (REQ-F-001, CON-002,
  ...) so every design decision traces back to a requirement, and flag any
  **should** you choose not to satisfy, with justification.
