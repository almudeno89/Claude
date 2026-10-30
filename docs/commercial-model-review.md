# Higgsfield commercial model review

Reviewed: 2026-10-29 (America/Los_Angeles). Status: **commercial generation resale blocked pending documented authorization**.

## Evidence and scope

- Official [API Terms of Service](https://open.higgsfield.ai/terms-of-service), retrieved on the review date; displayed last update: **September 2, 2026**. Section references below refer to that version.
- Project baseline: `5e3d8ac0bdf5d375eae5b6b6867f37d52cc87ed8` (October 14 research), especially D-005, D-014, D-019/D-020, UI inventory A1–A5, and the model catalog.
- All tracked files in that baseline were reviewed. It contains documentation, not an implemented application. No written resale authorization was found there. Private agreements, workspace verification and account settings were not inspected. **Authorization is unestablished, not proven absent.**
- This is a project implementation assessment, not a complete review of the incorporated Terms of Use, DPA, endpoint documentation or a private Order Form. Review those before release.

## Verified contractual points

Section 4.1 requires material application functionality and bars direct endpoint/key exposure. Section 4.2 separately requires prior written consent for the covered resale, competing generation, service-bureau and outsourcing models. Section 4.4 requires enforceable protective end-user terms; 4.5–4.6 address enforcement and age/compliance measures. Sections 2.2 and 6.3 gate protected features through verification/attestations; 6.2 requires documented likeness consent covering submission and intended distribution. Section 12.1 permits commercial output use; that does not authorize service resale. Sections 1.2 and 9 separate API funding from consumer subscriptions. Section 2.1 keeps keys server-side. Sections 5 and 7 require attention to upstream models and data handling; 12.3 addresses disclosure and provenance. Section 1.4 permits an applicable Order Form to control conflicts.

## Planned-feature assessment

The classifications and controls below are **project judgments**, not Higgsfield approval. "Conditional" means buildable only within an otherwise authorized use; it is not permission for the credit business.

| Planned feature | Classification | Project disposition |
|---|---|---|
| Owner-only image/video generator, private projects, asset history, downloads | Conditional personal/internal path | Proceed with private development; validate endpoints and account eligibility before live use. |
| Public model picker, presets, prompt/settings controls and a generation feed | Restricted in proposed resale context | Keep customer generation disabled pending commercial gate. A polished wrapper is not evidence of material functionality or resale consent. |
| Marked-up generation credits, top-ups, monthly/annual plans including generations, upgrade/checkout/portal | Requires prior written authorization for proposed model | Do not activate or accept money for Higgsfield-backed generation. Pricing research and disconnected mockups may continue. |
| Platform-funded free trials or free public generation | Requires scope clarification | No public beta until reviewed; zero price alone does not settle competing-service or service-bureau classification. |
| Project organization, preset workflows, asset management, later editing/review/export tools | Conditional; potential material functionality | Define measurable independent workflow value. Do not infer an exception to 4.2 from these features. |
| Faces, reference likenesses, avatars, voice/lip-sync, Soul ID/identity consistency | Additional verification and consent dependencies | Default off; maintain a feature-specific eligibility record. Fictional people and a portrait model are not automatically equivalent to real-person identity features; assess inputs and endpoints. |
| Community publishing, Recreate, Report | Conditional with added distribution risks | Defer for personal MVP. Before launch, implement rights/consent checks, moderation, reporting and removal. |
| End-user signup and legal pages | Required before admitting end users | Replace legal placeholders with reviewed agreements and recorded assent; a footer link alone is not the acceptance design. |
| Raw proxy, exposed platform credentials, standalone API access | Restricted | Exclude from scope. |
| Client deliverables produced using the hub | Commercial output use differs from selling generation access | Review each service scope. Do not market an outsourced generation bureau as an automatic output-sale exception. |

## Practical paths

| Option | Commercial mechanism and implementation | Decision / evidence needed |
|---|---|---|
| Personal/internal MVP now | One owner login; private library; actual provider-cost tracking; no customer wallets, plans, checkout, public signup or public generation. | Recommended interim scope. Keep work for customers and third-party operation out until classified. This does not waive API/content obligations. |
| Authorized credit platform | Operator buys API usage; customers buy the app's own credits at a markup; credits debit on generation; subscriptions may bundle credits. | Closest to original business. Obtain explicit written 4.2 consent and model/feature scope; evaluate margins after contracted terms, storage, moderation and support costs. |
| Customer-owned deployment / BYOK software | Customer contracts with and pays provider directly; charge for installation, workflow software, storage, editing or support. Keep customer credentials in their server environment. | Potential software revenue, not preapproved. Seek written classification of hosted versus customer-operated deployments and multi-tenant use; do not disguise generation markup as a software fee. |
| Workflow product with generation outside the app | Sell brief/storyboard tools, prompt organization, editing, asset review and export; users import media generated under their own provider arrangements. | Strongest separation for near-term software sales. No Higgsfield API generation traffic or credentials in this product; verify asset, software and branding rights separately. |
| Other licensed provider or self-hosted models | Preserve provider adapter boundary; choose image/video providers or model licenses that explicitly cover intended customer usage and resale. | Viable candidate, not a verified vendor recommendation. Record written rights, model licenses, output/data terms and full operating costs before selection. |
| Human-led creative service | Sell a finished creative project with meaningful human work. | Scope-sensitive: outsourcing/service-bureau language makes this unsuitable as an assumed workaround. Obtain written clarification for the actual workflow. |

Renaming credits, bundling generation into subscriptions, removing markup, adding editing, or using customer keys does not by itself resolve the applicable restrictions.

## Next decision

Adopt the personal MVP while seeking the permission described in
[higgsfield-authorization-request.md](higgsfield-authorization-request.md).
Follow [launch-requirements.md](launch-requirements.md) before any customer rollout.
If permission is declined, use the generation-external workflow product or a separately licensed provider; reopen the architecture and business case before implementing customer billing.
