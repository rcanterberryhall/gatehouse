# Requirements Sheet — PDF Export for Inspection Reports

*(Example of a filled sheet in **Feature** mode. Fictional but realistic.
Note how §8 pins down what must NOT change — that's the heart of a feature
sheet.)*

| Field | Value |
|---|---|
| **Title** | PDF export for inspection reports |
| **Type** | Feature |
| **Target system** | `inspecta` web app — report view module |
| **Author / owner** | R. Hall |
| **Date** | 2026-07-05 |
| **Revision** | A |
| **Status** | Issued for Design |

---

## 1. Purpose & Background

Inspection reports live in the `inspecta` web app, but clients and QA
packages need them as files. Today users print-to-PDF from the browser,
which breaks tables across pages, drops the header block, and produces
inconsistent output per browser. The app should generate the PDF itself so
every exported report is complete and identical regardless of who exports it.

## 2. Scope

### 2.1 In scope

- A per-report "Export PDF" action producing a complete, paginated PDF.
- Consistent header/footer: report ID, revision, date, page X of Y.

### 2.2 Out of scope

- Bulk/batch export of many reports.
- Editing or annotating the PDF after generation.

### 2.3 Non-goals

- Not a general template engine; one house layout is enough.

## 3. Definitions & References

| Term / Ref | Meaning / Location |
|---|---|
| Report | An inspection report record in `inspecta` (existing data model) |
| House layout | Margins/fonts per the QA document standard, `QA-DOC-004` |

## 4. Functional Requirements

| ID | Requirement | Rationale | Verify |
|---|---|---|---|
| REQ-F-001 | The system shall provide an "Export PDF" action on the report view that returns the full report as a single PDF. | The feature | D |
| REQ-F-002 | The exported PDF shall include all report sections, photos, and result tables, with no content truncated at page breaks. | The browser print path fails exactly here | T |
| REQ-F-003 | The PDF shall carry a header/footer with report ID, revision, export date, and page X of Y. | QA package traceability | I |
| REQ-F-004 | The system should complete an export of a typical report (≤ 30 photos) within 15 s. | Users will wait in-page | T |
| REQ-F-005 | The system may offer a photos-at-full-resolution toggle. | File size vs. fidelity; default can decide | — |

## 5. Non-Functional Requirements

| ID | Requirement | Rationale | Verify |
|---|---|---|---|
| REQ-N-001 | Export shall not block other users; a slow export must not stall the app for anyone else. | Single shared instance | T |

## 6. Constraints

| ID | Constraint |
|---|---|
| CON-001 | Implementation stays inside the existing `inspecta` stack and deploy (no new services in the compose file unless unavoidable). |
| CON-002 | Open-source PDF tooling only. |

## 7. Applicable Standards & Codes

| ID | Standard / Code | Applies to |
|---|---|---|
| STD-001 | gatehouse `STANDARDS.md` (day-0 engineering standards) | The new code — DOC/CODE/TST/LOG requirements |
| STD-002 | QA-DOC-004 (house document standard) | Page layout, fonts, header/footer content |

## 8. Interfaces & Compatibility

| ID | Interface / Compatibility item | Direction / Notes |
|---|---|---|
| IFC-001 | Existing report data model | Read-only; the feature shall not alter report schema or stored data |
| IFC-002 | Existing report view UI | Adds one action; existing view behavior and URLs shall not change |
| IFC-003 | Existing auth | Export honors the same per-report access as viewing |

## 9. Acceptance Criteria

1. Exporting the three reference reports (small / typical / photo-heavy)
   yields complete PDFs with correct pagination and headers (REQ-F-002,
   REQ-F-003).
2. A second user can browse normally while a photo-heavy export runs
   (REQ-N-001).

## 10. Open Questions & Deferred Decisions

| ID | Question | Options considered / notes |
|---|---|---|
| OQ-001 | Render server-side from HTML (headless browser) vs. a native PDF library? | Headless matches the web view exactly but is a heavy dependency (CON-001 tension); a PDF library is lighter but means maintaining a second layout. |
| OQ-002 | Generate synchronously in-request vs. as a background job with a download link? | Depends on REQ-F-004 feasibility for photo-heavy reports; REQ-N-001 pushes toward a job. |

---

*(Conventions and Note-to-the-design-assistant sections carry over from the
template unchanged — trimmed here for brevity in the example.)*
