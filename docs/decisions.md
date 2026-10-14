# Decisions log

Each entry: the decision, why, and the source. Entries marked **OPEN** need an
answer from the product owner. Entries marked **DISAGREEMENT** record where the
build brief, the Higgsfield template and other sources disagree (§0 rule 3).

Reference snapshot used for every entry below (cloned into `_reference/`,
gitignored):

| Repo | Commit | Date |
|---|---|---|
| `higgsfield-ai/app-templates` | `1e15e22` | 2026-10-06 |
| `higgsfield-ai/higgsfield-js` | `e3f2742` | 2026-09-18 |
| `higgsfield-ai/higgsfield-client` | `aefd1ca` | 2026-09-17 |
| `higgsfield-ai/cli` | `dc7e2d2` | 2026-09-11 |
| `higgsfield-ai/skills` | `f83af0b` | 2026-09-26 |

---

## D-001: Phase 0 ran without the live docs or the live site (BLOCKER)

The session's network egress policy denies `docs.higgsfield.ai`,
`higgsfield.ai`, `api.higgsfield.ai`, `console.higgsfield.ai`,
`platform.higgsfield.ai` and `open.higgsfield.ai` (the proxy answers 403 to
CONNECT; WebFetch can't resolve them either). GitHub clones work.

Consequences:

- `docs/model-catalog.md` is built only from the template source, the CLI's
  `MODELS.md` and the skills repo. No model is marked **Live** from the docs.
  Template REST paths are marked "path from template" and their parameters are
  marked unverified where the template uses generic defaults (see D-006).
- `docs/ui-inventory.md` covers the template components (Higgsfield's own
  studio UI) and the page structure in the brief. The live-site capture
  (screenshots at 1440px/390px, spacing, copy states) **has not been done**,
  and `docs/screens/` is empty. It is not faked.
- No request has been sent to the Higgsfield API.

To unblock: allow those hosts in the environment's network settings (or pick a
broader access level). Later phases also need `api.resend.com`,
`api.stripe.com`, `<account>.r2.cloudflarestorage.com`, the Higgsfield upload
and CDN hosts, and `ui.shadcn.com` (scaffold). See D-012 for MongoDB.

## D-002: API base URL is `https://api.higgsfield.ai` (DISAGREEMENT, resolved)

The brief says the template uses `https://platform.higgsfield.ai`. At commit
`1e15e22` it no longer does: `.env.example`, `README.md` and the shadcn init
all use `https://api.higgsfield.ai`. That matches what the brief says the docs
use. We use `HF_API_BASE_URL=https://api.higgsfield.ai`. Recheck against the
live docs when reachable.

## D-003: Where API keys come from (DISAGREEMENT, open until docs reachable)

The template's copy and validation errors point users to
`https://open.higgsfield.ai/api-keys` (`generation/credentials.ts`,
`README.md`). The brief says `console.higgsfield.ai`. We only need one
platform key, created by the owner, so this only affects `README.md` and the
runbook. Use whichever the live docs name, and note it there.

## D-004: Template license (OPEN before launch)

`app-templates` still has **no LICENSE file** at `1e15e22` (2026-10-06). The
repo is published as a public shadcn registry intended for scaffolding apps on
the Higgsfield API, which is our use. Keep the template's attribution
comments, re-check for a LICENSE before launch, and get written confirmation
from Higgsfield if none appears. The SDK, CLI and skills repos carry their own
licenses (see `docs/model-catalog.md` sources).

## D-005: Higgsfield API terms for resale (OPEN)

The business resells Higgsfield generations for credits at a markup. Before
launch, confirm the Higgsfield API terms allow reselling generations to end
users, and what attribution or content-policy pass-through they require. This
was not checkable (D-001).

## D-006: Template model parameters are often generic, not per-model

`generation/catalog/defaults.ts` gives every `imageModel(...)` the same
settings (aspect `auto/1:1/4:3/3:4/16:9/9:16`, resolution `1k/2k/4k`, up to 8
reference images) and every `videoModel(...)` the same settings (aspect
`16:9/9:16/1:1`, resolution `720p/1080p`, duration 4–10s). Only Soul, Seedance
and Kling 3 have hand-written settings and mappers. So for most template
models the **path** comes from Higgsfield's source, but the **parameter names,
enums and ranges** do not reflect the model's real schema. Phase 4 must check
each against its docs page before marking it Live.

## D-007: Template mapper defects to fix in Phase 4

Found by running every template mapper with sample inputs:

- **LTX 2.5 Fast / Pro:** `t2v("lightricks/ltx-2.5/text-to-video/fast")`
  doesn't end in `/text-to-video`, so no image path is derived. The model
  offers a start-frame slot, but the frame is **silently dropped** and the
  request goes to text-to-video.
- **DoP, Kling 2.5:** only an image-to-video path exists, but no
  `requiredRoles: ["start"]` is set, so a prompt-only submit goes to the
  image path with no image.
- **Kling O1, Kling O3:** only a first-last-frame path; prompt-only submits go
  there with no frames. Needs `requiredRoles`.
- **Grok Imagine Video 1.5:** reference-only path; prompt-only submit sends no
  references. Check whether the endpoint allows that.
- **All generic image models:** reference images are sent as `image_urls` to
  the text path. Unverified per model.
- **Cancel:** `useRuns.cancel` marks the run `failed` with "Canceled" as soon as
  the cancel call returns, without reading the platform status. We will finalize
  from the platform's `canceled` status instead (and refund there).

## D-008: Template logs full request and response bodies

`generation/platform.ts` `console.info`s every request body and every response
payload (except upload tickets). Prompts are user content and output URLs are
unguessable links. Our server client will log method, path, status and
`request_id` only.

## D-009: Scaffold into the repo root, from the pinned reference commit

`shadcn init -n <slug>` creates a sub-folder; this repository *is* the app, so
the scaffold is moved to the repo root. `ui.shadcn.com` is also blocked (D-001).
If `shadcn init` can't run, we copy the `studio` registry item's files from
`_reference/app-templates` at `1e15e22`, which is the same content the
registry serves.

## D-010: Placeholders in the brief are unfilled (OPEN)

`BRAND_NAME`, `APP_SLUG`, `PRODUCTION_URL`, `EMAIL_FROM`, `SUPPORT_EMAIL` and
`CREDIT_MARKUP` were not filled in. Phase 0 doesn't need them. Phase 1 needs
at least `BRAND_NAME` and `APP_SLUG`.

## D-011: Placeholder Phase 0 decisions that stand until contradicted

- Package manager: pnpm (the template's lockfile and scripts).
- Node: 22.x (the template's tests need 22.15+; this container has 22.22).
- Dark only; no theme toggle (template rule).

## D-012: MongoDB from the cloud container (OPEN for dev/test)

The egress proxy doesn't carry raw TCP, so this container can't reach
MongoDB Atlas. Production (Vercel → Atlas) is unaffected. For local dev and
tests here we need a local `mongod` started as a single-node replica set
(transactions need a replica set). Ubuntu 24.04 has no `mongodb` package,
`repo.mongodb.org` and `fastdl.mongodb.org` (used by `mongodb-memory-server`)
are blocked, but Docker is installed and Docker Hub answers. Default: run the
official `mongo` image with `--replSet rs0` for dev and integration tests.
Confirm in Phase 1.

## D-013: Call the REST API with our own thin `fetch` client, not `@higgsfield/client` v2

Choice recorded per brief §1a. Reasons, from the SDK source at `e3f2742`:

- v2 has **no cancel method and no generation upload method**
  (`higgsfield-js/src/v2/types.ts:91`, only `cancel_url` is exposed). We need both.
- v2 polling treats only `completed | nsfw | failed` as terminal
  (`src/v2/client.ts:206-210`); `canceled` is missing from its status type
  (`src/v2/types.ts:77`), so a canceled run is polled until timeout.
- v2 wraps the **billable submit POST** in retries with no idempotency key, and
  because its response interceptor converts HTTP errors to `APIError` before the
  retry check, it effectively retries 4xx as well (`src/utils/retry.ts:27-37`,
  `src/v2/client.ts:166-178`). That risks duplicate paid jobs.

The template's client (`generation/platform.ts`) already does submit / status /
cancel / upload-URL with the right shapes. We port it to `lib/higgsfield/`
(server-only), drop body logging (D-008), and add our own retry policy (D-015).

## D-014: Credentials are one `key_id:key_secret` string in `HF_CREDENTIALS`

Both SDKs send `Authorization: Key <key_id>:<key_secret>`
(`higgsfield-js/src/v2/client.ts:119`, `higgsfield_client/http/client.py:179`).
The JS SDK reads `HF_CREDENTIALS` (then `HF_KEY`, then
`HF_API_KEY`+`HF_API_SECRET`, `src/auth.ts:10-23`). The template treats the key
as one opaque string and sends `Key <string>`, which is the same header when the
string is `id:secret`. We keep `HF_CREDENTIALS=key_id:key_secret` (brief §11)
and validate at boot that it has exactly one `:` and only printable ASCII.
The credentials dashboard is named as `cloud.higgsfield.ai` in the Python SDK
README, `open.higgsfield.ai` in the template, `console.higgsfield.ai` in the
brief (see D-003).

## D-015: Submit retry policy (our own; SDKs disagree)

- The SDKs disagree (JS: ~all HTTP errors, 4 attempts, jitter; PY: 408/429/5xx,
  3 attempts, no jitter). Neither sends an idempotency key, and the docs could
  not be checked for one (D-001).
- Ours: retry the submit POST **only** on network errors that happened before
  any response, and on **429** (honouring `Retry-After` if present), with
  exponential backoff plus full jitter, at most 3 attempts. **Do not
  auto-retry 5xx or timeouts on submit**: the request may have been accepted, so
  the generation stays `pending_submit` and the reconciler resolves it (brief §7
  step 4). Status/cancel GETs are idempotent and are retried on 429/5xx.
- If the live docs describe an idempotency header, add it and widen retries.

## D-016: Webhook authenticity: our own HMAC token; never trust the body

Neither SDK has webhook signature code. JS v2 accepts `webhook.secret` and
discards it; a JS helper comment mentions an `X-Webhook-Secret-Key` header that
no code sends (`src/helpers.ts:177`). So, as the brief says: the webhook URL
carries `gid` + `t=HMAC_SHA256(gid, HF_WEBHOOK_SECRET)` (constant-time compare),
and the handler re-fetches `GET /requests/{id}/status` before acting. If a
signature or secret header is documented later, verify it too. The attach
format is `?hf_webhook=<encodeURIComponent(url)>`, using `&` if the path already
has a query (`src/v2/client.ts:269-273`).

## D-017: Status handling

- Statuses: `queued`, `in_progress`, `completed`, `failed`, `nsfw`,
  `canceled` (one "l"; `higgsfield_client/types_.py:71-77`). Terminal:
  `completed`, `failed`, `nsfw`, `canceled`.
- Outputs: the SDKs only define `images: [{url}]` and `video: {url}`. Audio and
  other artifact shapes are undefined in every source we have, so the status
  parser must accept unknown output keys without crashing, and the audio surface
  stays blocked until a docs page defines its output shape.
- Error bodies are FastAPI-style: `detail` is a string or an array of
  `{loc, msg, type}` (`src/v2/client.ts:155-165`). Out of credits on the
  generation API is **403** (`src/v2/client.ts:169-170`), which for us is an
  operator alert (our platform balance), never shown to the user as their
  problem.
- Cancel: `POST /requests/{id}/cancel`; 400 once processing has started
  (`higgsfield_client/http/client.py:298-302`).

## D-018: Upload PUT must send the returned `upload_headers`

The PY SDK sends `upload_headers` (its fixture includes
`x-amz-tagging: retention=temporary`, so storage is S3-like and uploads may be
short-lived); JS v1 ignores them. The template sends exactly `upload_headers`
with `credentials: "omit"` and rejects tickets whose headers include
`authorization`/`cookie` or a mismatched content type. We keep the template's
behavior. Because reference uploads may be tagged temporary, the `uploads`
library tab should copy reference files to R2 too if we want them reusable
after Higgsfield's retention window. Decide in Phase 3.

## D-019: Which models appear in the picker (DISAGREEMENT, resolved)

The brief (§0 rule 2) says unverified models stay out of the picker. The
template (`AGENTS.md`) says to keep every installed model visible even when its
live behavior is unverified. The brief also calls template REST paths
"verified from its source" (§4). Resolution: the 38 template entries
(**Path verified**) stay in the picker; anything without a REST path from the
template or the live docs (**Unverified**: Veo, Nano Banana, GPT Image, audio…)
stays out until its docs page is read. Phase 4 must still check every template
entry's parameters against its docs page, because many use generic defaults
(D-006) and some are invalid (e.g. Hailuo's default duration 5; the CLI allows
only 6 or 10).
