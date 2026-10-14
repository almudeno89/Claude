# UI inventory

What we build, page by page, and which template parts each page uses.

- **Part A (this file):** the target pages, global chrome, generator layout,
  required states and modals, mapped to template components, plus the gaps
  we must close.
- **Part B ([`ui-inventory-template.md`](ui-inventory-template.md)):** a
  file-by-file inventory of Higgsfield's Studio template (Higgsfield's own
  studio UI code): every component's props, states, copy, controls,
  responsive rules, design tokens, the `useRuns` contract and gallery
  internals, with `file:line` citations.

## Capture status: the live site has NOT been captured

`higgsfield.ai` is blocked by this session's network policy
(`docs/decisions.md` D-001), so nothing in this file was observed on the live
site. `docs/screens/` is empty. Page structure below comes from the build
brief (§1c, §9) and from the template's components. **Every "Live-site check"
line is an open item**, to be filled from screenshots at 1440px and 390px
before Phase 6 starts. Spacing, exact copy patterns, hover/empty states and
mobile behavior of the real site are unknown until then.

Pages to capture once the host is reachable (from brief §1c): `/`,
`/ai/image?model=gpt_image_2`, `/ai/video?model=seedance_2_5`, `/audio`,
`/effects/use` plus one effect detail, `/community`, `/pricing`, the
login/sign-up modal (look only), the generator's History tab, the asset
library, and a generation's lightbox. Decline cookie banners; submit nothing.

---

## A1. Global chrome

| Element | Spec (brief §9) | Template support | Work needed | Live-site check |
|---|---|---|---|---|
| Top nav | Logo · Explore · Image · Video · Audio · Effects · Community · Pricing | **None.** The template rule is "no app header" (`AGENTS.md`) | New `SiteHeader`. Brief wins over the template for page structure (D-020) | Height, sticky behavior, active-state style, logo size, dropdowns under Image/Video? |
| Signed-in right side | Credit pill (✦ + balance → `/account`), lime **Upgrade**, avatar menu (Assets, Account, Billing, Sign out) | None. `components/ui/dropdown-menu.tsx` exists for the avatar menu | New `CreditPill`, `AccountMenu` | Pill style, Upgrade visibility on paid plans |
| Signed-out right side | **Login**, **Sign up** (open the auth modal) | None | New `AuthModal` trigger | Which button is primary |
| Mobile nav | Collapses into a sheet | None (the template sidebar doesn't even collapse on phones, Part B §12.13) | New sheet using `components/ui/dialog.tsx` | Breakpoint; what stays visible (logo + credits?) |
| Promo bar | Optional, dismissible, copy from config | None | New `PromoBar` | Height, color, dismiss persistence |
| Cookie consent | Accept all / Reject non-essential / Preferences; no analytics before consent | None | New `CookieBanner` + preferences dialog | Placement |
| Projects sidebar | Template's `StudioSidebar` (Home, All Generations, projects, key row) | Yes (`sidebar.tsx`) | Keep inside `/assets` and project views only; drop the key row; add responsive collapse | Does the live site show a projects rail? |

Lime is reserved for the Generate CTA and active states (template rule). The
brief adds a lime **Upgrade** button; that is our one other lime use. Record
in D-020.

## A2. Routes

"Template parts" names components from Part B. "New" is what we write.

| Route | Site equivalent | Purpose | Template parts | New | Required states |
|---|---|---|---|---|---|
| `/` Explore | `/` | Hero (glow + dot field + uppercase headline), feature cards for top models → generators with `?model=`, Effects strip, Community teaser | `HERO_GLOW`/`HERO_DOTS` atmosphere, `HeroComposition`, `ExamplePresets`/`TemplateCard`, `UserGenerations` (read-only, for the teaser) | Model feature cards, Effects strip, Community teaser block | Loading (skeleton), signed-out, empty community |
| `/image`, `/video`, `/audio` | `/ai/image`, `/ai/video`, `/audio` | Generator (layout in A3); `?model=`, `?preset=` | `PromptBox.*` parts, `AssetLibraryModal`, `SettingsDialog`, `UserGenerations`, `GenerationTile`, `GenerationDetailModal`, `ScreenEmptyState`, `DensityControl` | Left control panel layout, model card, `ModelPickerModal`, audio player tile, upgrade modal hook, elapsed-time label | All tile states (A4); signed-out empty; insufficient credits |
| `/effects`, `/effects/[slug]` | `/effects/use` | Preset grid with looping covers + category chips; detail with "Use effect" → generator pre-filled | `TemplateCard` / `ExamplePresets` (needs a video-cover variant) | Category chips, detail page, `presets` data source | Empty category, loading |
| `/community`, `/g/[id]` | `/community` | Published generations feed; detail with prompt/model/settings, **Recreate**, **Report** | `UserGenerations` (public variant), `GenerationTile`, `GenerationDetailModal` | Public feed query + pagination (the gallery's dormant `hasMore`/`onLoadMore`), report dialog | Empty, loading more, item removed |
| `/assets` | asset library | All / Images / Videos / Audio / Uploads tabs + projects sidebar; multi-select, download, delete, move to project, publish | `StudioSidebar`, `MyProjects`, `UserGenerations`, `AssetLibraryModal` (for uploads), `components/ui/tabs.tsx` | Multi-select mode on the feed, bulk action bar | Empty per tab, 500+ items |
| `/projects/[id]` | — | Feed filtered by `projectId`; rename/delete | `UserGenerations`, `ProjectActions`, `ProjectCreateModal` | Server-backed projects | Empty project |
| `/pricing` | `/pricing` | Monthly/annual toggle, plan cards (Most popular / Best value), comparison table, top-ups, FAQ | `components/ui/tabs.tsx` (toggle), `button.tsx` | All of it | Signed-out vs current plan |
| `/account`, `/account/billing` | — | Profile, balance by bucket, ledger history, plan, portal link | — | All of it | Empty ledger |
| `/login`, `/login/check-email`, `/login/error` + modal | auth modal | Magic-link email field | `components/ui/input.tsx`, `dialog.tsx` | `AuthModal`, pages | Sending, sent, rate-limited, error |
| `/admin` | — | Users, credit adjust, generations, model toggles/pricing, presets CRUD, moderation | ui primitives | All of it | — |
| `/legal/*` | — | Placeholders | — | Pages | — |

## A3. Generator page layout

The brief describes higgsfield.ai's `/ai/video`: a **left control panel
(~380px, own scroll, sticky footer)** and a **right canvas**. The template's
Studio instead uses a **horizontal prompt dock** floating over the feed
(`layouts/studio.tsx`, Part B §11). The brief makes the site's layout the
source of truth for page structure, so we compose the template's `PromptBox`
parts into a vertical panel rather than using `StudioPromptBox` as-is
(D-021). Same components, same controlled dock state, different arrangement.

| Panel section (top → bottom) | Brief spec | Template part | Gap |
|---|---|---|---|
| 1. Mode tabs | Video: Create / Edit / Motion Control. Image: Create / Edit | `PromptBox.ModeRail` switches **surfaces** (Image/Video), not modes | New tabs mapping to catalog entries/`mediaModes` (e.g. Edit → `seedance-2.5-edit`, Motion Control → `kling-3-motion-*`) |
| 2. Model card | Looping preview, category label, name, **Change** → model picker modal (searchable grid by family, icon, one-liner, badges, "from ✦N") | Model `PillSelect` dropdown | New `ModelCard` + `ModelPickerModal`; metadata from `docs/model-catalog.md` §4 |
| 3. Sub-tabs | References / Extend Video, or Frames / References | `mediaModes` + `inferInputMode` (the template infers mode from attachments) | Explicit sub-tabs that set the input mode; keep inference as the fallback |
| 4. Add references | Drop zone "Image, Video or Audio" → `AssetLibraryModal`; removable chips; frame/source slots only when supported; per-kind limits | `ReferencePicker` + `MediaInputs` chips + frame/source `PromptBox.Upload` tiles + `AssetLibraryModal` with `kindLimits` | A drop zone on the panel itself (the template only drops inside the modal) |
| 5. Prompt | Textarea with placeholder hints, "Enhance prompt" toggle when supported, character count | `PromptBox.Field` (no auto-grow, clips long text: Part B §12.4) | Auto-grow, counter, enhance toggle bound to the model setting |
| 6. Settings row | Headline pill (aspect or duration), resolution pill, "More" → `SettingsDialog` | One headline `PillSelect` + settings pill | A second inline pill; fix pill focus rings (§12.2) |
| 7. Sticky Generate | Lime full width, "Generate ✦ {cost}"; **Cancel** while a queued run is selected; disabled with reason on hover; opens upgrade modal when short of credits | `PromptBox.Generate` has `cost`/`oldCost` props (never passed); CTA turns into Cancel while *anything* runs | Pass `cost`; Cancel only for a *selected queued* run (parallel runs allowed per plan); make the disabled reason reachable (§12.1) |

Right canvas:

| Element | Brief spec | Template part | Gap |
|---|---|---|---|
| Header | **History** / **How it works** toggles, density control | `DensityControl` in the gallery header | Toggles + "How it works" panel |
| Empty state | Big uppercase headline ("MAKE VIDEOS IN ONE CLICK"), subline, three step cards | `ScreenEmptyState` (three images + title + description + action) | Step-card variant |
| Feed | Justified virtualized `UserGenerations` filtered to this surface, newest first | `UserGenerations` | Surface filter; wire `hasMore`/`onLoadMore` for server pagination |
| Lightbox | Media, prompt, model, settings, cost, created time, same actions | `GenerationDetailModal` | Cost row; camelCase setting labels render as "AspectRatio" (fix with `settingLabel`) |

Mobile: panel stacks above the feed; Generate becomes a bottom sticky bar;
never a second app shell. The template's own ≤640px dock rules (Part B §3.1)
are a starting point.

## A4. Generation tile states

| State | Brief requires | Template today (Part B §5) | Change |
|---|---|---|---|
| pending / queued / in progress | Shimmer **with elapsed time** | `generating` pulse + "Generating", no time | Add elapsed time; queued tiles get Cancel |
| completed | Media; hover rail | Ready tile(s) | Serve R2 URLs |
| failed | Visible tile + "Credits refunded" | Failed tile with error text | Plain-language error + refund note |
| nsfw | "Blocked by content filter. Credits refunded." | Failed tile "Blocked by the content filter" | Copy + refund note |
| canceled | Visible tile, refunded | Failed tile "Canceled", set client-side before the platform confirms | Finalize from the platform's `canceled` status |
| submit failed | Visible outcome | **No tile**, dock error only (§12.6) | Server creates the generation first, so a failed submit is a failed tile |
| initial load | No flash of empty state | None (§12.5) | Loading skeleton |

Hover rail (≤3 visible, rest behind ⋯, `CardActions`): Download, Reuse prompt
& settings, Animate (image → video with this image as start frame), Upscale
(only if the API exposes it; Unverified today), Publish to Community, Add to
project, Delete (with confirmation; the template deletes with none, §12.15).
The template rail has Download / Copy prompt / Share / Delete.

## A5. Modals

| Modal | Template | Status |
|---|---|---|
| Auth (magic link) | — | New |
| Model picker | — (dropdown only) | New |
| Upgrade / insufficient credits | — | New |
| Settings | `SettingsDialog` | Keep; fix switch accessible name (§12.3) |
| Asset library | `AssetLibraryModal` | Keep; Uploads tab backed by the `uploads` collection |
| Lightbox | `GenerationDetailModal` | Keep; add cost, reduced-motion autoplay gate |
| Project create/rename/delete | `ProjectCreateModal`, `ProjectActions` | Keep; fix the rename reset bug (§12.7) |
| Report content | — | New |
| Cookie preferences | — | New |
| API key | `KeyDialog` | **Delete** (platform key is server-side) |

## A6. Design language (summary; full tables in Part B §8)

- Dark only. Background `#131416`, card `#1c1e21`, popover `#2a2d32`, text
  `#ffffff`, muted text `#828282`, border `#ffffff1a`, destructive `#fa0019`.
- Primary lime is exactly **`#d1fe17`** (`--hf-color-lime-500`). The Generate
  CTA lays a yellow `#ffff14` sheen gradient over the lime.
- Fonts via `next/font`: **Space Grotesk** (headings, uppercase display),
  **Inter** (body), **IBM Plex Mono** (numbers). Type scale steps up at 768px
  and 1280px. The brief asks for "bold condensed uppercase display headings";
  Space Grotesk is not condensed. Confirm against the live site, and pick a
  condensed display face only if the capture shows one (open item).
- Radius base 12px; Tailwind `rounded-2xl` is 21.6px here.
- Motion tokens with reduced-motion handling for modals, dropdowns, CTA,
  glow and hover video. Not handled: spinners, sidebar width, smooth scroll,
  lightbox fade and autoplay (Part B §8.6). We fix those.
- Z-index: dropdown 10, overlay 30, modal 40, lightbox and tooltip raw 50.
  Selects inside modals may render under the modal; verify in a browser
  (§12.17).

## A7. Template defects we inherit and must fix

From Part B §12, in the order they affect users:

1. Generate's disabled reason can never be seen (`pointer-events:none`).
2. No keyboard focus ring on dock pills; failed-tile Remove is hover-only;
   settings switch has no accessible name.
3. Prompt field doesn't grow and clips long prompts.
4. Empty state flashes before history loads.
5. Submit failures create no tile.
6. Renaming a project a second time opens an empty field.
7. Videos never appear in hero, project covers or mosaics; feed tiles use the
   requested aspect ratio, so `auto` renders square.
8. Lime used outside CTA/active states (Try, Download, Add, switch, slider).
9. Sidebar doesn't collapse on phones (256px + 20px at 390px wide).
10. No toast feedback for copy/share/download.
11. Tile delete has no confirmation.

## A8. Copy

Every user-visible template string is listed in Part B §7c and §5. All of it
is placeholder or Higgsfield-branded and gets replaced with our copy. The
Higgsfield name, logo and marketing copy never appear to users (brief §1d);
third-party model icons in `public/model-icons/` may stay next to model names.
