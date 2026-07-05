# Requirements Sheet — CalTrack (Test-Instrument Calibration Tracker)

*(Example of a filled sheet in **New Project** mode. Fictional but realistic.)*

| Field | Value |
|---|---|
| **Title** | CalTrack — test-instrument calibration tracker |
| **Type** | New Project |
| **Target system** | N/A |
| **Author / owner** | R. Hall |
| **Date** | 2026-07-05 |
| **Revision** | A |
| **Status** | Issued for Design |

---

## 1. Purpose & Background

The department owns ~120 test instruments (multimeters, calibrators, torque
tools, insulation testers) whose calibration status is tracked in a shared
spreadsheet. The spreadsheet goes stale: due dates are missed until an
auditor or a job's QA package catches it, and finding "which calibrated meter
can I take to site this week" means reading the whole sheet. We need a small
internal web app that is the single source of truth for instrument
calibration status and surfaces what is due before it lapses.

## 2. Scope

### 2.1 In scope

- Instrument register: identity, location/custodian, cal interval, cal due
  date, certificate reference.
- Status views: overdue, due within N days, in-cal.
- Recording a completed calibration (new due date, cert reference).
- CSV import of the existing spreadsheet for initial load.

### 2.2 Out of scope

- Performing or scheduling the calibrations themselves (external lab).
- Purchase/asset-value tracking — that stays in the asset register.

### 2.3 Non-goals

- Not a full asset-management system; instruments only.
- No mobile app; the web UI on a phone browser is sufficient.

## 3. Definitions & References

| Term / Ref | Meaning / Location |
|---|---|
| Cal interval | Months between calibrations, set per instrument |
| Cert | Calibration certificate PDF from the external lab |
| Existing sheet | `\\fileserver\qa\instrument_cal_register.xlsx` |

## 4. Functional Requirements

| ID | Requirement | Rationale | Verify |
|---|---|---|---|
| REQ-F-001 | The system shall store, for each instrument: asset ID, description, manufacturer/model, serial, custodian, location, cal interval, last cal date, due date, and certificate reference. | Single source of truth replacing the spreadsheet | T |
| REQ-F-002 | The system shall compute due date from last cal date + interval. | Removes the manual-arithmetic error source | T |
| REQ-F-003 | The system shall list instruments filtered by status: overdue, due within 30 days, in-cal. | The core daily question | D |
| REQ-F-004 | The system shall record a completed calibration by entering the cal date and certificate reference, recomputing the due date. | Keeps the register live | T |
| REQ-F-005 | The system shall import the existing register from CSV for initial load. | 120 instruments won't be re-typed | T |
| REQ-F-006 | The system should attach or link the certificate PDF per calibration event. | Auditors ask for the cert, not the date | D |
| REQ-F-007 | The system may email the custodian when an instrument comes within 30 days of due. | Nice push channel; the due list is the must-have | — |

## 5. Non-Functional Requirements

| ID | Requirement | Rationale | Verify |
|---|---|---|---|
| REQ-N-001 | The system shall keep a change history per instrument (who changed what, when). | Audit trail for QA | I |
| REQ-N-002 | The system shall render any status view in under 2 s at 500 instruments. | Headroom over the current 120 | T |
| REQ-N-003 | The system should be usable by a first-time user without training. | Casual users, quarterly | D |

## 6. Constraints

| ID | Constraint |
|---|---|
| CON-001 | The app will run on the existing internal Ubuntu server; no cloud services. |
| CON-002 | Deployment shall be a single Docker Compose stack. |
| CON-003 | No per-seat licensing costs; open-source dependencies only. |

## 7. Applicable Standards & Codes

| ID | Standard / Code | Applies to |
|---|---|---|
| STD-001 | ISO/IEC 17025 (awareness) | Cal record fields must carry what an accredited-lab cert provides |
| STD-002 | Internal Python style guide | All backend code |

## 8. Interfaces & Compatibility

| ID | Interface / Compatibility item | Direction / Notes |
|---|---|---|
| IFC-001 | Existing register CSV export | In, once, initial load (REQ-F-005) |
| IFC-002 | Company SSO reverse proxy | In; app reads authenticated user from proxy header |
| IFC-003 | Desktop and phone browsers (current Chrome/Edge/Safari) | UI must work on both |

## 9. Acceptance Criteria

1. The full existing register is imported and every instrument's computed
   due date matches the spreadsheet (REQ-F-002, REQ-F-005).
2. A user can find all overdue instruments in one click (REQ-F-003).
3. Recording a calibration updates status immediately and appears in the
   change history (REQ-F-004, REQ-N-001).

## 10. Open Questions & Deferred Decisions

| ID | Question | Options considered / notes |
|---|---|---|
| OQ-001 | Store certificate PDFs in the app, or link to the existing file server? | Storing is self-contained but duplicates QA's filing; linking is fragile if paths move. |
| OQ-002 | Single flat register vs. supporting instrument "kits" (e.g., a cal kit checked out as a unit)? | Kits exist in practice but add data-model weight; maybe v2. |
| OQ-003 | Who may edit vs. view? Is a two-role model (QA edits, everyone views) enough? | Simplest that could work; needs QA lead's confirmation. |

---

*(Conventions and Note-to-the-design-assistant sections carry over from the
template unchanged — trimmed here for brevity in the example.)*
