# Nirgam — Discharge Desk

An agentic AI system that turns hospital discharge from a five-hour scramble into a continuous background process.

Built for **UXplorer 2026** (yuj Designs). Agent workflow runs in yoxa.ai; this repository holds the frontend and the synthetic datasets.

---

## The problem

In Indian private hospitals a doctor clears a patient at 9am and the patient leaves at 4pm. None of that delay is medical. The discharge summary is typed from scratch, the bill waits on departmental clearances, the insurance packet is assembled by hand, and insurer queries bounce the file back.

- IRDAI's 2024 master circular requires insurers to authorise final discharge within **3 hours**
- NABH sets **180 minutes** as the discharge standard
- Published studies find insured patients average **5h 09m**

## The idea

Discharge is not an event at the end of a stay. It is a **state maintained from admission**: the summary drafts itself from daily notes, the bill reconciles live, the insurer checklist is audited continuously. On discharge day the only thing left is a consultant's signature.

## Agents

| Agent | Responsibility |
|---|---|
| Supervisor | State machine, routing, audit log, human escalation |
| Admission & TPA | Policy validation, stay thresholds, clinical justification extraction |
| Clinical Summary | EMR sweep, section mapping, allergy conflict scanning, query replies |
| Payment Handler | Ledger reconciliation, duplicate charge audit, exception routing |
| TPA Communications | Pre-auth, FHIR payload, NHCX dispatch, query classification |

Five human gates: submit pre-auth, sign extension, **sign discharge summary**, approve query reply, medical director reconciliation.

Governing rule: **stop or escalate, never guess.** Clinical content is restatement-only — every summary line carries a `[Source: Lxxx]` citation back to the note it came from. Agents are read-only on hospital systems.

## Running it

```
python3 -m http.server 5173
```

Then open `http://localhost:5173`. No build step, no dependencies. Or just open `index.html` directly.

## Files

| File | What it stands in for |
|---|---|
| `index.html` | The portal — ward dashboard, patients, review and sign, insurer queries, mock HIS |
| `mock_admissions.csv` | Hospital information system (HIS) |
| `mock_emr_logs.csv` | EMR daily notes |
| `mock_billing_ledger.csv` | Billing module |
| `insurer_checklist.csv` | Insurer document requirements (C1–C8) |
| `mock_insurer_queries.csv` | Inbound NHCX queries |

All joined on `patient_id`.

## Test patients

| Patient | Proves |
|---|---|
| PT001 Ramesh Kumar — appendectomy | Clean run. Checklist 7/8: final bill amount correctly flagged missing, because it is never in clinical notes |
| PT002 Sunita Devi — dengue | No discharge meds or follow-up recorded, so the summary marks them `[PENDING CONSULTANT INPUT]` rather than inventing them |
| PT003 Arjun Mehta — pneumonia | Penicillin allergy on record, penicillin derivative prescribed. Conflict raised, sign-off locked |
| PT004 Test Entry | Malformed policy number and non-clinical notes. Rejected at intake |

## What is simulated

- **NHCX transmission and query receipt** — no public sandbox exists. Payloads are built and logged locally, marked SIMULATED in the interface.
- **Patient data** — synthetic by design. A real deployment runs inside hospital infrastructure against live EMR/HIS feeds.
- **Role switching** — stands in for hospital SSO.
- **The HIS tab** — deliberately styled as a separate legacy product. Nirgam reads from hospital systems and never writes into them.

## Note on secrets

No API keys are present in this repository and none should ever be added. Keys belong in the backend proxy's environment variables only.

## Requirement

Nirgam needs clinical notes entered digitally. That holds in NABH-accredited corporate hospitals, the target segment. Where notes are still on paper, a discharge summary cannot be built continuously — a documentation problem, not one software can solve.
