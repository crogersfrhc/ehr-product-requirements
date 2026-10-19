# Handoff — Veradigm developer-portal research (Cowork, browser)

**Internal — FrontRunnerHC.** Prepared 2026-10-19 for a Claude Cowork session with browser access. Paste everything below the line into the new session.

---

## Who you are working for and why

You are helping Craig Rogers (Product Manager, FrontRunnerHC) fill vendor-fact gaps in an internal **Veradigm evidence pack**. The pack feeds a future PRD for a **Veradigm Sidecar**: an app embedded in the Veradigm EHR/PM that shows patient insurance, eligibility and demographics and writes staff-approved demographic and insurance changes back to Veradigm.

Public Veradigm pages have already been researched. What is missing sits **behind the Veradigm developer-portal login**, which Craig has an account for. Your job is to read those pages in the browser and record verified facts. You write **no requirements** and make **no decisions**.

## Ground rules (non-negotiable)

1. **Craig logs in himself.** Open https://developer.veradigm.com/ and ask Craig to sign in. Never ask for, type, store, or repeat his password, MFA codes, API keys, client IDs or secrets. If a page displays a credential or key, do not copy it; note only that it exists.
2. **Read only.** Do not create or register apps, request production access, start a tier upgrade or purchase, accept agreements or terms, submit forms, open support tickets, or change account settings. If a step would do any of these, stop and ask Craig.
3. **No sandbox calls.** Do not call any API, including sandbox, unless Craig explicitly tells you to in this session. Never send patient data anywhere.
4. **No PHI.** Sample data on portal pages may look like patient data; never reproduce it.
5. **Summarize, don't copy.** Portal content may be covered by developer terms or an NDA. Write short summaries in your own words with citations. No long quotes (one short phrase at most when wording matters).
6. **Fail honest.** If something is not documented, say "not documented in portal" and list where you looked. Never fill a gap with a guess; label any conclusion `[INFERRED]` with its basis.

## What is already known (verify, correct, or extend)

Public pages (opened 2026-10-19) said:
- Program "Veradigm Connect": tiers **Open** (free, FHIR only) and **Integrator** (FHIR + proprietary **Unity** APIs; sub-tiers Bronze → Platinum Plus); no public prices; Integrator apps must be tested and certified by Veradigm before client deployment.
- **FHIR API is read-only.** Unity is bidirectional and required for PM demographic, appointment and financial data.
- **SMART on FHIR EHR launch** documented for **Veradigm EHR**, Integrator tier and above; SSO for FHIR R4 needs **Veradigm EHR 24.5+**; for older versions Veradigm recommends Unity for app launch.
- **No launch mechanism documented for Practice Management.**
- TouchWorks is listed as an **Altera** product.
- The Unity API reference, Unity sandbox and approval process were **not public**.

FrontRunner's legacy (V1) code uses Unity like this (use it to know what to look for, not as fact about current Unity):
- Endpoints: `UnityService.svc/json/GetToken` then `UnityService.svc/json/MagicJson`.
- Login: app-level user + password to get a token; username can be a composite of **Ubiquity ID**, **PM Client ID**, **Tenant ID** and the app user; then `GetUserAuthentication` with a per-client service user.
- Actions used: GetUserAuthentication, GetAppointmentById, GetPatientDemographics, GetPatientPolicy, SavePatientPolicy, SavePatient, GetPractitioners, GetLocations, SaveAccountContact, GetRequiredFields, SavePatientNote, GetCarriers.
- Coverage priority values P / S / X; V1 works around a "Veradigm bug" by demoting a policy to X before re-assigning P/S; V1 notes that SaveAccountContact blanks fields that are not supplied.

## Questions to answer (map each finding to its ID)

| ID | Question |
|---|---|
| OQ-5 | Which Connect tiers exist, what each costs, which tier unlocks **Unity** read/write and **SMART launch**, and what **certification** involves (steps, duration, fees, re-certification). What contract/partner agreement is required. |
| OQ-6 | What Craig's current account includes: tier, Unity documentation access, **sandbox** (FHIR, Unity, SMART launch), and how to request more. |
| OQ-7 | Whether and how a third-party app can **launch inside Veradigm Practice Management** (embedded window, button, launcher, URL with context, SSO). |
| — | **Launch inside Veradigm EHR**: supported versions, launch contexts passed (patient, encounter, user, practice/tenant), where the app appears (tab, panel, window), and SSO requirements. |
| OQ-8 | Can a **Unity write be attributed to the launching user** (user-level token or "user" parameter) or only to a service account? How does auth work in the current Unity docs (GetToken, GetUserAuthentication, Ubiquity, token lifetime and refresh)? |
| — | **Unity action reference** for: patient demographics read/write; insurance/policy read, update, **add**, and **coverage priority/order**; subscriber/guarantor contacts; appointments; notes; required-field lookup. Note any **deprecated** actions and any newer REST API. |
| OQ-9 | **Idempotency / duplicate protection** on policy creates and other writes. |
| OQ-10 | **Rate limits**, throttling, and **locked-record** / lockout behavior; error codes. |
| OQ-11 | Whether the **two-pass priority workaround** and the **SaveAccountContact blanking** behavior are documented, fixed, or still expected. |
| OQ-12 | Confirm the auth model for writes (Unity token vs OAuth) and whether OAuth applies only to FHIR. |
| OQ-18 | Which **Veradigm EHR and PM versions** each API and launch method supports. |
| — | **Environments and hosting**: sandbox vs production endpoints; how per-client endpoints work for client-hosted installs; IP allow-listing. |
| — | **Eligibility (270/271)** or claims capabilities inside the product that an app can read. |
| — | **Roadmap / deprecations**: any announced Unity retirement, FHIR write roadmap, or program changes. |
| — | **BAA / compliance** requirements the program imposes on partners. |

## Output

Produce one Markdown file named **`veradigm-portal-findings.md`** and give it to Craig. Mark it "Internal — FrontRunnerHC" at the top. Structure:

1. **Session record**: date, account tier observed, which portal sections were reachable, what could not be opened.
2. **Findings**, one per line, in this form:
   `OQ-ID · finding (your words, ≤ 40 words) · [DOCS] portal page title · URL path · page date/version if shown · login-gated: yes/no`
   Use `[TESTED]` only if Craig authorized a sandbox call (state what was done and when). Use `[REPORTED]` for anything Craig tells you. Use `[INFERRED]` with basis. Use "not documented in portal" with the sections searched.
3. **Corrections** to the "already known" list above (what the portal says differently).
4. **Unity action table**: action name · purpose · read/write · documented parameters (names only) · user-attribution support · notes.
5. **New open questions** raised by what you read, each with a suggested owner (Product, Development, Orion team, Partnerships, Compliance, EMR vendor).
6. **Pages visited** (title + URL path), so the next session can re-check.

Do not include screenshots containing credentials, keys or sample patient data. Do not paste portal pages wholesale.

Craig will add the file to `crogersfrhc/ehr-product-requirements` under `veradigm/`, where the evidence pack (`veradigm-evidence-pack.md`) and open questions (`veradigm-open-questions-seed.md`) already live.

## When you finish

Tell Craig in three sentences: what the portal confirmed, what it contradicted, and which open questions are still open.
