# CLAUDE.md — read this before anything else in this repo

**Internal.** This repo holds product evidence packs and PRDs for EHR
integrations (one folder per EHR). Its content is product-management input.

## Authority boundary

- **Nothing here is evidence about the FrontRunner platform.** The only source
  of truth for what the platform does is `FrontRunnerHCV2/frhc-platform`. A
  `[CODE]` citation in a pack records what was true at the cited commit on the
  cited date; re-verify it against `frhc-platform` before relying on it.
- `[INFERRED]` items and open questions are not findings. Do not promote them to
  requirements without the evidence or decision they are waiting on.
- Vendor-documentation facts (`[DOCS]`) are summaries with citations, never
  copied vendor text. Material from login-gated vendor portals stays here only
  as summary.
- Decision labels: "Recommended" = an analyst's proposal awaiting the PM.
  "Leaning", "Proposed", "Suggested", "Decided" = the PM's own view or ruling.

## Never put in this repo

PHI, credentials, tokens, keys, connection strings, or customer-identifying
data beyond practice names the PM has approved for internal use.

## Related repos

- `FrontRunnerHCV2/frhc-platform` — the platform (V2), source of truth.
- `crogersfrhc/frontrunner-legacy-v1` — V1 legacy app, read-only archive.
- `crogersfrhc/app` — athena-embed prototype and athena planning state only;
  do not add other EHRs' material there.
