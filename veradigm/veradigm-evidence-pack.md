# Veradigm — Evidence Pack

**Internal — FrontRunnerHC.** Not for distribution to Veradigm or any vendor. Names internal defects, decisions and platform status.

As of 2026-10-19 · Prepared for Craig Rogers (PM) · Input to a separate PRD-writing session. **This document contains no requirements.**

---

## 0. Working interpretation (read first)

Interview answers given by Craig in chat on 2026-10-19. Where Craig was unsure, the working interpretation used here is stated and the item is logged as an open question in `veradigm-open-questions-seed.md`.

| Topic | Answer / working interpretation | Basis |
|---|---|---|
| Target product | **Decided (Craig, 2026-10-19):** a Veradigm Sidecar — an embedded app like the athena Sidecar. | [REPORTED] Craig, PM, chat 2026-10-19 |
| Product lines in scope | Craig unsure. Working interpretation: **Veradigm Practice Management (PM)** and **Veradigm EHR** (the former Allscripts Professional EHR). TouchWorks is treated as out of scope (it is an Altera product, §5.7). Practice Fusion out of scope. | [REPORTED] Craig unsure; evidence §1.3, §5.7. OQ-1 |
| Existing solution | Craig unsure; classified from evidence only (§1). | [REPORTED] |
| Aliases | Veradigm, Allscripts, Unity, Ubiquity, `vera-`, `veradigm_pm`; Craig knows no others. | [REPORTED]; search §1.1 |
| Customers / Beta practice | Unknown. No customer named in any evidence; none are named here. | [REPORTED]; [NOT FOUND] §1.5. OQ-3 |
| Launch mechanism | Unknown to Craig; researched in §5. | [REPORTED] |
| Constraints | Unknown to Craig; found by search (§1.6, §7). | [REPORTED] |
| Comparison basis | Both athena and NextGen. | [REPORTED] |
| Vendor docs / sandbox | Craig holds a Veradigm developer-portal account but shared no material; this pack uses **public pages only**. Login-gated pages, sandbox and Unity reference are unread. | [REPORTED]. OQ-6 |

## 1. Scope and method

### 1.1 What was searched

| Source | Ref (commit, date) | How |
|---|---|---|
| `FrontRunnerHCV2/frhc-platform` `development` | `438029fe`, 2026-10-19 | `git grep -i` on the ref |
| `frhc-platform` `staging` | `5cc9f2c1`, 2026-10-16 | diff against development |
| `frhc-platform` `main` (production) | `4c87bbba`, 2026-10-16 | diff against development |
| `frhc-platform` unmerged branches | `feat/core-1666-ehr-embed-extract` @ `69eca606` (2026-10-13); `feat/odev-4374-per-tenant-vendor-resolution` @ `6e4b4cf9` (2026-10-18); branch list via `git ls-remote` | read-only fetch |
| `crogersfrhc/frontrunner-legacy-v1` (V1 archive) | `39340a1`, imported 2026-08-13 | `grep -ri` |
| Linear (all teams incl. CORE, FR, ODEV, PM) | read 2026-10-19 | issue search "Veradigm", "Allscripts", "Sidecar"; project search "Veradigm" |
| Sibling PRDs (Claude Docs) | read 2026-10-19 | full read (§6) |
| Veradigm public web pages | opened 2026-10-19 | §5 |

**Search terms** (all repos): `veradigm`, `allscripts`, `touchworks`, `practice ?fusion`, `unity`, `unityapi`, `ubiquity`, `twks`, `vera-`, `veradigm_pm`, `PlanCode_Veradigm`, `VERADIGM`, `veradigm.com`, `allscripts.com`; combined with `hl7`, `fhir`, `x12`, `270`, `sftp`, `batch`, `embed`, `sidecar`, `smart`, `launch`, `iframe`.

### 1.2 What could not be accessed or was not run

- Veradigm developer-portal content behind login; Unity API reference; Veradigm sandbox. **Nothing was run against any Veradigm system.**
- Production databases (no count of clients with Veradigm enabled; no count of V1 Veradigm activity).
- `frhc-platform` history before 2026-09-30 (the clone is shallow; "since" logs start there).
- No tests or builds were run in any repo.
- Microsoft 365/SharePoint (connector not authorized); sibling PRDs were read from Claude Docs instead.

### 1.3 Evidence rules used

`[CODE]` = re-opened and verified this session at the cited commit. `[DOCS]` = vendor or internal documentation. `[TICKET]` = Linear ticket or ADR. `[REPORTED]` = a person. `[INFERRED]` = conclusion, basis stated. `[NOT FOUND]` = searched and absent, terms stated. Repo docs inside `frhc-platform` (ADRs, `features.md`, requirements) are cited as `[TICKET]` (ADRs) or `[DOCS]` (internal docs), never as proof of behavior; code is cited as `[CODE]`. Path prefixes: **P:** = `frhc-platform` at `development` `438029fe` unless another ref is named. **V1:** = `frontrunner-legacy-v1` at `39340a1`.

---

## 2. Existing solution inventory

### 2.1 Classification

| Class | Present? | Where | Basis |
|---|---|---|---|
| Embedded app | **No** (V1 and V2) | — | [NOT FOUND] `embed`, `sidecar`, `smart`, `launch`, `iframe`, `toolbar`, `patientcontext` with `veradigm|allscripts|unity` in P: embed paths, embed ADRs, `adr-catalog.md`; V1: whole tree. Only athena has an embedded app (§2.4). |
| Backend adapter | **V1: yes, working code. V2: stub only.** | V1 `Common/FRHC.Veradigm`; P: `applications/services/orion/app/adapters/vendors/veradigm/` | [CODE] §2.2, §2.3 |
| Batch / file exchange | **V1: queue-based batch load and update** (not file exchange). V2: none. | V1 batch services | [CODE] §2.2 |
| Messaging / HL7 | No | — | [NOT FOUND] Veradigm terms in files matching `*hl7*` (P:, V1); `services/hl7-adapter` has no Veradigm reference. |
| FHIR / API integration | **V1: Veradigm Unity API (proprietary, JSON "MagicJson")**; no FHIR. V2: none. | V1 connector | [CODE] §2.2 |
| Manual / other | V1 website: manual "post" to Veradigm, Verify Credentials, Load Payors. V2 website: settings screen that saves nothing. | V1 website; P: website | [CODE] §2.2, §2.3 |

**Summary.** The only real Veradigm integration is **V1's server-side Unity API integration with Veradigm Practice Management** (batch plus manual posting). V2 (`frhc-platform`) has placeholders only. No Veradigm embedded app exists anywhere.

### 2.2 V1 (legacy) — what exists [CODE, V1 @ 39340a1]

- **API:** Unity web service, JSON. Token call `UnityService.svc/json/GetToken` (V1 `Common/FRHC.Veradigm/FRHC.Veradigm/Connectors/PracticeManagementConnector.cs:45`); all other calls `UnityService.svc/json/MagicJson` (same file :142).
- **Actions called** (same file): GetUserAuthentication :60, GetAppointmentById :74, GetPatientDemographics :87 and :100, GetPatientPolicy :114, SavePatientPolicy :181, SavePatient :205, GetPractitioners :218, GetLocations :231, SaveAccountContact :246, GetRequiredFields :261, SavePatientNote :288, GetCarriers :305.
- **Product:** Practice Management. [INFERRED] Basis: all config and classes are named `PracticeManagement…`; actions are PM-domain (carriers, plan codes, account contacts, appointments); no clinical (EHR) actions found.
- **Auth:** app-level credentials in service config plus per-client credentials and Ubiquity/Client/Tenant IDs; token accepted only if the response is exactly 36 characters (connector :50-53).
- **Per-client config table:** `ClientVeradigmConfiguration` with columns PracticeManagementUnityEndpointOverride, PracticeManagementUsername, PracticeManagementPassword, PracticeManagementUbiquityID, PracticeManagementClientID, PracticeManagementTenantID, EnableBlindUpdates, UseInsuranceDemographics, DefaultNoteTypeID (V1 `Database/FRHC.Database.OneProduction/FRHC.Database.OneProduction/dbo/Tables/ClientVeradigmConfiguration.sql:5-15`).
- **Routing:** client config `LivePost` = "VERADIGM" selects the Veradigm manager (V1 `Automation/FRHC.Sentinel.Common/BillingVendorManagers/BillingVendorManagerFactory.cs:10,19`).
- **Components:** core library, gRPC service, batch load service, batch update service, test console, tests (V1 `Common/FRHC.Veradigm/*`); posting manager (V1 `App/Integration/InternalPostBackService/FRHC.PostBack/Managers/PostingManager/VeradigmPostingManager.cs`); website config and web-service history (V1 `PatientRemedi/Website/...`).
- **Deployment state:** [INFERRED] V1 Veradigm is or was used in production by some clients. Basis: Linear ODEV-2409 treats "the legacy C# Veradigm adapter" as the reference implementation and plans a dual-write cutover [TICKET ODEV-2409]; ODEV-4159 states the PatientIdsOnly mode "in practice … serves Veradigm clients" [TICKET ODEV-4159]. Not verified against production.

### 2.3 V2 (`frhc-platform`) — what exists

| Item | Evidence | Deployed? |
|---|---|---|
| Orion Veradigm adapter: **empty stub** (`__all__: list[str] = []`) | [CODE] P: `applications/services/orion/app/adapters/vendors/veradigm/__init__.py:13` | Same on `main` [CODE] |
| `VendorType.VERADIGM` in Orion's vendor enum | [CODE] P: `applications/services/orion/app/domain/enums.py:24` | Same on `main` |
| Registry refuses Veradigm (test asserts `ConfigurationError`) | [CODE] P: `applications/services/orion/tests/adapters/vendors/test_registry.py:224` | — |
| Orion feature catalog: Veradigm "SCAFFOLD, not registered", 0 ADRs | [DOCS] P: `applications/services/orion/docs/features.md:92` | — |
| Orion real-time read/write handlers registered for athena only | [CODE] P: `applications/services/orion/app/core/events.py:435,466,477` | — |
| Veradigm in vendor-name lists only (secrets loader, DLQ module default) | [CODE] P: `applications/services/orion/app/adapters/secrets/loader.py:14`; `infrastructure/modules/orion-dlq/variables.tf:25` | — |
| Admin "Connection Settings — Veradigm" screen (fields: Unity Endpoint Override, Ubiquity ID, Practice Management Client ID, Tenant ID, username, password, Enable Blind Updates, plan-code mappings) | [CODE] P: `website/client/src/features/client-configuration/components/postback-config/connection-settings/VeradigmConnectionSettings.tsx:154,175,192,212,262` | Identical on `main` (`git diff --stat origin/main origin/development` on this file: no change) |
| That screen **saves nothing**; buttons only show a toast (see Defect F-01) | [CODE] same file :326-351; `.../connection-settings/index.tsx:148-160`; `.../postback-config/index.tsx:1682-1685` | Identical on `main` |
| Persisted Veradigm data: one flag per client, `VeradigmEnabled` | [CODE] P: `applications/migrations/versions/0001_baseline.py:804` | — |
| Integration-type seed: `('Allscripts', false)`, `('Veradigm', true)` (boolean = `is_active`) | [CODE] P: `applications/migrations/versions/0011_seed_integration_type.py:38,48` | — |
| Postback field matrix: 15 fields marked `veradigm: true` | [CODE] P: `website/client/src/shared/lib/postback-field-matrix.ts` (15 matches of `veradigm: true`; `InsPlanCode_Veradigm` :657) | — |
| Sign-up catalogs list "Veradigm (Allscripts)" and "Veradigm PM" | [CODE] P: `website/client/src/features/claritev-signup/lib/catalogs.ts:22-23` | — |

### 2.4 Reusable Sidecar platform (athena built; NextGen in progress)

- **athena Sidecar is in production**, currently accepting athena Preview launches in a demo window to 2026-10-29. [CODE] embed code present on `main` (e.g. P: `applications/services/action-handler/app/actions/embed/athena/routes.py:72` router prefix `/public/embed/athena`; `EMBED_ENABLED` true for stg and prod in `applications/.github/config/services.json.tpl:1282,1324` on `main`). Demo window: [CODE] P: `applications/services/action-handler/config/ehr_registration/prod.json:18` (`not_after` 2026-10-29); [DOCS] P: `applications/docs/runbooks/ehr-embed-athena.md:117-120` (§8, CORE-1932; revert CORE-1935).
- **Authentication model:** ADR-EMBED-001, "vendor-neutral, athena first", **Status: Proposed**, 2026-09-16 [TICKET] P: `applications/docs/adr/ADR-EMBED-001-ehr-embedded-app-authentication-model.md:1-4`. Embed user-mapping table carries a `vendor` column [CODE] P: `applications/services/action-handler/app/models/embed/ehr_user_mapping.py:43`.
- **Vendor seams:** Orion keys readers/writers by `VendorType` (only athena registered) [CODE] `events.py:435,466,477`. action-handler routes, settings and launch-context resolution are athena-specific [CODE] `routes.py:72`.
- **NextGen Sidecar: in progress on unmerged branches**, not on `development`, `staging` or `main`. ~40 `feat/`/`docs/` branches (e.g. `feat/core-1666-ehr-embed-extract`, `feat/odev-4383-nextgen-launch-vend-pairing-v2`) [CODE] `git ls-remote` 2026-10-19. Epic ODEV-4371 "PatientRemedi launched from NextGen Enterprise via App Launcher" In Progress [TICKET ODEV-4371].
- **Vendor-neutral SPA layer being extracted:** `features/ehr-embed/` with a plug-in interface whose vendor type allows only `"athena" | "nextgen"` [CODE] branch `feat/core-1666-ehr-embed-extract` @ `69eca606`, `website/client/src/features/ehr-embed/README.md:49`.

### 2.5 Planning state in Linear

| Item | State | Evidence |
|---|---|---|
| Project "Veradigm Solution" (P-ODEV-33), lead jfoster, initiative "Orion - Multi-Vendor EHR Integration Platform" | Backlog; created 2025-09-16; no start or target date | [TICKET] P-ODEV-33 |
| Epic ODEV-2409 "Veradigm Adapter — Test Matrix Parity & Migration (V2 reimplementation)": 13 stories, ~62 dev-days, legacy parity + dual-write cutover | Draft | [TICKET] ODEV-2409 |
| ODEV-2412, 2415, 2419, 2423, 2427, 2432, 2435, 2438, 2441, 2447, 2452, 2454, 2457 (sub-stories) | Draft | [TICKET] |
| ODEV-2579 VG-DECISION-GUARANTOR (Veradigm guarantor ADR) | In Progress since 2026-06-05 | [TICKET] ODEV-2579 |
| ODEV-1163 epic "Veradigm Bidirectional Integration (Slice 7)" and ODEV-1207/1208/1209/1238 | Canceled 2026-02-16 | [TICKET] |
| ODEV-4159 architect ruling (2026-09-24): "Veradigm is not built or planned in this platform yet and it will live inside Orion when we get to it." | Ruling recorded in ticket | [TICKET] ODEV-4159 |
| PM team (athena-embed product requirements) | No Veradigm items | [NOT FOUND] Linear search "Veradigm" returned no PM-team issue |

### 2.6 Constraints of record

- **ADR-PLAT-001** (Accepted 2026-06-17): scope is locked to Eligibility, Discovery, Demographics, Credit; "Orion, nextgen/AMD/Xifin, and anything not feeding the four are parked/out of scope." [TICKET] P: `applications/docs/adr/ADR-PLAT-001-stabilize-in-place-no-rewrite.md:52-54`. [INFERRED] Veradigm Orion work falls under "Orion … parked", because the architect ruled Veradigm will live inside Orion (ODEV-4159).
- **Amendment A1** (Accepted 2026-10-19, signed by Leadership decision owner) un-parks a bounded NextGen slice for the NextGen Sidecar only; it is on `development` and not yet on `staging`/`main` [TICKET] same file :90-160; [CODE] diff `origin/main..origin/development` on this file shows +73 lines.
- A Veradigm Sidecar therefore needs its own leadership decision equivalent to A1 [INFERRED from A1's pattern].

---

## 3. Capability trace

Status axis: **V2 PLATFORM for Veradigm** (does it exist for Veradigm in `frhc-platform`). V1 status is given separately because it is the only working Veradigm code. Status values: Working / Partial / Broken / Absent / Unknown. Checked 2026-10-19 at P: `438029fe` and V1 `39340a1`.

| # | Capability | V2 status | V1 status | Direction | Fields (names only) | Trigger | User sees | Failure behavior | Evidence |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Launch and patient/user/practice context | Absent | Absent | — | — | — | — | — | [NOT FOUND] §2.1 terms. Vendor offers SMART EHR launch for Veradigm EHR only (§5.2) [DOCS] |
| 2 | User linking to a FrontRunner user | Absent for Veradigm (athena pairing exists, vendor column present) | Absent | — | — | — | — | — | [CODE] P: `.../models/embed/ehr_user_mapping.py:43`; athena-only routes `routes.py:72` |
| 3 | Practice-to-client mapping | Absent (only the per-client `VeradigmEnabled` flag) | Working: per-client Ubiquity/Client/Tenant IDs and endpoint identify the Veradigm tenant | FRHC config → Veradigm login | Ubiquity ID, PM Client ID, Tenant ID, endpoint override | Admin config | V1 website config | Login fails → error | [CODE] V1 `ClientVeradigmConfiguration.sql:5-10`; connector :164-171; P: `0001_baseline.py:804` |
| 4 | Patient identity and loading into PatientRemedi | Absent; by-patient retrieve deferred until a Veradigm Orion adapter exists | Working (batch): appointment ID → appointment → demographics → policies; `FRHC_{PatientID}_{yyyyMM}` builds a placeholder visit | Veradigm → FRHC | appointment ID, patient ID, account ID, MRN | Batch load queue; gRPC GetAccountWebServiceID | Loaded case in V1 | Error row marked and never retried (F-12) | [CODE] V1 connector :74,87,114; [TICKET] ODEV-4159 D-1/D-2 |
| 5 | Demographics read | Absent | Working | Veradigm → FRHC | first/middle/last name, sex, SSN, DOB, MRN, address, phones, email, guarantor | Batch load | V1 case | as #4 | [CODE] V1 connector :87,100 |
| 6 | Demographics write-back | Absent | Working: sends a field only if allowed for the client and different from live Veradigm data | FRHC → Veradigm | FirstName, MiddleName, LastName, Address1/2, City, State, Zip, Gender, DateOfBirth, SSN, Email, HomePhone, WorkPhone | Batch update; manual PostData | Case note + status | Partial writes not rolled back (F-11) | [CODE] V1 connector :205; `Utilities/SavePatientRequestBuilder.cs` (15 Demo* field references) |
| 7 | Insurance read | Absent | Working (patient-level policies; P/S mapped to COB 1/2; expired dropped) | Veradigm → FRHC | certificate, subscriber, carrier, coverage priority, relationship, group, plan codes | Batch load | V1 case | as #4 | [CODE] V1 connector :114 |
| 8 | Insurance write (update, add, coverage order) | Absent | Working with vendor workarounds: match by certificate; priorities P/S/X; demote-then-reassign two-pass; subscriber contact update; **visit-level insurance write absent** | FRHC → Veradigm | carrier, certificate, effective/termination dates, group, relationship, plan code, specialist plan code, priority | Batch update; manual PostData | Case note + status | Partial writes not rolled back (F-11) | [CODE] V1 connector :181,246; `Utilities/PolicyUpdatePayloadBuilder.cs:102` ("workaround for an existing veradigm bug") |
| 9 | Eligibility (270/271) | Partial: platform Eligibility product exists and is in scope; not connected to Veradigm | Uses FRHC's own 270/271 results for plan codes and notes; nothing read from or written to Veradigm eligibility | FRHC internal | copay, eligibility status | Batch | Plan code in Veradigm | — | [TICKET] ADR-PLAT-001:52; [CODE] V1 `Utilities/PolicyUpdatePayloadBuilder.cs:184` (reads FRHC 270/271 result) |
| 10 | Insurance discovery | Partial: platform Discovery product exists; not connected to Veradigm | V1 posts discovery results as Veradigm notes | FRHC → Veradigm (note) | note text | Realtime post | Veradigm note | — | [TICKET] ADR-PLAT-001:52; [CODE] V1 `VeradigmPostingManager.cs:51-63` (PostCreditInfo, PostDiscoveryInfo → PostNote) |
| 11 | Redetermination | Unknown | Absent | — | — | — | — | — | [NOT FOUND] `redetermin` in embed paths (P:) and V1 Veradigm code |
| 12 | Credit / financial disposition | Absent for Veradigm (cross-vendor CW-055 story is Draft) | Credit details written into Veradigm notes; no financial-class/FSC write | FRHC → Veradigm (note) | FPL, income, score (in note) | Post | Veradigm note | — | [TICKET] ODEV-2365 (Draft), ODEV-2435; [CODE] V1 `Utilities/SavePatientNoteRequestBuilder.cs:79-83` |
| 13 | Good Faith Estimates | Absent for Veradigm (athena Sidecar has a GFE tab) | Absent | — | — | — | — | — | [CODE] P: `website/client/src/features/athena-embed/components/EmbedGfeTab.tsx` exists; [NOT FOUND] `gfe|good faith` in V1 Veradigm |
| 14 | Claim validation | Absent (athena has a placeholder mount) | Absent | — | — | — | — | — | [CODE] P: `.../athena-embed/components/ClearlineTab.tsx` exists; [NOT FOUND] claim actions (`claim`, `charge`) among V1 connector actions (connector :60-305) |
| 15 | Appointment and claim data | Absent | Appointment read Working (GetAppointmentById); claims Absent | Veradigm → FRHC | appointment ID, DOS, location, practitioners | Batch load | V1 case | DOS change aborts post | [CODE] V1 connector :74,218,231 |
| 16 | Readiness rollup | Absent for Veradigm (athena has one; vendor-neutral copy on branch) | Absent | — | — | — | — | — | [CODE] P: `.../athena-embed/components/ReadinessRollup.tsx`; branch `ehr-embed/lib/readiness.ts` |
| 17 | Roles and field-level permissions | Absent for Veradigm | Web-service history page limited to an admin role; no field-level roles found | — | — | — | — | — | [CODE] V1 `PatientRemedi/Website/PatientRemedi/Controllers/Tools/VeradigmWebServiceHistoryController.cs:14` (`Authorize(Roles = "FRMDAdministrator")`) |
| 18 | Per-client configuration and posting allow-lists | **Broken**: settings screen accepts input but saves nothing; allow-list matrix defined (15 fields) but not wired to any Veradigm writer | Working: per-client settings, plan-code config, `C_PostBackField.Veradigm` bit | Admin → config | see #3; plan-code mapping fields | Admin save | V2: "aren't saved to the platform yet" notice | V2: silent local-only save (F-01) | [CODE] P: `.../connection-settings/index.tsx:148-160`, `.../postback-config/index.tsx:1682-1685`; V1 `ClientVeradigmConfiguration.sql:5-15` |
| 19 | Audit logging | Absent for Veradigm calls (only `VeradigmEnabled` flag changes are audited) | Working but logs full request and response including PHI and the session token (F-08) | — | Request, Response, Action, RequestUri, RequestDate | Every logged call | Admin history page | — | [CODE] V1 `dbo/Tables/VeradigmAPILogging.sql:5-9`; P: `applications/lambda-functions/client-audit-event-handler/main/constants.py:93` |
| 20 | Error and outcome reporting | Absent | Working: case note and case status; explicit messages on manual post | FRHC | case status, note | Post | Case note | Batch fails whole on one error (F-11) | [CODE] V1 `VeradigmPracticeManagement.cs:50,81,114` (`Task.WaitAll`) |
| 21 | Idempotency and duplicate protection | Absent | Partial: field-level diff against live data; no request key; no in-progress claim on queue rows | — | — | — | — | Possible re-processing [INFERRED] | [CODE] V1 `Repositories/DataRepository.cs:102,111` |
| 22 | Session and token handling | Absent | Token per connector instance; no refresh; vendor lockout flag ignored | — | token | Each batch/operation | — | Long batch may outlive token [INFERRED] | [CODE] V1 connector :50-53; `Models/GetUserAuthenticationResponse.cs:25` |
| 23 | Observability | Absent (Veradigm observability story canceled; DLQ module default lists veradigm) | App Insights + API logging | — | — | — | — | — | [TICKET] ODEV-1238 (Canceled); [CODE] P: `infrastructure/modules/orion-dlq/variables.tf:25` |
| 24 | Rate limits | Absent | Partial: bulkhead of 100 concurrent calls per connector; no retry/timeout policy | — | — | — | — | — | [CODE] V1 `PracticeManagementFactory.cs:43` |
| 25 | Practice onboarding and offboarding | Partial: `VeradigmEnabled` flag and setup-wizard rule; credentials cannot be stored; no offboarding | Working onboarding (config + Verify Credentials + Load Payors); offboarding not found | Admin | see #3 | Admin | V1 config page | V2 buttons are toasts (F-01) | [CODE] P: `VeradigmConnectionSettings.tsx:326-351`; [NOT FOUND] `offboard`, `deactivate` in Veradigm code |

---

## 4. Defects (numbered findings — not fixed)

| # | What happens | Where | Impact | Evidence |
|---|---|---|---|---|
| F-01 | V2 Veradigm connection settings accept input but write nothing to the platform; "Verify Credentials" and "Load Payors" only show a toast. A notice says the settings "aren't saved to the platform yet". Present in production. | P: `.../connection-settings/VeradigmConnectionSettings.tsx:326-351`; `.../connection-settings/index.tsx:148-160`; `.../postback-config/index.tsx:1682-1685`; identical on `main` | No Veradigm client can be configured on V2; risk that staff believe setup succeeded | [CODE] |
| F-02 | `InsSpecialistPlanCode_Veradigm` is marked not supported for Veradigm in the postback field matrix, although it is a Veradigm-only field V1 posts | P: `website/client/src/shared/lib/postback-field-matrix.ts:667-673` | Allow-list would block a field V1 sends | [CODE] |
| F-03 | Platform docs say Veradigm uses OAuth 2.0 client credentials; V1 code and the Orion test matrix use the Unity token flow with username/password and Ubiquity IDs | P: `applications/services/orion/app/platform/docs/adr/ADR-ORN-018-vendor-authentication.md:27`; `applications/docs/projects/orion/requirements.md:363`; V1 connector :45,60 | Wrong auth design if built from the ADR; docs/code disagreement | [TICKET] + [CODE] |
| F-04 | Orion requirements say Veradigm has "no project yet"; Linear has project P-ODEV-33 (created 2025-09-16) | P: `applications/docs/projects/orion/requirements.md:41,363` | Stale planning doc | [DOCS] + [TICKET] |
| F-05 | Orion BAA register lists Veradigm (2024-01-01 to 2027-01-01) citing a PDF that is not in the repo; the BAA-verification story was canceled | P: `applications/services/orion/docs/compliance/baa-vendor-register.md:20`; no `baa/*.pdf` in tree | Unverified BAA status for a PHI vendor; route to Compliance | [CODE] + [TICKET] ODEV-1226. [INFERRED] the dates may be placeholders (no PDF exists for any vendor: 0 files under `baa/*.pdf`) |
| F-06 | V1 commits Veradigm application-account credential values (app user name, app password, app name) in service config; Application Insights keys also present | V1 `Common/FRHC.Veradigm/FRHC.Veradigm.BatchLoadService/appsettings.json:7-9`; `.../BatchUpdateService/appsettings.json:7-9`; `.../TestConsoleApp/appsettings.json:7-9`; `.../gRPC.Service/appsettings.json:34-36`; test helper `FRHC.Veradigm.Tests/IntegrationHelpers/IntegrationUtilities.cs` | Credential exposure; route to Security (not an Orion topic). Values not reproduced. | [CODE] (presence checked; values not read into output) |
| F-07 | V1 stores each client's Veradigm PM password in plain text, sends it to the browser, and shows it in a plain text input | V1 `ClientVeradigmConfiguration.sql:7`; `PatientRemedi.Repository/PostBackConfigRepository.cs:484,509`; `clientapp/src/components/ClientManagement/PostbackConfig.vue:272` (`type="text"`) | Credential exposure; route to Security | [CODE] |
| F-08 | V1 logs full Unity request and response bodies (including demographics/SSN) and the session token; a debug setting emails the mapped request | V1 `dbo/Tables/VeradigmAPILogging.sql:5-6`; `FRHC.Veradigm.gRPC.Service/Services/VeradigmService.cs:107,128-129` | PHI in logs (contradicts platform invariant 3); route to Compliance/Security | [CODE] |
| F-09 | V1 note posts hard-code note type "22" and subject "FRHC Note" and do not record who posted | V1 `FRHC.Veradigm/VeradigmPracticeManagement.cs:180-186` | Writes not attributed to the user | [CODE] |
| F-10 | V1 builds write payloads as XML by string interpolation; only note text is escaped | V1 `Models/SavePatientNoteRequest.cs:20` (escape) vs `Models/SavePatientPolicyRequest.cs` (13 interpolations, no escape) | Malformed or altered writes if data contains markup | [CODE] |
| F-11 | V1 batch fails as a whole if one account throws (`Task.WaitAll`); earlier writes in a failed post are not rolled back | V1 `VeradigmPracticeManagement.cs:50,81,114`; `Utilities/VisitUpdater.cs:463-465` | Partial writes; whole-batch failure | [CODE] |
| F-12 | V1 queue rows marked in error are never selected again (no retry) | V1 `Repositories/DataRepository.cs:102,111,156-176` | Silent loss of work items | [CODE] |
| F-13 | V1 ignores the Unity "Lockout" field and never refreshes the token | V1 `Models/GetUserAuthenticationResponse.cs:25`; connector :50-53 | Locked accounts and expired tokens surface as generic failures [INFERRED] | [CODE] |
| F-14 | V1 plan-code list is filtered by billing vendor, not by client | V1 `Repositories/DataRepository.cs:41` | Clients may see each other's plan codes [INFERRED] | [CODE] |
| F-15 | Planning contradiction: ODEV-2409 plans a parity cutover from a live legacy adapter, ODEV-2579 is In Progress, but the architect ruled (2026-09-24) that Veradigm is "not built or planned" and ADR-PLAT-001 parks Orion | [TICKET] ODEV-2409, ODEV-2579, ODEV-4159; ADR-PLAT-001:52-54 | Unclear whether any Veradigm work is sanctioned | [TICKET] |
| F-16 | Orion test matrix cites a Veradigm workflow inventory document that is not in the repo | P: `applications/docs/projects/orion/orion-integration-test-scenario-matrix-v2.md:549`; [NOT FOUND] `veradigm-workflow-inventory` in the `development` tree | Source of Veradigm scenarios unverifiable | [DOCS] |

---

## 5. Vendor facts (public pages only, opened 2026-10-19)

All items [DOCS] unless marked. No login-gated material, no sandbox use.

### 5.1 Developer program
- Program is "Veradigm Connect" (formerly the Allscripts Developer Program). Tiers: **Open** (free, FHIR only) and **Integrator** (FHIR + proprietary Unity APIs), with Integrator sub-tiers Bronze to Platinum Plus; Integrator is billed monthly or annually; no prices published. Source: Veradigm Developer Program, "Learn More" (developer.veradigm.com/Home/LearnMore), © 2026.
- Integrators can deploy to clients only after Veradigm tests and **certifies** the integration. Same source.
- FHIR apps: register for client ID/secret, request production access, Veradigm reviews; clients then activate the app in their license portal. Source: "ProcessOverview" (developer.veradigm.com/Fhir/ProcessOverview), © 2026.
- Unity sandbox and Unity approval process: **not publicly documented**. Partner Testing Environments page returned an error.

### 5.2 Launch and authentication
- **SMART on FHIR EHR launch** is documented for **Veradigm EHR** and requires a Connect **Integrator** tier or above. Source: "SMARTonFHIR" (developer.veradigm.com/Fhir/SMARTonFHIR), © 2026.
- SSO for FHIR R4 apps requires **Veradigm EHR 24.5 or later**; for earlier versions Veradigm recommends Unity for app launch. Source: ProcessOverview pages. (One search summary contradicted this; the pages opened say 24.5+.)
- **Practice Management launch: not publicly documented.** No launcher, tab or URL-launch mechanism found for PM.
- FHIR OAuth: SMART v1 user apps with EHR credentials; system apps use client credentials with a signed JWT. Source: ProcessOverview, SMARTonFHIR.
- **Unity auth model and whether writes can be attributed to the launching user: not publicly documented.** V1 code shows an app token plus per-client service user (§2.2) — that is a service account, not the launching user [CODE, INFERRED for attribution].

### 5.3 Read and write capability
- **FHIR API is read-only** — no write-backs. Source: developer.veradigm.com/content/fhir/ProcessOverview_index.html.
- Unity is bidirectional and is **required** for reading or writing PM patient demographic, appointment or financial data. Same source.
- FHIR R4/USCDI supported; DSTU2 support ended 2025-06-01. Source: "Introduction" (developer.veradigm.com/Fhir/Introduction).
- Insurance update/add/coverage priority, eligibility inside the product, claims detail: **not publicly documented**. V1 code shows Unity actions for policy save and priority (§2.2) [CODE].

### 5.4 Idempotency, locking, rate limits, hosting
- Not publicly documented: idempotency, duplicate protection, locked-record behavior, rate limits, per-client Unity endpoint resolution. V1 evidence: per-client endpoint override, a lockout field, a duplicate-blanking quirk on SaveAccountContact [CODE V1 connector :270-276].

### 5.5 Sandbox and approval
- FHIR sandbox on sign-up; production requires Veradigm approval then client activation [DOCS ProcessOverview]. Unity sandbox unknown. Craig holds a portal account [REPORTED]; whether it includes Unity or sandbox access is unknown (OQ-6).

### 5.6 Business and roadmap
- No Unity deprecation announcement found (searches returned unrelated results; not proof).
- [NEWS, single source: telecareaware.com/tag/veradigm, items to 2026-05-29] Veradigm ended its sale process (Jan 2025), the SEC closed its investigation without enforcement recommendation (Feb 2026), staff and product reductions, combined 2023–24 10-K filed (May 2026) with relisting planned. Not corroborated from primary filings in this session.

### 5.7 Product mapping (affects scope)
- Veradigm's ProcessOverview lists **TouchWorks as an Altera product** (Altera = the former Allscripts hospital/large-practice business). [DOCS] ProcessOverview. "Veradigm EHR" appears to be the renamed Professional EHR [INFERRED; third-party listing only].
- Practice Fusion is a separate product with its own developer center [DOCS practicefusion.com/developer-center].

### 5.8 Where docs and code disagree
- Platform ADR says OAuth client credentials; V1 code uses Unity token + username/password (F-03).
- Public docs offer OAuth only for read-only FHIR; every V1 write uses Unity.

---

## 6. Cross-EMR comparison

Sources: athena V1 PRD ("FrontRunner for athenahealth (Embedded App, V1) — PRD", 2026-10-02); athena 2.0 PRD ("Athena Integration PRD — FrontRunner 2.0", v1, 2026-10-06); NextGen Sidecar PRD (v1 draft, 2026-10-07); NextGen Integration PRD (v1 as-built, 2026-10-05, pins `main` @ `df392956`). All read 2026-10-19 [DOCS internal]. Veradigm column = this pack. Classification: CORE-CANDIDATE (same across EMRs, holds for ≥2), EMR-SPECIFIC, UNKNOWN.

| Capability | athena (V1 / 2.0 PRDs; platform) | NextGen (Sidecar / Integration PRDs) | Veradigm (this pack) | Class |
|---|---|---|---|---|
| Launch mechanism | SMART launch in athenaOne (V1 EMB-FND-01; 2.0 LCH-01); built, prod | App Launcher: Patient Info Bar tab, NGE 6.2021.1+ (SC-LCH-01); template window 5.9.2+ (SC-LCH-02) | SMART EHR launch, Veradigm EHR only, Integrator tier; SSO needs 24.5+; PM unknown | EMR-SPECIFIC |
| Patient/user/practice context at launch | From token/launch (V1 §8.1) | Patient, encounter, user, practice (SC-LCH-03/04) | Unknown | CORE-CANDIDATE (requirement shape); mechanism EMR-SPECIFIC |
| User linking | Emailed code pairing (V1 EMB-FND-07–11; 2.0 LCH-02); built | Link before data (SC-ACC-11); method open | Absent | CORE-CANDIDATE |
| Practice-to-client mapping | Practice ID → provider group (V1 §8.3; 2.0 PG-01/02) | From API authorization; refuse if no match (SC-ACC-12) | Ubiquity/Client/Tenant IDs (V1) | CORE-CANDIDATE (rule); keys EMR-SPECIFIC |
| Patient identity / load | Create/update at load (V1); IDs + department (2.0 PID) | Auto-create or find (SC-DAT-06) | V1 load by appointment ID; V2 absent | CORE-CANDIDATE |
| Demographics read | Built | SC-DAT-01 | V1 Working | CORE-CANDIDATE |
| Demographics write-back | Per-field Post Data, through Orion (2.0 PST-01) | Approved, allowed fields (SC-WRT-01/02); required fields (WRT-08) | V1 Working, diff-based; Unity only | CORE-CANDIDATE (behavior); transport EMR-SPECIFIC |
| Insurance read | Built | SC-DAT-02 (P/S/T) | V1 Working (patient-level) | CORE-CANDIDATE |
| Insurance write / coverage order | Update in place, add at next slot, never delete to reorder (2.0 PST-18) | Update in place (WRT-03), add once (WRT-04), order (WRT-05) | Priorities P/S/X with two-pass demote workaround; no visit-level write | CORE-CANDIDATE (requirements); ordering mechanics EMR-SPECIFIC |
| Eligibility | Shown (V1 §11; 2.0 EMB-03) | Findings + re-run (SC-DAT-03/07) | Platform product exists; not connected | CORE-CANDIDATE |
| Insurance discovery | Shown (2.0 EMB-04) | Findings (SC-DAT-03) | V1 posted as notes | CORE-CANDIDATE |
| Redetermination | V1 mentions credit for Medicaid redetermination | Findings (SC-DAT-03) | Unknown | UNKNOWN |
| Credit / financial disposition | Credit check (2.0 EMB-02) | Open (§18) | Credit in notes only; no FSC write | UNKNOWN |
| Good Faith Estimates | 2.0 EMB-11 (P1); built in platform | Open | Absent | UNKNOWN |
| Claim validation | 2.0 EMB-12 (P1); placeholder in platform | 2027 non-goal | Absent | UNKNOWN |
| Appointment / claim data | Live reads (2.0 DS-05) | Appointments open; claims 2027 | Appointment read in V1 | EMR-SPECIFIC |
| Readiness rollup | 2.0 EMB-08; built | Suggested P1 | Absent | CORE-CANDIDATE |
| Roles / field permissions | Server-side roles (2.0 RBAC-01) | NextGen groups gate launch (SC-ACC-01) | Absent | CORE-CANDIDATE (FrontRunner side); EHR gating EMR-SPECIFIC |
| Per-client config / allow-lists | Field list per client, server rejects off-list (2.0 FLA-01/06) | Honors per-client post list (SC-ACC-06) | V1 working; V2 broken (F-01) | CORE-CANDIDATE |
| Audit logging | Every read/write, redacted (2.0 AUD-01) | Every write, no PHI (SC-WRT-11, SC-NFR-05) | V1 logs PHI (F-08) | CORE-CANDIDATE |
| Error / outcome reporting | Never a false success (2.0 OUT-02) | Written / refused / not sure, per field (SC-WRT-07/14) | V1 case notes/status | CORE-CANDIDATE |
| Idempotency / duplicates | Uncertain writes not retried (2.0 PST-11) | No duplicate on retry (SC-WRT-04) | V1 diff only; no key; vendor support unknown | CORE-CANDIDATE (requirement); vendor support EMR-SPECIFIC |
| Session / token; write attribution | 60-min embed token, no refresh (platform); writes via Orion credential | 60 min (SC-LCH-08); launching user as author (SC-WRT-06, pending) | V1 service account; attribution unknown | CORE-CANDIDATE (session); attribution EMR-SPECIFIC |
| Observability | Ops view (2.0 AUD-03); probe + alarms built | Failures per practice (SC-NFR-11) | Absent | CORE-CANDIDATE |
| Rate limits | 2.0 X-01 vendor limits | SC-NFR-09 | Not documented; V1 bulkhead | EMR-SPECIFIC |
| Practice onboarding / offboarding | Setup checklist (2.0 CFG-10); no offboarding | 7-step setup (SC-ACC-08/09); no offboarding | V1 config + verify; V2 broken | CORE-CANDIDATE |

**Sibling statements contradicted or affected by findings**
- athena 2.0 PRD says 1.0 posting runs on "a vendor-agnostic post-back service also used by Xifin, Veradigm and AdvancedMD" — consistent with V1 (`LivePost` routing). It is not a V2 capability: V2's Veradigm connection settings write nothing (F-01). [DOCS] + [CODE]
- NextGen Integration PRD lists Veradigm adapters as out of scope "except where they share the Orion platform" — consistent; no contradiction.
- NextGen Sidecar PRD and ADR-PLAT-001 A1 confirm the un-parking pattern a Veradigm Sidecar would need (§2.6).
- No sibling PRD mentions Veradigm launch or Unity; none is contradicted on Veradigm specifics.

---

## 7. Dependencies

| # | Dependency | Owner side | Needed | Current state | Blocks | Hard blocker? |
|---|---|---|---|---|---|---|
| DP-1 | Leadership decision to un-park Veradigm Orion work for a Sidecar (ADR-PLAT-001 amendment like A1) | FrontRunner leadership | A signed scope ruling | Parked [TICKET ADR-PLAT-001:52-54]; architect: "not built or planned" [TICKET ODEV-4159] | All V2 Veradigm build | Yes |
| DP-2 | Veradigm adapter in Orion (read, write, auth) | Orion team | Real-time read and write for demographics and insurance | Empty stub [CODE] | Every data capability | Yes |
| DP-3 | Vendor-neutral embed layer that admits a third vendor | Development | `ehr-embed` plug-in and per-vendor action-handler routes | On an unmerged branch; vendor type allows athena and NextGen only [CODE branch] | Launch, linking, Sidecar UI | Yes |
| DP-4 | ADR-EMBED-001 acceptance and a Veradigm amendment | Development | Auth model for a Veradigm launch | Proposed [TICKET] | Launch | Yes (for production) |
| DP-5 | Credential storage for Veradigm connection settings | Development / Security | Secure per-client storage and a write path | None (F-01) [CODE] | Onboarding | Yes |
| DP-6 | Veradigm Connect Integrator membership and certification | Partnerships / Veradigm | Unity access (writes) and SMART launch | Not started [REPORTED: account only]; tier and cost unknown | Writes, launch, production | Yes |
| DP-7 | Veradigm sandbox with Unity and SMART launch | Veradigm | Test environment | Unknown [NOT FOUND public] | Testing | Yes |
| DP-8 | Unity API reference (auth, user attribution, insurance priority, idempotency, limits) | Veradigm | Documentation | Not public [DOCS] | Spec of writes | Yes |
| DP-9 | Supported launch for Practice Management | Veradigm | Confirmation | Not documented [DOCS] | PM-only practices | Unknown — depends on OQ-1 |
| DP-10 | BAA with Veradigm confirmed | Compliance | Signed BAA on file | Register row cites a missing PDF (F-05) | Production PHI flow | Yes |
| DP-11 | Platform Eligibility, Discovery, Credit services | Platform | Reuse | In scope and running [TICKET ADR-PLAT-001:52] | Findings panels | No |
| DP-12 | Production Orion bring-up | Orion team / Platform | Orion prod | Listed as a prerequisite for the NextGen Sidecar (ODEV-3943) [DOCS NextGen PRDs] | Production Sidecar | Yes [INFERRED same as NextGen] |

## 8. Risks

| # | Risk | Impact | Possible mitigation |
|---|---|---|---|
| R-1 | Veradigm's FHIR API cannot write; all write-back needs proprietary Unity behind a paid, certified tier | Cost, contract and certification lead time before any write ships | Engage Partnerships early; price tiers; confirm Unity access with the portal account |
| R-2 | Launch is documented only for Veradigm EHR 24.5+; PM launch undocumented | Practices on PM only, or on older EHR versions, may not be able to launch a Sidecar | Confirm with Veradigm; consider a non-embedded fallback for PM-only practices |
| R-3 | Writes may be attributed to a service account rather than the launching user | Audit and accountability gap | Ask Veradigm whether Unity supports user-level tokens |
| R-4 | Planning contradiction (F-15) and parked scope | Effort spent on unsanctioned work | Leadership ruling (DP-1) before build |
| R-5 | Known Veradigm write quirks (contact fields blanked; priority bug needing two passes) | Data loss or wrong coverage order | Carry V1's workarounds as test cases; confirm with Veradigm whether still needed |
| R-6 | V1 security and PHI defects (F-06 to F-08) could be copied into V2 by "parity" work | PHI in logs; credential exposure | Treat V1 as a behavior reference, not a design reference; route F-06–F-08 to Security/Compliance now |
| R-7 | Vendor business instability (restructuring, delisting history) | Partnership continuity | Contract terms; monitor filings |
| R-8 | Product mapping unclear (TouchWorks is Altera) | Wrong scope or wrong vendor relationship | Resolve OQ-1 with Partnerships |
| R-9 | No V2 Veradigm customer data or Beta practice identified | No validation path | Identify V1 Veradigm clients from production data (Ops) |

## 9. Decisions needed

| # | Decision | Options | Recommended (analyst proposal, awaiting Craig) | Proposed owner |
|---|---|---|---|---|
| DN-1 | Is a Veradigm Sidecar on the roadmap, and does Orion Veradigm work get un-parked? | (a) amend ADR-PLAT-001 for a bounded Veradigm Sidecar slice, as A1 did for NextGen; (b) stay parked | (a) only after DN-2 and DN-3 are answered, since vendor access gates everything | Leadership |
| DN-2 | Which Veradigm products are in scope? | PM only; Veradigm EHR only; both | Decide from the customer list (OQ-3); launch evidence favors Veradigm EHR, V1 history favors PM | Product + Partnerships |
| DN-3 | Which Veradigm Connect tier to buy? | Open (FHIR read only) vs an Integrator tier | Integrator — Open cannot write | Partnerships |
| DN-4 | Write credential model | Launching user's credential; service account naming the user; service account only | Ask Veradigm first; mirror the NextGen Sidecar decision (open there too) | Product + Development |
| DN-5 | How to treat V1 Veradigm behavior | Byte-level parity (ODEV-2409 premise) vs behavior reference only | Behavior reference for the Sidecar; parity only matters if batch posting migrates | Product + Orion team |
| DN-6 | Reconcile ODEV-2409/ODEV-2579 with the architect ruling | Close, keep Draft, or re-scope under a Sidecar epic | Re-scope once DN-1 is decided | Product + Development |
| DN-7 | Fallback for practices that cannot launch | Full web app; none | Full web app (athena V1 precedent: non-athena clients use the full app) | Product |

## 10. Sources

**Repos (all read-only)**
- `FrontRunnerHCV2/frhc-platform` @ `development` `438029fe` (2026-10-19), `staging` `5cc9f2c1`, `main` `4c87bbba` (2026-10-16); branches `feat/core-1666-ehr-embed-extract` @ `69eca606`, `feat/odev-4374-per-tenant-vendor-resolution` @ `6e4b4cf9`.
- `crogersfrhc/frontrunner-legacy-v1` @ `39340a1` (2026-08-13).

**Linear** (read 2026-10-19): P-ODEV-33; ODEV-1163, 1167, 1207, 1208, 1209, 1226, 1238, 2365, 2409, 2412, 2415, 2419, 2423, 2427, 2432, 2435, 2438, 2441, 2447, 2452, 2454, 2457, 2579, 4159, 4169, 4371; CORE-1666, 1676, 1677.

**ADRs**: ADR-PLAT-001 (+ Amendment A1), ADR-EMBED-001, ADR-ORN-018 — paths in §2.

**Internal PRDs (Claude Docs, read 2026-10-19)**
- NextGen Sidecar — PRD — https://claude.ai/code/artifact/f0195b75-e9f5-45a6-88d1-bbdbfb8c1f84
- FrontRunner × NextGen Integration (Orion Adapter, V2) — PRD — https://claude.ai/code/artifact/10b4eb0f-a2c3-490a-ae3a-098e900362b4
- Athena Integration PRD — FrontRunner 2.0 — https://claude.ai/code/artifact/25059b45-f429-4ceb-affd-022aeb7509b3
- FrontRunner for athenahealth (Embedded App, V1) — PRD — https://claude.ai/code/artifact/91b59937-5c59-47d7-94cd-4c49da830a49

**Vendor and public pages (opened 2026-10-19)**
- https://developer.veradigm.com/
- https://developer.veradigm.com/Home/LearnMore
- https://developer.veradigm.com/Fhir/Introduction
- https://developer.veradigm.com/Fhir/ProcessOverview
- https://developer.veradigm.com/Fhir/SMARTonFHIR
- https://developer.veradigm.com/content/fhir/ProcessOverview_index.html
- https://expo.veradigm.com/Developers
- https://www.practicefusion.com/developer-center/
- https://practicefusion.com/fhir/
- https://telecareaware.com/tag/veradigm/ (news, single source)

---

## Quality gate results (2026-10-19)

- Findings without an evidence label: **0** (every defect, inventory row and capability row carries a label and reference).
- `[CODE]` references: every one was re-opened this session at the cited commit (P: `438029fe` or the named ref; V1: `39340a1`).
- `[NOT FOUND]` items list their search terms (§1.1 and in-row).
- Secrets/PHI: none reproduced; F-06 reports existence and location only. Scan result recorded in the session reply.
- Writes: only this folder in `crogersfrhc/ehr-product-requirements`. No command changed any system; no Veradigm system was contacted.
