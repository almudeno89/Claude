# Model catalog

Phase 0 inventory of every model the Higgsfield API may expose, and how sure we
are of each one. Built **without** the live docs or the console (both blocked;
`docs/decisions.md` D-001), so nothing here is **Live** yet.

Sources (snapshot commits in `docs/decisions.md`):

- **T**: `higgsfield-ai/app-templates`, `generation/catalog/models/*.ts`. The
  only source of REST paths we treat as verified (brief §4). Paths below were
  produced by running every template mapper with sample inputs, not by reading
  the files by eye.
- **M**: `higgsfield-ai/cli`, `MODELS.md` (58 models with parameter tables) and
  `README.md`. CLI names (`nano_banana_2`, `seedance_2_0`) are **job-set
  types** from a different API. They are not REST paths, and the CLI parameter
  *names* may differ from REST body fields (e.g. Soul: CLI `quality` 1.5k/2k vs
  REST `resolution` 720p/1080p). Use **M** for which models exist and their
  ranges, never for request shapes.
- **S**: `higgsfield-ai/skills`, `higgsfield-generate/references/*.md`. "Best
  for" guidance, and a few modes/ranges M omits.
- **SDK**: `higgsfield-js` / `higgsfield-client` example endpoints.

Not yet consulted (blocked): `docs.higgsfield.ai/docs/models.md` and every
model page, `/docs/openapi.json`, the console model list and price list.

## Status legend

| Status | Meaning | In the picker? |
|---|---|---|
| **Live** | REST path and request schema checked against the model's docs page, and one real generation succeeded. | Yes |
| **Path verified** | REST path comes from Higgsfield's own template source. Parameters not yet checked against docs. | Yes from Phase 1 (brief §4 counts these as verified; the template's own rule says to keep them visible and report the limitation). Phase 4 must check each against its docs page before calling it Live. |
| **Unverified** | Higgsfield ships it (CLI, skills or site), but we have no confirmed REST path. | **No** (brief §0 rule 2) |
| **Not in API** | Docs or console confirm it is not in the public REST API. | No |

No model can be moved to **Not in API** without the docs. Rows we *suspect* are
app-only or agent-only are flagged "suspected app-only" under Unverified.

Price source for every row: **console price list, not yet read** (blocked).
`lib/billing/price.ts` cannot be filled in until the console prices are
available (or you send me a copy).

## Counts

| | Image | Video | Audio | 3D | Utility / other | Total |
|---|---|---|---|---|---|---|
| Path verified (template) | 8 | 30 | 0 | 0 | 0 | **38** entries (28 files) |
| Unverified (CLI/skills only) | 20 | 18 | 6 | 5 | 4 (+12 named with no schema) | 53 (+12) |
| Live | 0 | 0 | 0 | 0 | 0 | **0** |

---

## 1. Path verified: the template catalog (38 entries)

Generated from the template at `1e15e22`. "media roles" are the template's
upload slots and max counts; "settings" are the template's UI settings (the
mapper turns them into body fields).

| # | id | surface | label | file | REST path(s) | media roles | settings (template) | mapper |
|---|---|---|---|---|---|---|---|---|
| 1 | `soul-2` | image | Soul 2 | `soul.ts` | `higgsfield-ai/soul/v2/standard` | none | `aspectRatio` {9:16, 16:9, 4:3, 3:4, 1:1, 2:3, 3:2} =1:1; `resolution` {720p, 1080p} =720p; `batchSize` {1, 4} =1; `enhancePrompt` bool =false | custom |
| 2 | `soul-cinema` | image | Soul Cinema | `soul.ts` | `higgsfield-ai/soul/cinema` | none | `aspectRatio` {9:16, 16:9, 4:3, 3:4, 1:1, 2:3, 3:2} =1:1; `resolution` {720p, 1080p} =720p; `batchSize` {1, 4} =1; `enhancePrompt` bool =false | custom |
| 3 | `seedance-2.5` | video | Seedance 2.5 | `seedance-2.5.ts` | `bytedance/seedance-2.5/image-to-video`<br>`bytedance/seedance-2.5/reference-to-video`<br>`bytedance/seedance-2.5/text-to-video` | start×1, end×1, reference×30, video×10, audio×10 | `aspectRatio` {16:9, 4:3, 1:1, 3:4, 9:16, 21:9} =16:9; `duration` 4–30 =5; `resolution` {480p, 720p} =720p; `generateAudio` bool =true; `bitrateMode` {standard, high} =high | custom |
| 4 | `seedance-2.5-edit` | video | Seedance 2.5 Edit | `seedance-2.5.ts` | `bytedance/seedance-2.5/video-edit` | source×1, video×10, reference×30, audio×10 (req: source) | `resolution` {480p, 720p} =720p; `generateAudio` bool =true; `bitrateMode` {standard, high} =high | custom |
| 5 | `seedance-2.5-extend` | video | Seedance 2.5 Extend | `seedance-2.5.ts` | `bytedance/seedance-2.5/video-extend` | source×1, video×10, reference×30, audio×10 (req: source) | `duration` 4–30 =5; `resolution` {480p, 720p} =720p; `generateAudio` bool =true; `bitrateMode` {standard, high} =high | custom |
| 6 | `seedance-2` | video | Seedance 2.0 | `seedance-2.ts` | `bytedance/seedance-2.0/image-to-video`<br>`bytedance/seedance-2.0/reference-to-video`<br>`bytedance/seedance-2.0/text-to-video` | start×1, end×1, reference×9, video×3, audio×3 | `aspectRatio` {16:9, 4:3, 1:1, 3:4, 9:16, 21:9} =16:9; `duration` 4–15 =5; `generateAudio` bool =true; `resolution` {480p, 720p, 1080p, 4k} =720p | custom |
| 7 | `seedance-2-fast` | video | Seedance 2.0 Fast | `seedance-2.ts` | `bytedance/seedance-2.0/fast/image-to-video`<br>`bytedance/seedance-2.0/fast/reference-to-video`<br>`bytedance/seedance-2.0/fast/text-to-video` | start×1, end×1, reference×9, video×3, audio×3 | `aspectRatio` {16:9, 4:3, 1:1, 3:4, 9:16, 21:9} =16:9; `duration` 4–15 =5; `generateAudio` bool =true; `resolution` {480p, 720p} =720p | custom |
| 8 | `seedance-2-mini` | video | Seedance 2.0 Mini | `seedance-2.ts` | `bytedance/seedance-2.0/mini/image-to-video`<br>`bytedance/seedance-2.0/mini/reference-to-video`<br>`bytedance/seedance-2.0/mini/text-to-video` | start×1, end×1, reference×9, video×3, audio×3 | `aspectRatio` {16:9, 4:3, 1:1, 3:4, 9:16, 21:9} =16:9; `duration` 4–15 =5; `generateAudio` bool =true; `resolution` {480p, 720p} =720p | custom |
| 9 | `kling-3-turbo` | video | Kling 3.0 Turbo | `kling-3.ts` | `kling-video/v3.0-turbo/image-to-video`<br>`kling-video/v3.0-turbo/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 3–15 =5 | custom |
| 10 | `kling-3-std` | video | Kling 3.0 Standard | `kling-3.ts` | `kling-video/v3.0/std/image-to-video`<br>`kling-video/v3.0/std/text-to-video` | start×1, end×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `duration` 3–15 =5; `sound` bool =true; `cfgScale` 0–1 =0.5; `multiShots` bool =false | custom |
| 11 | `kling-3-pro` | video | Kling 3.0 Pro | `kling-3.ts` | `kling-video/v3.0/pro/image-to-video`<br>`kling-video/v3.0/pro/text-to-video` | start×1, end×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `duration` 3–15 =5; `sound` bool =true; `cfgScale` 0–1 =0.5; `multiShots` bool =false | custom |
| 12 | `kling-3-4k` | video | Kling 3.0 4K | `kling-3.ts` | `kling-video/v3.0/4k/image-to-video`<br>`kling-video/v3.0/4k/text-to-video` | start×1, end×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `duration` 3–15 =5; `sound` bool =true; `cfgScale` 0–1 =0.5; `multiShots` bool =false | custom |
| 13 | `kling-3-motion-std` | video | Kling 3.0 Motion Control | `kling-3.ts` | `kling-video/v3/motion-control/std` | start×1, video×1 | `keepOriginalSound` bool =true; `characterOrientation` {video, image} =video | custom |
| 14 | `kling-3-motion-pro` | video | Kling 3.0 Motion Control Pro | `kling-3.ts` | `kling-video/v3/motion-control/pro` | start×1, video×1 | `keepOriginalSound` bool =true; `characterOrientation` {video, image} =video | custom |
| 15 | `flux-2` | image | Flux 2 | `flux-2.ts` | `flux-2-pro` | reference×8 | `aspectRatio` {auto, 1:1, 4:3, 3:4, 16:9, 9:16} =1:1; `resolution` {1k, 2k, 4k} =1k | paths |
| 16 | `grok-imagine-2` | image | Grok Imagine 2.0 | `grok-imagine-2.ts` | `xai/grok-imagine-image-2.0` | reference×8 | `aspectRatio` {auto, 1:1, 4:3, 3:4, 16:9, 9:16} =1:1; `resolution` {1k, 2k, 4k} =1k | paths |
| 17 | `ideogram-4` | image | Ideogram 4.0 | `ideogram-4.ts` | `ideogram/v4.0` | reference×8 | `aspectRatio` {auto, 1:1, 4:3, 3:4, 16:9, 9:16} =1:1; `resolution` {1k, 2k, 4k} =1k | paths |
| 18 | `recraft-4.1` | image | Recraft 4.1 | `recraft-4.1.ts` | `recraft/v4.1/text-to-image` | reference×8 | `aspectRatio` {auto, 1:1, 4:3, 3:4, 16:9, 9:16} =1:1; `resolution` {1k, 2k, 4k} =1k | paths |
| 19 | `qwen-image-3` | image | Qwen Image 3 | `qwen-image-3.ts` | `alibaba/qwen-image-3/text-to-image` | reference×8 | `aspectRatio` {auto, 1:1, 4:3, 3:4, 16:9, 9:16} =1:1; `resolution` {1k, 2k, 4k} =1k | paths |
| 20 | `z-image-turbo` | image | Z-Image Turbo | `z-image-turbo.ts` | `z-image/turbo` | reference×8 | `aspectRatio` {auto, 1:1, 4:3, 3:4, 16:9, 9:16} =1:1; `resolution` {1k, 2k, 4k} =1k | paths |
| 21 | `wan-3` | video | Wan 3.0 | `wan-3.ts` | `alibaba/wan-3.0/image-to-video`<br>`alibaba/wan-3.0/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 22 | `wan-3-prime` | video | Wan 3.0 Prime | `wan-3-prime.ts` | `alibaba/wan-3.0-prime/image-to-video`<br>`alibaba/wan-3.0-prime/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 23 | `wan-2.7` | video | Wan 2.7 | `wan-2.7.ts` | `wan/v2.7/image-to-video`<br>`wan/v2.7/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 24 | `wan-2.6` | video | Wan 2.6 | `wan-2.6.ts` | `wan/v2.6/image-to-video`<br>`wan/v2.6/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 25 | `flux-3` | video | Flux 3 | `flux-3.ts` | `blackforestlabs/flux-3/image-to-video`<br>`blackforestlabs/flux-3/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 26 | `minimax-h3` | video | MiniMax H3 | `minimax-h3.ts` | `minimax/h3/image-to-video`<br>`minimax/h3/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 27 | `minimax-hailuo-2.3` | video | MiniMax Hailuo 2.3 | `minimax-hailuo-2.3.ts` | `minimax/hailuo-2.3/standard/image-to-video`<br>`minimax/hailuo-2.3/standard/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 28 | `happy-horse-1` | video | Happy Horse 1.0 | `happy-horse-1.ts` | `alibaba/happy-horse/image-to-video`<br>`alibaba/happy-horse/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 29 | `happy-horse-1.1` | video | Happy Horse 1.1 | `happy-horse-1.1.ts` | `alibaba/happy-horse/v1.1/image-to-video`<br>`alibaba/happy-horse/v1.1/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 30 | `kling-2.6` | video | Kling 2.6 | `kling-2.6.ts` | `kling-video/v2.6/pro/image-to-video`<br>`kling-video/v2.6/pro/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 31 | `kling-2.5` | video | Kling 2.5 | `kling-2.5.ts` | `kling-video/v2.5-turbo/standard/image-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 32 | `kling-o3` | video | Kling O3 | `kling-o3.ts` | `kling-video/o3/first-last-frame` | start×1, end×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 33 | `kling-o1` | video | Kling O1 (Omni) | `kling-o1.ts` | `kling-video/omni/first-last-frame` | start×1, end×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 34 | `ltx-2.5-fast` | video | LTX 2.5 Fast | `ltx-2.5-fast.ts` | `lightricks/ltx-2.5/text-to-video/fast` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 35 | `ltx-2.5-pro` | video | LTX 2.5 Pro | `ltx-2.5-pro.ts` | `lightricks/ltx-2.5/text-to-video/pro` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 36 | `grok-imagine-video-1.5` | video | Grok Imagine Video 1.5 | `grok-imagine-video-1.5.ts` | `xai/grok-imagine-video/v1.5/reference-to-video` | reference×8, video×3 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 37 | `pixverse-6` | video | PixVerse 6 | `pixverse-6.ts` | `pixverse/v6/image-to-video`<br>`pixverse/v6/text-to-video` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |
| 38 | `dop` | video | DoP | `dop.ts` | `higgsfield-ai/dop/lite` | start×1 | `aspectRatio` {16:9, 9:16, 1:1} =16:9; `resolution` {720p, 1080p} =720p; `duration` 4–10 =5 | paths |

### 1.1 Status, CLI counterpart and known problems per template entry

Every row is **Path verified**; none is **Live**. "CLI" is the matching
`MODELS.md` job type (M) or skills/README id (S); "—" means Higgsfield's CLI
and skills don't mention it at all.

| id | CLI | Known problems (fix in Phase 4 against the docs page) |
|---|---|---|
| `soul-2` | `text2image_soul_v2` (M:316) | Template default 720p; CLI default is the higher tier (2k ≈ 1080p, S SKILL.md:161). No reference image, Soul ID or style slots (CLI allows 1 ref + `--soul-id` + `style_id`). |
| `soul-cinema` | `soul_cinematic` (M:295) | 21:9 missing; default 720p vs CLI 2k; no ref / Soul ID. `enhancePrompt`/`batchSize` aren't in the CLI schema for this model. |
| `seedance-2.5` | `seedance_2_5` (S only; **no MODELS.md section**) | Template allows 480p/720p; skills and README examples use up to **1080p**. `bitrateMode` defaults to `high` (undocumented for 2.5). Template forbids mixing frames with references; the skill sends frames through `omni_reference` together with refs. Media caps 30/10/10 unverified. |
| `seedance-2.5-edit` | `seedance_2_5` mode `video_edit` (S) | Caps unverified. |
| `seedance-2.5-extend` | `seedance_2_5` mode `video_extension` (S) | Template path says `video-extend`, CLI mode says `extension`. Caps unverified. |
| `seedance-2` | `seedance_2_0` (M:595) | No `auto` aspect, no `bitrate_mode` (mapper supports it), no `genre`. CLI caps ≤12 files total and needs an image/video beside audio refs; template allows 15 and audio-only. |
| `seedance-2-fast` | `seedance_2_0` mode `fast` (M:595) | As `seedance-2`. |
| `seedance-2-mini` | `seedance_2_0_mini` (M:622) | As `seedance-2`. |
| `kling-3-turbo` | `kling3_0_turbo` (M:515) | Matches CLI. Drops `aspect_ratio` when a start frame is set (may be intended). |
| `kling-3-std` / `-pro` / `-4k` | `kling3_0` modes std/pro/4k (M:503) | Sends `cfg_scale` and `multi_shots`, which aren't in the CLI's `kling3_0` schema. |
| `kling-3-motion-std` / `-pro` | — (S: "motion transfer", model-catalog.md:51) | No CLI schema to compare. |
| `flux-2` | `flux_2` variant `pro` (M:28) | Generic defaults: offers `auto` aspect and `4k`; CLI has neither. 8-ref cap unverified. Flex/Max variants have no path. |
| `grok-imagine-2` | `grok_image_2_0` (S, unprofiled); maybe `grok_image` (M:87) | Generic defaults. If it is `grok_image`: missing 1:2, 2:1, 3:2, 2:3 and `mode`; adds 4k. |
| `ideogram-4` | — | Generic defaults; nothing to check against. |
| `recraft-4.1` | `recraft_v4_1` (M:243) | CLI is **prompt-only**; template allows 8 refs. Missing 4:5, 5:4, 3:2, 2:3, `model_type` (vector etc.), `colors`, `background_color`; adds `auto` and 4k. |
| `qwen-image-3` | — | Generic defaults. |
| `z-image-turbo` | `z_image` (M:330) | CLI is prompt-only and has **no resolution**; template allows 8 refs and sends 1k/2k/4k. |
| `wan-3`, `wan-3-prime` | `wan3_0`, `wan3_0_prime` (S, unprofiled) | Generic defaults. |
| `wan-2.7` | `wan2_7` (M:714) | Missing 4:3/3:4, end frame, 1 audio ref. |
| `wan-2.6` | `wan2_6` (M:698) | Duration must be 5/10/15; field is `quality`, not `resolution`; CLI takes image/video/audio **references, no start frame**; template takes a start frame. |
| `flux-3` | `flux_3_video` (S, unprofiled) | Generic defaults. |
| `minimax-h3` | `minimax_h3` (S, unprofiled) | Generic defaults. |
| `minimax-hailuo-2.3` | `minimax_hailuo` variant `minimax-2.3` (M:560) | **Template default duration 5 is invalid** (6 or 10). Resolution values are `512/768/1080` (default 768), not `720p/1080p`; no aspect ratio; 1080 not allowed at 10 s. |
| `happy-horse-1`, `happy-horse-1.1` | `happy_horse_video` (S, version not stated) | Generic defaults. |
| `kling-2.6` | `kling2_6` (M:493) | Duration must be 5 or 10; no resolution field; missing `sound` (default on). |
| `kling-2.5` | — | Image-to-video path only, but the start frame isn't required (see §1.2). |
| `kling-o3`, `kling-o1` | — (CLI only has the image model `kling_omni_image`) | First/last-frame path only, frames not required (§1.2). |
| `ltx-2.5-fast`, `ltx-2.5-pro` | — | **Start frame silently dropped** (§1.2). |
| `grok-imagine-video-1.5` | `grok_video_v15` (M:478) | CLI **requires one start image**; template has no start slot, only 8 image + 3 video references. CLI: 480p/720p, no aspect; template: 720p/1080p with aspect. Skill says 2–15 s. |
| `pixverse-6` | — | Generic defaults. |
| `dop` | — (legacy SDK v1 `/v1/image2video/dop`, `dop-lite/turbo/standard`) | Image path only, start frame not required (§1.2). |

### 1.2 Template mapper defects (reproduced by running the mappers)

- **LTX 2.5 Fast/Pro drop the start frame.** `t2v()` only derives an
  `image-to-video` path when the text path ends in `/text-to-video`; LTX paths
  end in `/text-to-video/fast|pro`. The UI offers a start-frame slot, and the
  frame never reaches the API.
- **DoP, Kling 2.5, Kling O1, Kling O3** have only an image or first/last-frame
  path and no `requiredRoles`, so a prompt-only submit goes to that path with
  no image.
- **Grok Imagine Video 1.5**: prompt-only submit goes to `reference-to-video`
  with no references.
- **Generic image models** send reference images as `image_urls` on the text
  path. Unverified per model, and wrong for prompt-only models (Recraft, Z).

These are template bugs, not API facts. Phase 4 fixes each model file against
its docs page and adds a snapshot test of its `toPlatform` output.

---

## 2. Unverified: Higgsfield ships these, we have no REST path

Left out of the picker until a docs page gives the path and schema. Parameter
summaries are from `MODELS.md` (job-set names; ranges only). "REST hint" is a
path-shaped string found in an SDK, never used without docs confirmation.

### 2.1 Image

| CLI id | Name | Inputs | Key params (M) | REST hint | Notes |
|---|---|---|---|---|---|
| `gpt_image_2_5` | GPT Image 2.5 | ≤16 refs | aspect (15 values incl. auto) =1:1; quality low…max =low; resolution 1k/2k/4k; background; variant flare/sunburst | — | Skills' default image model. Brief's `/ai/image?model=gpt_image_2` target. |
| `gpt_image_2` | GPT Image 2 | refs, mask | aspect (11) ; quality =high; resolution =2k; inpaint with mask | — | |
| `openai_hazel` | OpenAI Hazel | ≤16 refs | aspect 1:1/3:2/2:3/auto; quality =medium | — | |
| `nano_banana_2` | Nano Banana **Pro** | ≤14 refs | aspect (10); resolution 1k/2k/4k =2k | `nano-banana-pro` (higgsfield-js RELEASE_NOTES.md:45) | alias `nano_banana_pro` |
| `nano_banana_flash` | Nano Banana **2** | refs | aspect (10); resolution 1k/2k/4k =1k | — | |
| `nano_banana_2_lite` | Nano Banana 2 Lite | ≤14 refs | aspect auto+10; 1k; thinking MINIMAL/HIGH | — | |
| `nano_banana` | Nano Banana | ≤8 refs | aspect (10) | — | |
| `seedream_v4_5` | Seedream 4.5 | ≤14 refs | aspect (8); quality basic/high | v4 only: `bytedance/seedream/v4/text-to-image` (higgsfield-client README.md:39) | |
| `seedream_v5_lite` | Seedream 5 Lite | refs | aspect (5); quality basic/high | — | |
| `seedream_v5_pro`, `seedream_5_0_flash` | Seedream 5 Pro / Flash | ≤10 refs (Pro) | resolution 1k/1.5k/2k (Pro) | — | skills only |
| `flux_kontext` | Flux Kontext | ≤4 refs | aspect (5) | `flux-pro/kontext/max/text-to-image` (higgsfield-js README.md:34) | |
| `kling_omni_image` | Kling O1 Image | ≤10 refs | aspect (9); 1k/2k | — | |
| `grok_image` | Grok Image | refs | aspect (10); mode std/quality; 1k/2k | — | may equal template `grok-imagine-2` |
| `cinematic_studio_2_5` | Cinema Studio 2.5 image | ≤14 refs | aspect (10); 1k/2k/4k; mode; batch | — | suspected app-only |
| `marketing_studio_image` | Marketing Studio image | ≤14 refs | aspect (11); 1k/2k/4k | — | suspected app-only (needs avatar/product entities) |
| `soul_cast` | Soul Cast | none | aspect 16:9 only (required); budget | — | |
| `soul_location` | Soul Location | none | aspect (9) | — | |
| `soul_cinema_studio` | Soul Cinema Studio | — | style_id, enhance_prompt, +21:9 | — | skills only |
| `image_auto` | Image Auto | ≤14 refs | aspect (5) | — | router; probably skip |

### 2.2 Video

| CLI id | Name | Inputs | Key params (M) | Notes |
|---|---|---|---|---|
| `veo3_1` | Google Veo 3.1 | start ×1 | aspect 16:9/9:16; duration 4/6/8 =8; quality basic/high/ultra; variant preview/fast | |
| `veo3_1_lite` | Google Veo 3.1 Lite | start, end | aspect 16:9/9:16/auto; 4/6/8; generate_audio | 8 s required with start+end |
| `veo3` | Google Veo 3 | start ×1 **required** | aspect 16:9/9:16; variant preview/fast | |
| `gemini_omni` | Gemini Omni Flash | ≤7 refs, ≤1 video | aspect 16:9/9:16; 4/6/8/10 =8; 720p | ≤5 images with a video ref. `gemini_omni_flash_1_1` in skills only |
| `seedance1_5` | Seedance 1.5 Pro | start, end | aspect (7); 4/8/12; 480p–1080p; generate_audio | |
| `grok_video` | Grok Video | start ×1 | aspect (3); duration | |
| `minimax_hailuo` variants `minimax`, `minimax-fast`, `minimax-2.3-fast` | Hailuo variants | start/end | see M:560 | only `minimax-2.3` standard has a template path |
| `minimax_h3_max` | MiniMax H3 Max | — | — | skills only |
| `flux_3_video_edit`, `kling_video_edit` | Flux 3 / Kling video edit | — | — | skills only |
| `cinematic_studio_video_4_0`, `_3_5`, `_v2`, `cinematic_studio_video`, `cinematic_studio_3_0` | Cinema Studio video | many | multi-shot, genre, speed-ramp, camera/lighting presets | suspected app-only (brief §13 puts multi-shot out of scope) |
| `marketing_studio_video` | Marketing Studio video | refs, frames | avatars, products, hooks, settings | suspected app-only (§13 out of scope) |
| `video_explainer`, `explainer_video` | Explainer pipeline | — | — | suspected app-only |
| Genjutsu motion transfer | — | — | — | Brief cites `/higgsfield/genjutsu/motion-transfer/v1.0` from the docs; no source we have mentions it. Verify. |

### 2.3 Audio (all Unverified; output shape also undefined, see D-017)

| CLI id | Name | Key params (M) |
|---|---|---|
| `seed_audio` | Seed Audio 1.0 | prompt; format wav/mp3/pcm/ogg_opus; sample_rate; speech/pitch/loudness rate; voice_id + voice_type; ≤2 audio refs **or** image refs |
| `sonilo_music` | Sonilo Music | prompt; duration (required) |
| `mirelo_text_to_audio` | Mirelo text-to-audio (SFX) | prompt; duration (required) |
| `inworld_text_to_speech` | Inworld TTS | prompt; voice (required, 113 voices) |
| `text2speech_v2` | Text to Speech | prompt; variant elevenlabs/minimax/seed_speech/vibe_voice/cozy_voice; voice_id + voice_type (required) |
| `qwen_audio_tts` | Qwen TTS | skills only |

The audio surface also needs a voice catalog (`higgsfield voices list` in the
CLI); no REST equivalent is known.

### 2.4 Utilities, 3D and analysis

| CLI id | What | Params (M) |
|---|---|---|
| `outpaint` | Extend an image | exactly 1 image; aspect (11) =21:9; no prompt |
| `image_background_remover` | Image cut-out | exactly 1 image |
| `video_background_remover` | Video cut-out | exactly 1 video |
| `bytedance_image_upscale`, `topaz_image`, `bytedance_video_upscale`, `topaz_video`, `video_upscale` | Upscalers | **no schema published** |
| `video_deflicker`, `color_grading_lut`, `clipify`, `sam_3_video`, `speech2text`, `llm_text`, `autosprite` | misc | no schema |
| `image_to_3d`, `multi_image_to_3d`, `sam_3_3d`, `tripo_3d`, `3d_rigging` | 3D | see M:825–924. **Out of scope** (brief §13) |
| `brain_activity` | Virality Predictor (video → text report) | 1 video. Probably out of scope |
| Soul ID training | `higgsfield soul-id create` (5–20 face photos) | **v2 scope** (§13). Legacy SDK v1 endpoints `/v1/custom-references[/list|/{id}]` exist; no v2 REST path known |

---

## 3. The brief's "check and add" list, as of Phase 0

| Brief item | Where it exists | Status |
|---|---|---|
| Genjutsu motion transfer | brief only | Unverified |
| Nano Banana / 2 / 2 Lite / Pro | M (`nano_banana`, `nano_banana_flash`, `nano_banana_2_lite`, `nano_banana_2`) | Unverified (Pro has SDK hint `nano-banana-pro`) |
| GPT Image 2 / 2.5 | M | Unverified |
| Seedream 4.5 / 5.0 Lite | M | Unverified |
| Flux Kontext | M | Unverified (SDK hint) |
| Kling O1 Image | M | Unverified |
| Soul Cast / Soul Location | M | Unverified |
| Cinema Studio image & video | M + S | Unverified, suspected app-only |
| Marketing Studio image & video | M + S | Unverified, suspected app-only |
| Veo 3 / 3.1 / 3.1 Lite | M | Unverified |
| Gemini Omni Flash | M | Unverified |
| Seedance 1.5 Pro | M | Unverified |
| Upscale, outpaint, image/video background removal | M (upscalers schema-less) | Unverified |
| Seed Audio, TTS (Inworld and others), Mirelo, Sonilo | M | Unverified |
| Soul ID training | CLI + legacy SDK | v2 scope |

The home-page feature cards the brief names (Seedance 2.5, Nano Banana Pro,
Kling 3.0, Soul 2) include **Nano Banana Pro, which is Unverified**. It can't
appear on Explore until its REST path is confirmed.

---

## 4. Picker metadata (draft, our voice)

Descriptions are reworded from `skills/.../model-catalog.md` (MC). Badges come
only from MC wording: TOP (top-tier / SOTA / default high-fidelity), FAST,
4K, AUDIO, NEW. Template entries the skills don't profile get a plain
description and no badge.

| id | Category label | Description | Badges |
|---|---|---|---|
| `seedance-2.5` | GENERAL | Our all-round video model: 4–30 s shots, references, multi-shot motion. | TOP |
| `seedance-2.5-edit` | EDIT | Rework an existing clip with a prompt and references. | — |
| `seedance-2.5-extend` | EDIT | Continue an existing clip with a new shot. | — |
| `seedance-2` | GENERAL | Seedance with native 4K output and sound. | 4K, AUDIO |
| `seedance-2-fast` | FAST | Quicker Seedance 2.0 at 480p/720p. | FAST |
| `seedance-2-mini` | FAST | Lighter Seedance 2.0, up to 720p. | — |
| `kling-3-turbo` | FAST | Quick Kling for simple motion or animating one frame. | FAST |
| `kling-3-std`, `kling-3-pro` | CINEMATIC | Kling 3.0 with multi-shot and synced sound. | AUDIO |
| `kling-3-4k` | CINEMATIC | Kling 3.0 rendered at 4K. | 4K, AUDIO |
| `kling-3-motion-std`, `-pro` | MOTION | Drive a character image with the motion from a video. | — |
| `kling-2.6` | CINEMATIC | Earlier Kling with strong physics. | — |
| `wan-2.7` | GENERAL | Character-consistent video with synced sound. | NEW, AUDIO |
| `wan-2.6` | STYLIZED | Open-weight, stylized, experimental video. | — |
| `minimax-hailuo-2.3` | GENERAL | Natural physics at a low price; no sound. | — |
| `grok-imagine-video-1.5` | STYLIZED | Bold, stylized motion. | — |
| `soul-2` | PORTRAIT | Fashion, editorial and UGC-style people. | — |
| `soul-cinema` | CINEMATIC | Cinematic stills with film-grade lighting. | — |
| `recraft-4.1` | DESIGN | Clean vector-style logos, icons and flat illustration. | — |
| `flux-2` | GENERAL | Precise prompt following with a distinct look. | — |
| `z-image-turbo` | FAST | The quickest drafts for fast iteration. | FAST |
| `wan-3`, `wan-3-prime`, `minimax-h3`, `flux-3`, `happy-horse-1`, `happy-horse-1.1`, `grok-imagine-2`, `ideogram-4`, `qwen-image-3`, `kling-2.5`, `kling-o1`, `kling-o3`, `ltx-2.5-fast`, `ltx-2.5-pro`, `pixverse-6`, `dop` | per family | Plain one-liners from the model's docs page in Phase 4 (no skill source today). | — |

"from ✦N" prices in the picker wait on the console price list.
