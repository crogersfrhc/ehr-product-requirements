# EHR Product Requirements

**Internal — FrontRunnerHC. Do not share outside the company.** Anything shared
with an EHR vendor is a separate, sanitized document and does not live here.

Product evidence packs and PRD inputs for EHR integrations, one folder per EHR.

## Layout

| Folder | Contents |
|---|---|
| `veradigm/` | Veradigm evidence pack, capability matrix, open-questions seed |

Each EHR folder holds, as the work progresses:

- `<ehr>-evidence-pack.md` — verified facts, each labelled by evidence type
  (`[CODE]`, `[DOCS]`, `[TESTED]`, `[TICKET]`, `[REPORTED]`, `[INFERRED]`, `[NOT FOUND]`)
- `<ehr>-capability-matrix.csv` — capability status table
- `<ehr>-open-questions-seed.md` — numbered open questions with owners
- PRDs, once written from the evidence pack

## What this repo is not

These documents are product inputs. They are **not** documentation of the
FrontRunner platform. See [CLAUDE.md](CLAUDE.md).
