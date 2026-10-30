# Launch requirements

Baseline decision: D-005; assessment: [commercial-model-review.md](commercial-model-review.md).
Updated 2026-10-29. **No commercial release approval has been recorded.**

These are requirements for a future implementation, not claims that controls already exist.
The repository currently has no runnable application or billing configuration.

## Release modes

| Mode | Permitted project scope | Gate |
|---|---|---|
| Research / mocked demo | Local documentation, UI prototypes, mock jobs and disconnected test billing | No public generation or live payments. |
| Personal MVP | Owner allowlist, private image/video jobs, projects/history, owner-paid API costs | Complete P checks below. |
| Customer pilot, including free access | Limited external users only after classification and safeguards | Complete C checks; paid capabilities stay off unless the written scope permits them. |
| Commercial credit hub | Customer plans/top-ups and Higgsfield-backed generation | Complete C and B checks with explicit resale approval. |

## P — private implementation

- [ ] P1 — Owner records API account/workspace eligibility and confirms accepted current terms; validate live endpoint schemas, prices and model availability (D-001/D-006).
- [ ] P2 — Enforce owner allowlist on every job/upload/status/download route; no public signup. Test denial for unauthenticated and non-owner requests.
- [ ] P3 — Keep provider keys server-side; redact prompts, secrets and private media URLs from logs (D-008/D-014). Confirm private storage access.
- [ ] P4 — Review planned inputs and selected models; record any real-person consent and protected-feature verification before enabling the relevant feature. Keep such features off while evidence is missing.
- [ ] P5 — Record privacy, retention, training settings and upstream handling; do not promise no-training or zero retention without evidence. Establish output persistence and deletion policy.
- [ ] P6 — Track actual provider spend as an owner budget, not a purchasable wallet. No Stripe live setup, subscription, top-up, paid generation or customer credit issuance.

## C — external-user release

- [ ] C1 — Owner selects the exact business/deployment model and documents its classification. For the proposed credit hub obtain prior written consent explicitly covering section 4.2, rather than relying on API access or a generic commercial-output statement. A different pilot needs its own documented scope decision.
- [ ] C2 — Record authorization evidence: legal entity, app/domain, workspace, issuing Higgsfield representative and authority, agreement/reference and date, effective/expiry dates, territories, customers, approved models/endpoints, protected features, pricing structure, obligations and termination/revocation effects. Keep private contracts outside this public repository; commit a redacted reference and scope summary.
- [ ] C3 — Product owner demonstrates substantive workflow functionality against 4.1; document use cases and acceptance criteria. Resolve questions with Higgsfield independently of resale approval.
- [ ] C4 — Review incorporated Terms of Use, DPA, current documentation, model-specific restrictions and any Order Form with appropriate legal review. Resolve template licensing (D-004), model assets and branding requirements. Do not assume the historical no-Higgsfield-branding brief overrides an agreement.
- [ ] C5 — Replace `/legal/*` placeholders. Review end-user restrictions, consent obligations, age threshold (18 or higher local majority), Higgsfield-benefiting disclaimers/liability protections, enforcement and suspension rights. Record versioned affirmative assent before access, with reacceptance when required.
- [ ] C6 — Implement age/compliance controls, moderation, reporting and prompt suspension; test prohibited-user access and abuse handling. Define the operator responsible for responding to Higgsfield.
- [ ] C7 — Maintain protected-feature matrix with verification state, applicable attestations, model/endpoint scope and consent evidence for submission plus output use/distribution. Block missing/revoked/expired evidence in server authorization, not only UI.
- [ ] C8 — Review privacy notices, subprocessors/upstream models, data minimization, storage/deletion, content rights, AI disclosures and preservation of provenance. Review public sharing and Recreate separately.
- [ ] C9 — Recheck official terms and permission immediately before release and when model, feature, pricing or deployment scope changes. Define a person responsible for periodic review; no automation is configured by this document.

## B — paid generation activation

- [ ] B1 — Explicit credit-resale scope approved under C1/C2; a submitted request, silence, provider key or subscription is not approval.
- [ ] B2 — Define customer credits as the app's accounting units, distinct from provider units. Approve margins, credit expiry/refund disclosures, customer support, chargebacks, provider outages and outstanding-balance obligations. Do not copy provider refund rules as customer policy without review.
- [ ] B3 — Build and verify an idempotent ledger and payment handling in isolated test mode, including duplicate events, failed/blocked/canceled jobs, partial completions and reconciliation. Confirm provider behavior before promising refunds (D-007/D-015/D-017).
- [ ] B4 — Owner signs off dated release evidence and enabled scope; then, and only then, configure live payments for approved models/features.

## Required enforcement design

Use explicit release mode and separate server-side capabilities for external generation,
paid billing and protected features. Proposed defaults: `APP_MODE=personal`,
`EXTERNAL_GENERATION_ENABLED=false`, `PAID_BILLING_ENABLED=false`,
`PROTECTED_FEATURES_ENABLED=false`. These names are a design proposal, not existing environment variables.

Missing or invalid authorization must fail closed. Gate checkout/session creation,
payment-triggered credit grants, subscription renewals and job submissions as well as UI.
Personal-mode routes must reject customer credit operations. Test direct requests that bypass the UI.

On consent revocation, expiry or material scope change, disable affected new generation and
paid sales/renewals, preserve accounting records, notify the owner and resolve customer
balances under the applicable agreements. Do not automatically delete evidence or silently
substitute an unreviewed model.

## Current evidence register

| Dependency | Status | Owner / next evidence |
|---|---|---|
| Public API terms | Verified 2026-10-29; displayed version 2026-09-02 | Owner rechecks before rollout |
| Section 4.2 written permission | **Unestablished in reviewed repository** | Product owner obtains scoped written response |
| Authorization request | Draft prepared, **not sent** | Owner fills identity/domain/spend details and submits |
| End-user agreement / assent | Planned legal placeholders only | Legal/product review and implementation |
| Protected-feature eligibility | Not verified | Owner/provider confirmation per feature |
| Template license | D-004 remains open | Recheck license / obtain permission |
| Runtime and billing gates | Requirements only; no app exists here | Implement and test when scaffolding starts |
