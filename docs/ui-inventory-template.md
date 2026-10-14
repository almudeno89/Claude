# Template UI inventory (Part B of `docs/ui-inventory.md`)

Source: `_reference/app-templates` at commit `1e15e22` (read-only). All paths below are repo-relative; `file:N` / `file:N-M` are 1-based line numbers in that file. Strings in "double quotes" are verbatim user-visible copy (including aria-labels, titles, placeholders, sr-only text). `{x}` marks an interpolated value.

Contents
1. File map
2. Shell files: `app/layout.tsx`, `app/page.tsx`, `app/globals.css`
3. `components/studio/**`, one entry per file
4. `components/ui/**`
5. Run-state matrix: where each generation state renders, and the copy it shows
6. Template-only dependencies to remove (BYOK key cookie, device cookie, browser persistence)
7. Placeholders and scaffold copy to replace before shipping
8. Design tokens
9. `useRuns` contract
10. Gallery internals
11. Studio layout (`layouts/studio.tsx`)
12. Gaps and bugs found while reading (for planning)

---

## 1. File map

| File | Exports | Purpose |
|---|---|---|
| `app/layout.tsx` | `metadata`, `viewport`, `default RootLayout` | Root HTML, fonts, dark theme pin |
| `app/page.tsx` | `default Page` | Renders `NoModels` if catalog empty, else `StudioTemplate` |
| `app/globals.css` | (CSS) | Tailwind/shadcn/Quanta imports, shadcn-variable bridge, `.studio-glow`, `.studio-tile` |
| `layouts/studio.tsx` | `StudioTemplate`, `StudioTemplateProps` | The full screen. Owns all state and wiring (see section 11) |
| `components/studio/studio-prompt-box.tsx` | `StudioPromptBox`, `StudioPromptBoxProps`, `MAX_INLINE_SETTINGS`, `headlineSetting` | Prop-driven prompt dock |
| `components/studio/prompt-box.tsx` | `PromptBox` (`Root/ModeRail/Mode/Body/Field/Actions/Pill/Uploads/Upload/Generate`), `PromptBoxSurface` + part prop types | Composition primitive for the dock (styles in `styles/studio/prompt-box.css`) |
| `components/studio/asset-library.tsx` | `AssetLibraryModal`, `AssetLibraryModalProps` | The single media picker/upload modal |
| `components/studio/settings-dialog.tsx` | `SettingsDialog`, `SettingsDialogProps`, `settingLabel` | Renders every `model.settings` field |
| `components/studio/generation-card.tsx` | `GenerationCard`, `GenerationCardProps`, `GenerationCardState`, `CardActions`, `CardAction`, `GenerationTile`, `GenerationTileProps` | The only renderer for one generation |
| `components/studio/generation-detail.tsx` | `GenerationDetailModal`, `GenerationDetail`, `GenerationDetailRow`, `GenerationDetailModalProps` | Full-screen lightbox |
| `components/studio/user-generations.tsx` | `UserGenerations`, `UserGenerationsProps` | Public feed wrapper around `gallery/` |
| `components/studio/gallery/justified-gallery.tsx` | `JustifiedGallery`, `JustifiedGalleryProps` | Virtualized feed renderer (private) |
| `components/studio/gallery/gallery-tile.tsx` | `GalleryTile` (memo), `GalleryTileProps` | One positioned feed tile (private) |
| `components/studio/gallery/justified-engine.ts` | `JustifiedLayoutEngine`, `TileRect`, `LayoutRow`, `LayoutRowType`, `Layout`, `EngineConfig` | Layout and windowing math (no React) |
| `components/studio/gallery/use-justified-gallery.ts` | `useJustifiedGallery`, `UseJustifiedGalleryResult`, `DENSITY_ROW_HEIGHTS`, `DEFAULT_DENSITY` | React hook: measure, window, anchor, infinite scroll |
| `components/studio/gallery/density-control.tsx` | `DensityControl` | Tile-size slider |
| `components/studio/gallery/gallery-types.ts` | `GalleryItem`, `MediaKind`, `ItemStatus`, `LoadTier` | Feed data model |
| `components/studio/gallery/use-reduced-motion.ts` | `useReducedMotion` | matchMedia hook |
| `components/studio/screen-empty-state.tsx` | `ScreenEmptyState`, `ScreenEmptyStateProps`, `ScreenEmptyStateContent` | Full-screen empty state |
| `components/studio/hero-composition.tsx` | `HeroComposition`, `HeroCompositionProps` | Three-frame hero |
| `components/studio/template-picker.tsx` | `TEMPLATES`, `TemplateItem`, `TemplateCard`, `TemplateCardProps`, `ExamplePresets`, `ExamplePresetsProps` | Explore presets (mock data) |
| `components/studio/my-projects.tsx` | `MyProjects`, `MyProjectsProps`, `MyProjectsProject`, `ProjectCard` | Projects grid on Home |
| `components/studio/project-dialogs.tsx` | `ProjectCreateModal`, `ProjectActions` (internal `NameDialog`) | Create / rename / delete project UI |
| `components/studio/sidebar.tsx` | `StudioSidebar`, `StudioSidebarProps`, `StudioView` | Left rail |
| `components/studio/icon-tile.tsx` | `IconTile`, `IconTileProps`, `IconTileGradient`, `ICON_TILE_GRADIENT` | 24px gradient nav glyph |
| `components/studio/key-dialog.tsx` | `KeyDialog` | BYOK API key dialog (template-only) |
| `components/studio/no-models.tsx` | `NoModels` | Developer screen for an empty catalog |
| `components/ui/*.tsx` | shadcn-named wrappers over Base UI (`@base-ui/react`) with Quanta skins | See section 4 |

None of the `components/studio/*` files imports a zustand store, IndexedDB, or a cookie helper. The only exception is `key-dialog.tsx`, which calls server actions. All persistence wiring is in `layouts/studio.tsx`, which keeps migration contained.

---

## 2. Shell files

### `app/layout.tsx`: `RootLayout`, `metadata`, `viewport`
- **Fonts** (`app/layout.tsx:2,9-24`), all from `next/font/google`, `subsets:["latin"]`, `display:"swap"`:
  - `Space_Grotesk` → CSS var `--font-space-grotesk`
  - `Inter` → `--font-inter`
  - `IBM_Plex_Mono` weights `["400","500"]` → `--font-ibm-plex-mono`
  - The variables go on `<html>`. `styles/theme.css:9-12` maps them to `--hf-type-family-*-base`.
- **`<html>`** (`:38-47`): `lang="en"`, `data-theme="default-dark"` (selects the Quanta dark theme, `styles/theme.css:8`), class `"dark font-sans antialiased"` plus the font variables.
- **`<body>`** (`:48`): `min-h-svh bg-background text-foreground`.
- **Copy to replace** (`:26-30`): `metadata.title` = "Studio", `metadata.description` = "Generate images and video with Higgsfield models.". The comment "Replace with the product's own identity when adapting the scaffold." is at `:26`.
- `viewport.themeColor` = `"#131416"` (`:32`), equal to `--hf-color-cool-400` (the dark background).
- No states, controls, or responsive rules.

### `app/page.tsx`: `Page`
- If `MODELS.length === 0`, renders `<NoModels />`. Otherwise renders `<StudioTemplate />` with default props, so the title is "Studio" and the headline is "Turn any idea into images and video" (`app/page.tsx:6-9`, `layouts/studio.tsx:267-270`).

### `app/globals.css`
- Imports, in order: `tailwindcss`, `tw-animate-css`, `shadcn/tailwind.css`, `../styles/quanta/index.css`, `../styles/theme.css`, `../styles/studio/prompt-box.css` (`:1-6`).
- `@custom-variant dark (&:is(.dark *))` (`:8`).
- `@theme inline` (`:10-52`) maps the shadcn vars to Tailwind colors. Fonts: `--font-heading` = grotesk, `--font-sans` = secondary (Inter), `--font-mono` = mono. Radius scale is derived from `--radius` (= `--hf-radius-300` = 0.75rem = 12px): `sm`=×0.6 (7.2px), `md`=×0.8 (9.6px), `lg`=12px, `xl`=×1.4 (16.8px), `2xl`=×1.8 (21.6px), `3xl`=×2.2 (26.4px), `4xl`=×2.6 (31.2px). So `rounded-2xl` in components is 21.6px, not Tailwind's 16px.
- Base layer (`:54-64`): `* { border-border outline-ring/50 }`, body `bg-background text-foreground`, html `font-sans`.
- `.studio-glow` (`:68-76`): lime radial `--hf-color-transparent-lime-40` → `-15` → transparent, `animation: studio-pulse 2600ms ease-in-out infinite`. Used by the generating tile.
- `.studio-tile` (`:77-83`): `position:absolute; overflow:hidden; border-radius: var(--radius); background: var(--card); content-visibility:auto`. Wraps every feed tile.
- `@keyframes studio-pulse` opacity 0.22 ↔ 0.6 (`:86-89`). Under reduced motion: `.studio-glow { animation:none; opacity:0.7 }` (`:91-93`).

---

## 3. `components/studio/**`

### 3.1 `studio-prompt-box.tsx`: `StudioPromptBox` (the dock)
**Purpose:** prop-driven composition of `PromptBox`. Contains the mode rail, prompt field, inline error, reference picker, model pill, one headline-setting pill, settings-dialog pill, frame/source tiles, and the lime Generate/Cancel CTA.

**Props** (`:60-91`):
`surfaces: Surface[]`, `surface: Surface`, `onSurfaceChange(s)`, `models: ModelEntry[]`, `model: ModelEntry`, `onModelChange(id)`, `settings: Record<string,unknown>`, `onSettingChange(key,value)`, `inputMode?: string`, `media: MediaItem[]`, `onMediaAdd(role: MediaRole|"auto", selection: AssetSelection[])`, `onMediaRemove(id)`, `onMediaRoleChange(id, "reference"|"start"|"end")`, `library: { items: AssetLibraryItem[]; onUpload(file): Promise<AssetSelection> }`, `placeholder?` (default "Describe the scene you imagine..." `:515`), `prompt`, `onPromptChange`, `cost?: ReactNode` (never passed by the studio), `onGenerate`, `onCancel`, `generating?`, `canceling?`, `generateDisabled?`, `disabledReason?`, `error?: ReactNode`, `skin?: "plain"|"glass"`, `className?` (default `"w-[830px] max-w-full"` `:527`).

**Reads:** no stores. Fully controlled. It uses catalog helpers `inputRoles`, `MEDIA_ROLES`, `REFERENCE_ROLE`, `ROLE_KIND`, `ROLE_LABEL` (`generation/catalog/media-inputs.ts`) and `settingLabel`.

**Exports:** `MAX_INLINE_SETTINGS = 1` (`:56`) is exported but nothing reads it; the cap is implicit because only one headline pill is rendered. `headlineSetting(model)` (`:106-111`) returns the first enum key matching `/aspect|ratio/`, else the first enum key.

**Parts, states and copy**
- **Mode rail** (`:533-544`): unmounted when `surfaces.length < 2`. One `PromptBox.Mode` per surface. Labels: "Video" (Clapperboard icon), "Image" (ImageIcon). Active mode gets `aria-pressed` and the class `q-prompt-box-mode-active` (lighter fill, primary text).
- **Body > MediaInputs (inline chips)** (`:312-498`, rendered at `:546-556`). Shown when the model has single video/audio slots or any attached references.
  - Single `video`/`audio` slots render as 48px buttons labelled "Video" / "Audio" (`:393-398, 419-432`). aria-label: "Add video" or "Replace video", "Add audio" or "Replace audio" (`:421`). Empty slot shows a Plus icon; a filled slot shows `MediaPreview`.
  - Reference chips (`:445-474`): 48×48 thumbnail. A video chip adds a small Play overlay. Audio uses `AudioChip`. Per-kind ordinal labels "Image {n}" / "Video {n}" / "Audio {n}" (`:452`). Hover `title` = "{label}: {name ?? url}" (`:457`).
  - `AudioChip` (`:204-240`): 32px-wide play/pause disc. aria-label "Play {name ?? label}" or "Pause {name ?? label}" (`:213`). Label text, or "Cannot play" after a play/load error (`:224`).
  - `MediaPreview` image alt: `{item.name ?? "Reference image"}` (`:198`).
  - Role menu, image items only, when the model supports `reference` and ≥2 of reference/start/end (`:343-376`). Trigger is a ⋯ (Ellipsis) at the top-left with aria-label "Role for {name ?? "image"}". Items: "Reference image", "Start frame", "End frame" (ROLE_LABEL). A Check marks the current role.
  - Remove button (X, top-right): aria-label "Remove {name ?? ROLE_LABEL[role]}" (`:380`).
  - Chip controls (`controlClass` `:242-243`) are always visible below `sm` (640px). At `sm` and up they sit at `opacity-0` until `group-hover`, `group-focus-within`, or `data-[popup-open]`.
  - "Clear attachments" trash (`:475-485`): aria-label and title both "Clear attachments". Shown only when `media.length > 1`. It removes every item by calling `onMediaRemove` in a loop.
  - Container: aria-label "Attached media", `max-h-36 overflow-y-auto` (`:491-496`).
- **Field** (`:557-575`): textarea, `placeholder` = `aria-label` = "Describe the scene you imagine...". `rows=1`; CSS min-height 48px, `resize:none; overflow:hidden` (`prompt-box.css:145-165`). There is **no auto-grow**.
- **Error line** (`:576-583`): `<p role="alert">` in `text-q-state-error-fg`, rendered when `error != null`. The upstream copy sources are listed in section 5.
- **Actions row** (`:584-625`, `flex-wrap`):
  - `ReferencePicker` "+" pill (`:257-310`): `iconOnly`, aria-label and `title` "Add references". `disabled` when no capacity remains, with `disabled:opacity-40` (`:303-304`). Returns `null` when the model accepts no reference kinds outside slots. Opens `AssetLibraryModal` in multi-select mode with `accept` = allowed kinds, `kindLimits` = remaining per kind, `maxSelections` = total remaining, and `excludeIds` = attached URLs.
  - Model `PillSelect` (`:588-597`): aria-label "Model". Shows the current model label; the start icon is `/model-icons/{icon}.svg` (inverted white) or a Clapperboard fallback (`:93-104`). The trailing ChevronDown is shown in every state. Popup settings: `PICKER_POPUP` = picker variant, solid surface, bottom/start, offset 8, collision padding 16 (`:113-120`).
  - Headline-setting `PillSelect` (`:598-612`): aria-label is `settingLabel(key)`, e.g. "Aspect ratio". Options are raw enum values ("16:9", "auto", ...).
  - Settings pill (`:613-624`): `iconOnly`, aria-label "All settings", SlidersHorizontal icon. Opens `SettingsDialog`.
- **Frame/source tiles** (`MediaInputs tilesOnly`, `:627-638`): 80×80 `PromptBox.Upload` tiles for roles `start`, `end`, `source` when the active input mode allows them. Label text is "Start frame" / "End frame" / "Source video" (uppercase via CSS). aria-label "Add start frame" or "Replace start frame", and so on (`:416`). A filled tile shows the image or video with a bottom scrim, keeping the role menu and remove controls.
- **Generate CTA** (`:639-646`):
  - Label is "Generate" when idle, "Cancel" when `generating`, "Cancelling…" when `canceling` (`:645`).
  - `onClick` is `onCancel` while generating, else `onGenerate`.
  - `disabled` = `canceling || (!generating && generateDisabled)`.
  - `title` = `disabledReason` only when disabled and idle. Because `.q-prompt-box-generate:disabled` sets `pointer-events:none` (`prompt-box.css:380-384`), the tooltip never appears.
  - The `cost` meta is suppressed while generating. The studio never passes a cost, so the meta row (sparkle + cost) is never shown.

**Keyboard:** Cmd/Ctrl+Enter in the field calls `onGenerate()` when `!generating && !generateDisabled` (`:564-574`). Every other control uses the Base UI defaults: Select arrows/typeahead/Esc, Menu arrows/Esc, Dialog Esc.

**Visual states:** idle; disabled CTA (`--hf-opacity-disabled` 0.5); generating (same lime CTA reading "Cancel"); canceling (disabled, "Cancelling…"); error line; pill hover/open overlay (`data-popup-open`); disabled "+" pill.

**Responsive:** at ≤640px (`prompt-box.css:419-426`) the dock stacks vertically, the mode rail goes horizontal (60×44 buttons), the surface wraps, and Generate becomes full width at 48px tall.

**Motion:** CTA transition 140ms `cubic-bezier(0.23,1,0.32,1)`, `:active` scale 0.98 at 100ms. Under reduced motion: transform none, 1ms (`prompt-box.css:365-376, 428-433`).

**Template-only deps:** none directly. The submit path is gated by the key in `layouts/studio.tsx`.

### 3.2 `prompt-box.tsx` + `styles/studio/prompt-box.css`: `PromptBox.*`
**Purpose:** a design-owning composition primitive. Every part is optional. `hidden` unmounts a part.

**Parts and props**
- `Root` (`:90-121`): `ComponentProps<"div"> & { surface?: "plain"|"glass" }`. Sorts `ModeRail` children before a `.q-prompt-box-dock > .q-prompt-box-dock-surface` wrapper. `glass` adds `q-prompt-box-glass` (`:80-83`). Despite the name, that class makes the outer border/background transparent and removes backdrop-filter and shadow (`prompt-box.css:51-57`). The inner dock keeps `--hf-color-background-secondary`.
- `ModeRail` (`:128-135`): `hidden?`.
- `Mode` (`:152-188`): `active?`, `hidden?`, `start?` (icon), `children` (label), `render?` (Base UI `useRender`). Sets `type="button"` and `aria-pressed` when active.
- `Body` (`:195-208`): `surfaceClassName?`.
- `Field` (`:212-220`): textarea, `rows=1` default.
- `Actions` (`:223-225`).
- `Pill` (`:245-284`): `start?`, `end?`, `children?`, `iconOnly?`, `hidden?`, `render?`. 28px tall, `--hf-radius-200`.
- `Uploads` (`:291-298`), `Upload` (`:321-383`): `label?`, `src?`, `mediaType? "image"|"video"|"audio"`, `alt?`, `hidden?`, `add?` (corner glyph, default `PlusGlyph`), `children?`, `render?`. Filled audio renders `role="img"` with aria-label `{alt || "Audio reference"}` (`:352`). Filled video is muted `preload="metadata"`.
- `Generate` (`:415-452`): `cost?`, `oldCost?` (struck through), `start?` (default Sparkle glyph), `children` (default "Generate"), `render?`.

**States and CSS** (`prompt-box.css`):
- `mode`: hover overlay, `q-focus-ring`, disabled opacity 0.5.
- `pill`: hover and `[data-popup-open]` overlay, disabled. **No focus ring** (`:177-206`): `outline:none` with no `q-focus-ring`, so keyboard focus on the Model/aspect/settings/+ pills is invisible.
- `upload`: hover overlay, focus ring, filled scrim `::after` (`:291-299`).
- `generate`: lime gradient fill `linear-gradient(191deg, rgba(255,255,20,0) 12.7%, rgb(255,255,20) 75.8%)` over solid `rgb(209,254,23)` (= #d1fe17). Label is `--hf-color-text-inverse` (#1a1a1a), uppercase Space Grotesk 12px bold. Composite shadow, hover `brightness(1.04)`, focus ring, disabled (`:340-385`).
- Literal sizes: rail 70px (`--q-prompt-box-rail-width`), CTA 112px wide × 80px tall (`--q-prompt-box-generate-width`, `--hf-space-2000`), mode button 50px, upload tile 80px.
- Responsive and reduced motion as in 3.1.

### 3.3 `asset-library.tsx`: `AssetLibraryModal`
**Purpose:** the picker and uploader behind every "+" / slot. It returns durable public URLs.

**Props** (`:60-73`): `trigger: ReactElement`, `items: AssetLibraryItem[]`, `onUpload(file): Promise<AssetSelection>`, `maxSelections?` (default 1), `kindLimits?: Partial<Record<"image"|"video"|"audio",number>>`, `excludeIds?: string[]` (matched against `item.src`), `accept?: AssetKind[]`, `defaultOpen?`. Plus exactly one of `onSelect(selection)` (single mode) or `onSelectMany(selection[])` (multi mode).

**Reads:** `kindOf` from `lib/studio/uploads.ts` (a pure MIME → kind function that throws on unsupported types). No store.

**Structure, states and copy**
- Dialog `size="xl"` (946px, `min(…, 100vw-32px)`). sr-only title "Asset library", sr-only description "Pick a reference from your uploads or generations." (`:213-216`).
- Header tabs (`variant="pill"`, `:217-236`): "Uploads", "Images", "Videos", "Audio" (`:53-58`), filtered by `accept`. Uploads is always shown. Tabs are disabled while uploading. Close disc (`DialogCloseButton`, aria-label "Close") is disabled while uploading (`:237`).
- Toolbar: count "{n} element" or "{n} elements" (`:243-245`). Search input with aria-label "Search assets", placeholder "Search", Search icon, fixed `w-48` (`:246-254`). It filters by `name`, case-insensitive.
- Error banner `<p role="alert">` (`:256-265`). Copy:
  - "You can add {maxSelections} more files to this input." (`:151`)
  - "Choose up to {remaining} more files for this input." (`:161`)
  - "Only {accept.join(" or ")} files can be used here." (`:166`)
  - "You can add {limit} more {kind} files." (`:173`)
  - "Upload failed — check your connection and try again." (`:191`, fallback)
  - "Could not attach this media." (`:128`, fallback when the parent's `onSelect*` throws)
  - Any message thrown by `onUpload` / `onSelect*`. That includes the upload errors in section 5 and the `mergeReferences`/`mergeMedia` errors such as "{kind} references are not supported by this model." and "{ROLE_LABEL}: maximum {n} attachments."
- Grid area (`:266-318`): fixed `h-[595px]`. Columns: 3, `sm` 4, `lg` 5, gap 3px.
  - **Drag & drop:** only on the Uploads tab. `dragOver` sets an inset `ring-2 ring-q-border-focus`; drop calls `upload(files)`.
  - **Empty state** (`EmptyState` `:467-514`): an AtSign icon, or a spinning Loader2 while uploading. Title "No elements yet" on Uploads, else "No {tab} files yet" where `{tab}` is `image|video|audio`, giving "No image files yet". Description "Upload or drop a file to reuse it across every generation" on Uploads, else "Upload a reference or generate media to see it here". On Uploads the whole area is a button with aria-label "Upload media".
  - **UploadCard** (first cell on the Uploads tab, `:361-391`): Plus, or a spinner while uploading. Label "Upload" / "Uploading…". `disabled:opacity-60`.
  - **ElementCard** (`:393-465`): 148px tall with a 96px thumbnail. Images use `loading="lazy"`; videos are muted with `preload="metadata"` and a Play badge; audio shows the AudioLines icon with `role="img"` aria-label "Audio". Title is `{name ?? kind}`. Subtitle is "Upload" or "Generation". aria-label "Select {name ?? kind}" (multi) or "Use {name ?? kind}" (single). `aria-pressed` in multi mode.
  - ElementCard states: hover `bg-white/5`; focus-visible ring; selected = `bg-q-transparent-light-10 ring-1 ring-q-border-focus` plus a lime check badge (`bg-primary`); disabled `opacity-40` (while uploading, or when the kind limit is reached).
- Footer (multi mode only, `:321-343`): caption "{n} selected" or "Choose references". Ghost button "Clear selection". Primary (lime) button "Add {n}", or "Add references" when nothing is selected, disabled when empty or uploading.
- Hidden `<input type=file>` (`:344-355`): `accept` from `FILE_ACCEPT` (`:47-51`): `image/jpeg,image/jpg,image/png,image/webp,image/gif`, `video/mp4`, `audio/wav,audio/x-wav,.wav`. `multiple` in multi mode.

**Controls:**
- Open resets selection, error, query, and tab to "uploads" (`:108-117`).
- The dialog cannot close while uploading.
- Single mode: a click commits and closes. Multi mode: a click toggles selection, and "Add" commits.
- Upload in single mode commits the uploaded asset. In multi mode it adds the asset to the selection.

**Responsive:** grid column breakpoints; modal width clamps to the viewport minus 32px. The body is fixed at 595px tall, so on short screens it depends on `.q-modal` `max-height: calc(100dvh - 64px)` (`styles/quanta/components/modal.css:51-52`).

**Motion:** modal 200ms ease-out in / 150ms ease-in out, with reduced-motion snap (`modal.css`). Element cards use `transition-colors`.

**Template-only deps:** indirect. `onUpload` = `uploadAsset`, which hits `/api/upload`, which reads the key cookie. Uploaded items persist through `useUploads` (localStorage) in `layouts/studio.tsx`.

### 3.4 `settings-dialog.tsx`: `SettingsDialog`, `settingLabel`
**Props** (`:27-32`): `trigger: ReactElement`, `model: ModelEntry`, `values: Record<string,unknown>`, `onChange(key, value)`.
**Reads:** none (the catalog arrives through props).

**States and copy**
- Dialog `size="sm"` (469px). Title "{model.label} settings" (`:47`), description "Applied to the next generation." (`:48`). Header includes the close disc.
- Empty: "This model has no settings." (`:51-54`).
- Enum row (`:94-113`): `<label>` with `settingLabel(key)` and a small Select (`min-w-32`, picker variant, solid, align end). Values are shown raw.
- Range row (`:114-133`): label, current value (tabular), and `Slider` with min/max/step from the catalog.
- Boolean row (`:134-156`): custom `role="switch"` button with `aria-checked`. Track is `bg-q-brand-primary` (lime) when on, `bg-q-transparent-light-15` when off. The knob slides 16px (`transition-transform`). **The switch has no accessible name.** The label `<span>` is not associated with it.
- `settingLabel` (`:74-80`) turns camelCase/snake into sentence case: "aspectRatio" → "Aspect ratio", "generateAudio" → "Generate audio".

**Controls:** each change fires `onChange` immediately (no Save). Esc closes.

### 3.5 `generation-card.tsx`: `GenerationCard`, `CardActions`, `GenerationTile`
**`GenerationCard`** (`:55-126`). Props (`:40-53`): `state?: "ready"|"generating"|"failed"` (default ready), `src?`, `alt?`, `media?: ReactNode`, `title?: ReactNode`, `generatingLabel?` (default "Generating"), `failureLabel?` (default "The generation did not produce previewable media."), `children?`, `className?`, `style?`, `onMouseEnter?`, `onMouseLeave?`. Root has `data-state={state}` and `bg-white/5`.
- **ready:** `media`, or an `<img src alt>` with object-cover. Optional `title` overlay on a bottom gradient (`:88-94`). The feed never passes `title`.
- **generating** (`:95-111`): pulsing `.studio-glow` lime band across the top 45%. Top-left glass pill (`role="status"`) with a lime spinning `Loader2` and the lime label "Generating" (`generatingLabel`). The `animate-spin` spinner is not reduced-motion gated; the glow is.
- **failed** (`:112-122`): centered neutral block (`role="status"`) showing "Generation unavailable" with `failureLabel` underneath. The feed passes `item.failureLabel` = `run.error` (section 5).

**`CardActions`** (`:144-197`). Props: `actions: CardAction[]` (`{id, label, icon: LucideIcon, onSelect?, danger?}` `:129-135`), `max?` (default 3).
- Vertical rail at top-right, `z-[2]`. `opacity-0` → 100 on `group-hover` / `group-focus-within`, `transition-opacity duration-150`.
- If `actions.length > max`, the first `max-1` show inline and the rest go into a ⋯ (EllipsisVertical) menu, trigger aria-label "More actions" (`:175`), `align="end"`. Danger items use the `destructive` variant.
- Inline buttons: 32px glass discs `bg-black/40 hover:bg-black/60`, focus ring, aria-label = action label. Pointer and click events call `stopPropagation`.
- The container is `pointer-events-none` but the buttons are `pointer-events-auto` (GLASS `:137-138`). On touch devices the invisible buttons can still be tapped.

**`GenerationTile`** (`:221-296`). Props (`:200-219`): all card props, plus `generation?: GenerationDetail`, `actions?: CardAction[]`, `openable?`, `openLabel?`, `onFocus?`, `onBlur?` (attached to the open button).
- When `state !== "ready"` it renders a bare `GenerationCard`: no actions, no open trigger.
- When ready: a full-bleed transparent `<button>` (`z-[1]`, focus-visible inset ring) is the trigger for `GenerationDetailModal`. aria-label is `openLabel`, else "Open generation: {prompt}" or "Open generation" (`:277-281`).
- An action with `id === "download"` and no `onSelect` is wired to `downloadMedia(generation.src ?? src)` (`:253-258`).

### 3.6 `generation-detail.tsx`: `GenerationDetailModal`
**Props** (`:36-42`): `trigger`, `generation: GenerationDetail` (`{src, mediaType?, poster?, aspectRatio? (unused), prompt?, model?, createdAt?, settings?}` `:19-28`), `rows?: GenerationDetailRow[]`, `actions?: ReactNode` (replaces the default footer).
**Reads:** none. Uses `downloadMedia`, `navigator.clipboard`, `navigator.share`.

**Layout and states** (raw Base UI Dialog, not the q-modal skin):
- Backdrop `fixed inset-0 z-50 bg-black/80`. Popup `fixed inset-0 z-50`, `flex-col`, `md:flex-row`. 150ms fade via tw-animate `data-open/closed`, with no explicit reduced-motion gate (`:85-86`).
- sr-only title "Generation", sr-only description `{prompt ?? "Generated media"}` (`:87-92`).
- Blurred backdrop: the media still (`poster` for video), `opacity-40 blur-3xl scale-110` (`:93-100`). Feed videos have no poster, so a video gets no blurred backdrop.
- Stage `p-4 md:p-8`. Image is `object-contain rounded-xl`. Video has `controls autoPlay loop playsInline`; autoplay is not gated by reduced motion (`:101-119`).
- Aside: full width with `border-t` on mobile; `md:` 360px floating frosted panel (`bg-background/70 backdrop-blur-xl`, `md:m-4 md:rounded-2xl`) (`:120`).
  - Header "Generation" plus a ghost icon button with aria-label "Close" (`:121-130`).
  - "Prompt" (uppercase caption) and the prompt text, `max-h-40` scroll (`:131-140`).
  - Detail `<dl>` from `defaultRows` (`:48-67`): "Model", "Type" ("Video" or "Image"), "Created" (`toLocaleString()`), then each setting key with `_` → space and CSS `capitalize`. A camelCase key such as `aspectRatio` therefore renders as "AspectRatio".
  - Footer (`:155-198`):
    - "Download" (primary, **lime**, `flex-1`, Download icon).
    - Copy icon button, aria-label "Copy prompt", only when a prompt exists. It turns `text-primary` for 1200ms after copying (`:170-176`).
    - Share icon button, aria-label "Share": `navigator.share({text, url})`, else copies the URL silently.
- Esc and backdrop click close.

### 3.7 `user-generations.tsx`: `UserGenerations`
- Props (`:15-23`): `items: GalleryItem[]`, `title?: ReactNode`, `emptyState: ScreenEmptyStateContent`, `hasMore?`, `loadingMore?`, `onLoadMore?(): void|Promise<unknown>`, `onDelete?(item)`.
- Renders `<JustifiedGallery grouped={false} {...props} />` (`:25-27`). The studio passes `items`, `title`, `onDelete` and `emptyState`. **It never passes `hasMore`, `loadingMore` or `onLoadMore`**, so infinite scroll is dormant.

### 3.8 `gallery/*` (private internals; full mechanics in section 10)
- **`justified-gallery.tsx` `JustifiedGallery`** (`:33-114`). Props `:22-31`; `title` defaults to "Your generations" (`:35`).
  - Header: `<h1>` title (headline-sm-semibold). A "Loading" indicator (spinning Loader2) appears only when `loadingMore` (`:60-64`). `DensityControl` sits at the right (`:66`).
  - Viewport: `overflow-y-auto overscroll-behavior:contain` (`:69-72`).
  - Empty (`itemCount===0`): `<ScreenEmptyState className="h-full" {...emptyState}/>` (`:73-74`).
  - Otherwise: a sizer of `layout.totalHeight` with only the windowed rows rendered (`:76-110`). Header rows (grouped mode only) render a `<h3>` label; they are never used here.
  - There is no loading or skeleton state for the initial history load.
- **`gallery-tile.tsx` `GalleryTile`** (memo, `:118-231`). Props `:21-28`: `item, rect, top, tier, reducedMotion, onDelete?`.
  - generating: `.studio-tile` wrapping `GenerationTile state="generating"` (`:129-135`).
  - failed: `.studio-tile group` wrapping `GenerationTile state="failed" failureLabel={item.failureLabel}`, plus a Trash2 button with aria-label "Remove" (`:136-156`). The button is visible **on hover only** (`opacity-0 group-hover:opacity-100`, no `focus-within`), so keyboard focus is invisible.
  - ready image: `StillMedia`, a `bg-card` placeholder under an `<img loading={tier==="full"?"eager":"lazy"} decoding="async">` (`:42-55`).
  - ready video: `HoverVideo` (`:58-116`). Shows the poster if `item.src` is non-empty; the feed always passes `""`, so the `<video preload="metadata">` first frame is shown. It plays muted and looping on hover or focus, unless `reducedMotion`, and resets to 0 on leave.
  - Actions (`:170-203`):
    - "Download" (auto-wired).
    - "Copy prompt": `clipboard.writeText(prompt)`, no feedback.
    - "Share": `navigator.share({text: prompt, url})` or a silent copy of "{prompt}\n{url}".
    - "Delete" (danger) when `onDelete` is set; it calls `onDelete(item)` → `runs.remove(item.runId)` with **no confirmation**.
    - With 4 actions and `max` 3, Download and Copy show inline; Share and Delete sit in the ⋯ menu.
  - `openLabel` "Open generation: {prompt}". `GenerationDetail` is built from the item, with `src` = videoSrc for video, `model` = `modelLabel`, plus `createdAt` and `settings` (`:207-226`).
- **`density-control.tsx` `DensityControl`** (`:10-33`): props `value: number`, `onChange(level)`. A `w-32` row with a Grid3x3 icon and a Slider; `title="Tile size"`, slider aria-label "Tile density", range 0..4 step 1. Not persisted.
- **`use-reduced-motion.ts`** (`:5-15`): `useReducedMotion(): boolean` via `matchMedia("(prefers-reduced-motion: reduce)")` with a change listener. SSR-safe (initial `false`). Used only to gate hover video playback.
- **`gallery-types.ts`**: `GalleryItem` (`:5-25`) = `{id, runId, projectId?, createdAt, kind: "image"|"video", status: "ready"|"generating"|"failed", src, videoSrc?, width, height, failureLabel?, prompt, alt, modelLabel, settings, groupId, groupLabel}`. `LoadTier = "full"|"near"|"far"` (`:28`); `"far"` is never produced.

### 3.9 `screen-empty-state.tsx`: `ScreenEmptyState`
- Props (`:6-15`): `images: readonly [string,string,string]`, `title: ReactNode`, `description: ReactNode`, `action?: ReactNode`, `className?`.
- Visuals (`:28-70`): a 24px white-8% dot grid masked by an ellipse (`:35-45`). A three-frame composition, `aria-hidden`, `max-w-[444px] h-[105px]`: side frames 38% wide × 85px at opacity 70 with a black/30 overlay; center frame 41.5% × 105px with a glass inset shadow (`:47-59`). Then `<h2>` title (headline-sm-semibold), description (`text-sm text-muted-foreground`), and an optional action.
- Used only by `JustifiedGallery` (the studio passes no `action`). Studio copy: title "No generations yet" (All) or "No generations in {project name}"; description "Describe an idea below, then generate the first result." (`layouts/studio.tsx:230-238`).

### 3.10 `hero-composition.tsx`: `HeroComposition`
- Props (`:3-6`): `images: readonly [string,string,string]`, `alt?`.
- `role="img"` with aria-label = `alt` when one is given. The studio passes "Recent Studio outputs" (`layouts/studio.tsx:142`).
- Geometry (`:16-30`): container `h-[106px] w-[372px] max-w-[calc(100vw-3rem)]`. Side frames 42% × 88px (≈156×88, about 16:9) at opacity 80 with black/20 overlays, using `images[0]` and `images[2]`. Center frame 50.25% × 106px (≈187×106, about 16:9) with a border and shadow, using `images[1]`, `z-[1]`.
- No states, controls or motion. At narrow widths the container shrinks horizontally but the height stays fixed, so the frame ratios change.

### 3.11 `template-picker.tsx`: `TEMPLATES`, `TemplateCard`, `ExamplePresets`
- `TemplateItem` (`:32-46`): `{id, title, subtitle, category (unused by any UI), kind: "image"|"video", images: [string,string,string], icon: LucideIcon, prompt, modelId?, settings?}`.
- **`TEMPLATES` (mock, `:49-94`)**. Each entry is id / title / subtitle / category / kind / icon / images, then the prompt:
  1. `product-hero` / "Product hero shot" / "Studio lighting, larger than life" / commercial / video / BatteryFull / thumbs 1,2,3. Prompt: "Cinematic product hero shot on a dark studio backdrop, slow orbit, soft rim light, dust particles catching the light."
  2. `ugc-review` / "UGC review" / "Handheld, creator-style" / ugc / video / WandSparkles / thumbs 2,4,1. Prompt: "Handheld creator-style review filmed on a phone in a sunny kitchen, natural light, casual framing, subtle camera shake."
  3. `editorial-still` / "Editorial still" / "Magazine-grade key visual" / commercial / image / Calendar / thumbs 3,1,2. Prompt: "Editorial fashion still, 85mm, shallow depth of field, muted palette, single hard light from the left."
  4. `lifestyle-home` / "Lifestyle at home" / "Warm three-shot story" / ugc / image / House / thumbs 4,3,1. Prompt: "Warm lifestyle photo of a couple at home in the evening, lamp light, film grain, candid moment."
  - None of them sets `modelId` or `settings`.
- **`TemplateCard`** (`:134-206`). Props: `template`, `variant?: "single"|"triptych"` (default single), `onTry(t)`, `tryLabel?` (default "Try").
  - The root is a `div role="button" tabIndex=0` with aria-label "Use template: {title}". Enter or Space on the card itself fires `onTry` (`:140-146`). Click fires `onTry`.
  - Image area is `h-60` (240px). Single variant shows only `images[0]`, alt `{title}`, in one rounded frame (about 424×240 ≈ 16:9 at the 900px 2-column layout). Triptych variant shows three frames, alt "{title} — shot {n}" (each ≈137×240 ≈ 9:16). The studio only uses single, so `images[1..2]` are unused.
  - Footer: `GradientBadge` (36px; hue seeded from `id` via `gradientFromSeed` `:96-102`), title, subtitle, and a "Try" `Button size="sm"` **default variant (lime)** with `rounded-full` (`:193-202`). This conflicts with the "lime reserved for the CTA" invariant.
  - States: hover `-translate-y-0.5` with `bg-white/8` (200ms transform and background); `motion-reduce:hover:translate-y-0`; focus-visible ring.
- **`ExamplePresets`** (`:216-236`). Props: `items`, `onUse`, `tryLabel?`, `className?` (default `"w-full max-w-[900px]"`). Grid `grid-cols-1 sm:grid-cols-2 gap-5`.

### 3.12 `my-projects.tsx`: `MyProjects`, `ProjectCard`
- `MyProjectsProject` (`:12-18`): `{id, name, generationCount, cover?, updatedAt?}`.
- `MyProjectsProps` (`:20-27`): `projects`, `generations: GalleryItem[]`, `onCreateProject(name): void|Promise<void>`, `onOpenAllGenerations()`, `onOpenProject(project)`, `className?`.
- Previews (`:112-124`): only ready items with a non-empty `src`, which **excludes every video**. Up to 5 per project and 5 overall.
- Grid `repeat(auto-fit, minmax(min(100%,320px),1fr)) gap-6` (`:129`). Cards are `h-[248px] rounded-2xl border bg-card p-3` with a shadow (`CARD` `:70-71`).
  - "New project" card (`:133-149`): Plus disc and "New project". Hover `bg-white/8`, focus ring. Opens `ProjectCreateModal`.
  - "All Generations" card (`:150-155`): subtitle "{generations.length} generations". `generations.length` counts tiles (images split per URL, plus generating and failed tiles), not runs.
  - Project cards (`:156-167`): title is the project name; subtitle "{generationCount} generations  •  Private" (two spaces on each side of the bullet; `generationCount` = distinct runIds). "Private" is scaffold copy. There is no singular "generation".
- `ProjectCard` (`:73-101`): `<h3>` title and subtitle above a mosaic (1 large + 2×2 80px cells; empty cells `bg-white/6`) (`Mosaic` `:47-68`). A full-card transparent button with aria-label "Open {title}" and an inset focus ring. The card has no hover state; only the New-project card does.

### 3.13 `project-dialogs.tsx`: `ProjectCreateModal`, `ProjectActions`
- **`NameDialog`** (internal, `:27-92`). Props: `trigger?`, `title`, `description`, `initial?`, `submitLabel`, `onSubmit(name)`, `open?`, `onOpenChange?`.
  - Dialog `size="xs"` (349px) with the header close disc and a description paragraph.
  - `Input` with placeholder and aria-label "Project name", `maxLength=80`, autoFocus.
  - Submit is disabled when busy or blank.
  - The name is trimmed. After a successful submit the dialog closes and **resets `name` to ""** (`:55`). Because `useState(initial)` (`:46`) only seeds once, a second Rename opens with an empty field.
- **`ProjectCreateModal`** (`:94-113`). Props `trigger`, `onCreate`. Copy: title "New project", description "Group generations under one name.", submit "Create".
- **`ProjectActions`** (`:116-188`). Props `projectName`, `onRename(name)`, `onDelete()`, `className?`.
  - Trigger is a `q-close q-close-sm` EllipsisVertical button with aria-label "Actions for {projectName}".
  - Menu items: "Rename" (Pencil); "Delete" (Trash2, destructive).
  - Rename dialog: title "Rename project", description "Only the name changes; generations stay linked.", submit "Rename".
  - Delete confirm (`size="xs"`): title "Delete “{projectName}”?" (curly quotes), body "The project goes away. Its generations stay in All Generations.", buttons "Cancel" (outline) and "Delete" (destructive). There is no busy state on Delete.
  - Trigger and content `stopPropagation` so a click does not select the row.

### 3.14 `sidebar.tsx`: `StudioSidebar`, `StudioView`
- `StudioView` (`:29-30`): `{kind:"home"} | {kind:"all"} | {kind:"project"; projectId: string}`.
- Props (`:32-45`): `title`, `logo?: ReactNode`, `view`, `onViewChange`, `projects: MyProjectsProject[]`, `onCreateProject(name)`, `onRenameProject(id,name)`, `onDeleteProject(id)`, `collapsed`, `onCollapsedChange`, `keyConfigured`, `onOpenKey`.
- Shell (`:133-138`): `<aside>` with `m-2.5 h-[calc(100dvh-20px)] rounded-xl border bg-sidebar p-2`. Width `w-64` (256px) or `w-14` (56px) when collapsed, `transition-[width] duration-200` (not reduced-motion gated).
- Header (`:139-165`): a 24px lime (`bg-primary`) logo tile showing `logo` or `title[0].toUpperCase()` ("S"). Title text "Studio" when expanded. Collapse toggle with aria-label "Collapse sidebar" or "Expand sidebar" (PanelLeftClose / PanelLeftOpen icons).
- Nav (`:167-253`):
  - "Home" row (IconTile House, blue) and "All Generations" row (IconTile Images, purple).
  - "Projects" section label (uppercase caption, hidden when collapsed) with a "+" button, aria-label "New project", which opens `ProjectCreateModal`.
  - Project rows: 24px thumbnail (`cover`, else the uppercase initial), name, meta = `generationCount`. On hover, focus-within or open popup, the meta is swapped for the `ProjectActions` ⋯ (`:89-104`).
  - Empty, when not collapsed: "No projects yet." (`:247-251`).
- Footer key row (`:255-274`): KeyRound tile with a status dot (lime `bg-primary` when configured, `bg-destructive` red when not). Title "API key saved" or "Connect API key"; aria-label "Manage API key" or "Connect API key". Click calls `onOpenKey`. **Template-only.**
- `Row` (`:47-116`):
  - Selected: `bg-white/8 text-foreground` plus `aria-current="page"` (neutral, not lime).
  - Hover: `bg-white/5`.
  - Focus-visible ring on the overlay button.
  - Collapsed: centered, string titles get a Tooltip (side right). Tooltip uses no Provider; the Base UI default delay applies.
- **Responsive:** no breakpoints. The sidebar never auto-collapses on narrow screens; collapse is manual and not persisted (state lives in `layouts/studio.tsx:272`).

### 3.15 `icon-tile.tsx`: `IconTile`
- Props (`:27-30`): `ComponentPropsWithRef<"span"> & { as: LucideIcon; gradient?: IconTileGradient | string }`.
- Gradients (`:13-25`): blue, teal, purple, pink, orange, green, red, indigo. Each is a 135° linear gradient. A custom gradient string is also accepted.
- With a gradient: white glyph with `fill=currentColor`, border white/20, inner highlight. Without: neutral `bg-white/5 text-muted-foreground`. 24px tile, 14px glyph. No states.

### 3.16 `key-dialog.tsx`: `KeyDialog` (TEMPLATE-ONLY, remove)
- Props (`:26-36`): `open`, `onOpenChange`, `configured: boolean`, `onChange(configured)`.
- Calls `savePlatformCredentials({api_key})` and `clearPlatformCredentials()` (server actions, `generation/actions.ts:19-35`).
- Copy:
  - Title "Connect API key" or "Manage API key", with a KeyRound icon.
  - Body: "Paste the API key copied from [open.higgsfield.ai](https://open.higgsfield.ai/api-keys). Paste it as-is. It is stored in an HTTP-only cookie in this browser and used by the server for API requests." (`:82-94`).
  - Password input with placeholder "Paste your API key" and aria-label "Higgsfield API key".
  - Error text: the server message ("Enter an API key", "Paste the API key exactly as copied from open.higgsfield.ai") or the fallback "Could not save the key" (`:52`).
  - Footer caption, configured only: "Remove API key" link-button.
  - Submit: "Connect API key" or "Replace API key", disabled when busy or empty.
- The key is not verified against the platform on save. An invalid key surfaces later as a dock error or a failed tile.

### 3.17 `no-models.tsx`: `NoModels`
- Developer-only screen (`:6-24`): `<h1>` "No models installed"; "Add one and the dev server picks it up. Every model is one file in `generation/catalog/models/`."; two `select-all` `<pre>` commands: `pnpm dlx shadcn@latest add higgsfield-ai/app-templates/seedance-2.5` and `pnpm dlx shadcn@latest list higgsfield-ai/app-templates`. Replace it or keep it unreachable.

---

## 4. `components/ui/**` (Base UI + Quanta skins)

| File | Exports | Notes and states |
|---|---|---|
| `button.tsx` | `Button`, `buttonVariants` | cva variants (`:9-20`): `default` (lime `bg-primary`, hover /80), `outline`, `secondary`, `ghost`, `destructive` (red tint), `link`. Sizes (`:21-33`): `default` h-8, `xs` h-6, `sm` h-7, `lg` h-9, `icon` 32, `icon-xs` 24, `icon-sm` 28, `icon-lg` 36. Focus ring-3 ring/50; `active:translate-y-px`; disabled opacity 50. Contains shadcn `dark:` classes (`:12,16,18`); harmless because `<html class="dark">` is pinned, but AGENTS.md says "no `dark:` classes". |
| `dialog.tsx` | `Dialog`, `DialogTrigger`, `DialogPortal`, `DialogClose`, `DialogOverlay`, `DialogContent`, `DialogCloseButton`, `DialogHeader`, `DialogSpacer`, `DialogWorkspace`, `DialogBody`, `DialogFooter`, `DialogTitle`, `DialogDescription` | `DialogContent` `size: xs 349 / sm 469 / md 640 (default) / lg 708 / xl 946 / 2xl 1392px` (`:29-38,69-95`); focuses the card itself (`initialFocus` `:86`). `DialogCloseButton` aria-label "Close" (`:106`). `DialogHeader` (`:119-143`) has a 40px row with an auto close disc (`showCloseButton` default true) and lifts `DialogDescription` below the row. `DialogBody` auto-wraps content in a `DialogWorkspace` (`:180-208`). `DialogFooter` takes `caption?`, `full?` (`:211-235`). Skin: `q-modal-*` (glass `#23262abf` + 40px blur, radius 24px, backdrop scrim `#00000080`). |
| `dropdown-menu.tsx` | `DropdownMenu`, `…Portal`, `…Trigger`, `…Content`, `…Group`, `…Label`, `…Item` (`variant: default\|destructive`), `…CheckboxItem`, `…RadioGroup`, `…RadioItem`, `…Separator`, `…Shortcut`, `…Sub`, `…SubTrigger`, `…SubContent` | Content defaults to align start, side bottom, offset 4; positioner `z-q-dropdown` (10) (`:20-52`). Item `q-menu-item`, destructive `q-menu-item-danger` (`:75-97`). Sub/Checkbox/Radio parts keep stock shadcn classes (accent focus). |
| `input.tsx` | `Input` | Props add `start?`, `end?` (20px affixes), `invalid?` (→ `q-field-control-invalid` + `aria-invalid`) (`:9-41`). 40px white-5% field with lime focus ring. |
| `label.tsx` | `Label` | Stock shadcn label. **Unused** in the template. |
| `select.tsx` | `Select` (= Root), `SelectContent`, `SelectGroup`, `SelectGroupLabel`, `SelectIcon`, `SelectItem`, `SelectItemContent`, `SelectItemDescription`, `SelectItemIcon`, `SelectItemIndicator`, `SelectItemText`, `SelectSeparator`, `SelectTrigger`, `SelectValue` | `SelectTrigger` `size: sm 32 / default 40 / lg 48`, `bare` (no field skin; the host owns styling, used for PromptBox pills), `invalid` (`:57-86`). `SelectContent` `surface: "glass"(default)\|"solid"`, `variant: "default"\|"picker"` (44px rows, lime check), collision padding 16, `alignItemWithTrigger=false`, positioner `z-q-dropdown` (`:105-160`). `SelectItem` auto-wraps string children and adds the check indicator. |
| `slider.tsx` | `Slider` | Base UI slider: 4px track `bg-muted`, lime `bg-primary` indicator, 12px white thumb with ring on hover/focus/active, `thumbAlignment="edge"` (`:4-49`). |
| `tabs.tsx` | `Tabs`, `TabsList`, `TabsTrigger`, `TabsContent`, `TabsVariant`, `TabsShape`, `TabsTone` | `variant: underline(default)\|pill\|segmented\|soft`; `shape: rounded\|pill\|icon`; `tone` defaults to `glass` for segmented (`:39-65`). `TabsList` takes `indicator` (sliding, default on) and `fullWidth`. `TabsTrigger` takes `start`, `end`, `iconOnly`; text is width-locked so a bold active state does not shift neighbours (`:92-148`). Motion 200ms ease-out; reduced motion makes it instant (`tabs.css:572-579`). |
| `tooltip.tsx` | `Tooltip`, `TooltipTrigger`, `TooltipContent`, `TooltipProvider` | Provider delay 0 (unused; the sidebar has no provider). Content: `bg-foreground text-background text-xs` with an arrow, positioner `z-50`, tw-animate fade/zoom/slide (not reduced-motion gated) (`:27-63`). |

`cn` is imported from the npm package `"cn"` (`lib/utils.ts:1` re-exports it).

**Stacking note (unverified, check in a browser):** Select and Menu positioners use `z-index:10`, while `.q-modal` uses 40 and its backdrop 30. The Settings dialog's enum Selects and any menu opened inside a q-modal therefore depend on Base UI portal nesting to paint above the modal. The repository states that changing a setting works, but node_modules is not installed, so I could not confirm it.

---

## 5. Run-state matrix: where each state renders and its copy

| Situation | Where it shows | Copy (source) |
|---|---|---|
| Initial history load (IndexedDB) | Nothing. `useRuns.loaded` is unused, so the feed briefly shows the empty state and the hero shows placeholders | n/a |
| Empty feed (All) | `ScreenEmptyState` inside the gallery | "No generations yet" / "Describe an idea below, then generate the first result." (`layouts/studio.tsx:232-237`) |
| Empty project feed | same | "No generations in {project name}" / same description |
| Submitting (await `submitGeneration`) | No visual change. The CTA stays "Generate" until the record is added; a second click is blocked only once `running.length > 0` | n/a |
| Running | Feed tile `state="generating"`; dock CTA shows "Cancel" | "Generating" (`generation-card.tsx:61,107`); "Cancel" (`studio-prompt-box.tsx:645`) |
| Canceling | CTA disabled | "Cancelling…" |
| Canceled (user, success) | Failed tile | "Generation unavailable" / "Canceled" (`use-runs.ts:109`, also `settle` `:137`) |
| Cancel request failed (e.g. the job has left the queue; the platform cancels "Queued requests only", `generation/platform.ts:107`) | Dock error line (`runs.error`) | PlatformError message, e.g. the platform `detail`, or "Platform request failed ({status})" (`generation/platform.ts:179-183`) |
| Completed with media | Ready tile(s): one per image URL, or one video | n/a |
| Completed with no URLs / platform `failed` | Failed tile | `error.detail` / `error.message` / string error, else "The generation did not produce media" (`use-runs.ts:138-139`) |
| NSFW / content filter (`status:"nsfw"`) | Failed tile | "Blocked by the content filter" (`use-runs.ts:135`) |
| Per-request status error | Failed tile | the server error string (`poll.ts:89-100`) |
| Poll deadline (15 min from `createdAt`, set by `use-runs.ts:37`; the default in `poll.ts:8` is 10 min) | Failed tile | "timed out waiting for the platform" (`poll.ts:109`) |
| 3 consecutive poll-round failures (network, missing key, missing `HF_API_BASE_URL`) | Every running tile fails | the thrown message, e.g. "Connect your Higgsfield API key" (`credentials.ts:13`) or "Missing HF_API_BASE_URL" (`actions.ts:88`) |
| Submit throws (invalid key 401, validation, missing env) | Dock error line; **no tile is created** | PlatformError detail / "Platform request failed ({status})"; "Invalid {key}" (`parse-settings.ts`); "No platform map for {model}" / "A prompt is required for {label}." (`to-platform.ts:10-13`) |
| Missing key on Generate | `KeyDialog` opens instead of submitting (`layouts/studio.tsx:389-392`) | KeyDialog copy (3.16) |
| Invalid plane (pre-submit validation in the `prepared` memo) | CTA disabled; message goes to `disabledReason` (unreachable tooltip). It shows in the error line only if media is attached and no required role is missing (`layouts/studio.tsx:492-496`) | `validateMedia` / mapper errors, e.g. "Add a {role} to use {model}.", "{ROLE}: maximum {n} for {model}. Remove extra attachments.", "References are not supported with start/end frames. Change the frame to Reference image, or remove the other references.", "Add at least one image, video, or audio reference.", "Use either frames or references, not both. Remove incompatible attachments.", "Add a start frame before using an end frame.", "Enter a prompt or add a reference.", "Unknown input mode for {model}." (`generation/catalog/media-inputs.ts:40,143-196`, `generation/catalog/mappers.ts:99-127`) |
| Role change rejected | Dock error line (`localError`) | `changeImageRole` errors: "{ROLE} is not supported by this model.", "Only images can be assigned a frame role.", "{ROLE}: maximum {n} attachments."; fallback "Could not change the image role." (`layouts/studio.tsx:479`) |
| Upload errors | AssetLibrary error banner | "Unsupported file type. Use JPEG, PNG, WebP, GIF, MP4, or WAV." (`upload-contract.ts:24`, `route.ts:29`); "Could not prepare reference upload ({status}). Try again." (`upload.ts:15,26`); "Reference upload failed ({status}). Try uploading the file again." (`upload.ts:39`); "Invalid upload response from Higgsfield. Try again." (`upload-contract.ts:41`); route: "Cross-origin upload requests are not allowed." (`:14`), "Invalid upload request." (`:28`), "Connect your Higgsfield API key in the sidebar before uploading." (`:38`), "Uploads are not configured. Set HF_API_BASE_URL on the server." (`:44`), "Higgsfield rejected your API key. Update it in the sidebar and try again." (`:61`), "Higgsfield upload rate limit reached. Wait a moment and try again." (`:66`), "Could not prepare reference upload with Higgsfield. Try again." (`:70`) |
| Load more (pagination) | Gallery header spinner | "Loading". Dormant: the props are not wired |

There is no toast system. Copy, share and download give no feedback outside the lightbox's 1.2s copy tint.

---

## 6. Template-only dependencies to remove

### 6a. BYOK `KeyDialog` and the `api_key` cookie (`generation/credentials.ts`)
Every touchpoint:
- `generation/credentials.ts` (whole file): `PLATFORM_KEY_COOKIE="api_key"` (`:1`), cookie options (httpOnly, `sameSite:lax`, 30 days, `:3-9`), `MissingCredentialsError` "Connect your Higgsfield API key" (`:11-16`), `encodeCredentials` / `decodeCredentials` / `parseCredentialInput` / `toAuthorizationHeader` ("Key {apiKey}") / `requireApiKey` (`:18-59`).
- `generation/actions.ts`: `savePlatformCredentials` (`:19-27`), `clearPlatformCredentials` (`:29-35`), `hasPlatformCredentials` (`:37-39`), `readStoredCredentials` (`:79-82`), and `readCredentials` (`:84-90`). `readCredentials` is used by `submitGeneration` (`:48`), `getGenerationStatuses` (`:59`) and `cancelGeneration` (`:76`). Replace it with a server-held key, e.g. from env, keeping the `baseUrl` check.
- `generation/platform.ts:1,54`: `toAuthorizationHeader` builds the `Authorization: Key …` header. Keep the header and move the helper.
- `app/api/upload/route.ts:5-7,33-40`: reads the cookie; 401 copy "Connect your Higgsfield API key in the sidebar before uploading."; 401/403 copy "Higgsfield rejected your API key. Update it in the sidebar and try again." (`:61`).
- `components/studio/key-dialog.tsx` (whole component).
- `layouts/studio.tsx`:
  - import `KeyDialog` (`:14`) and `hasPlatformCredentials` (`:23`)
  - state `keyOpen` and `keyConfigured` (`:275-276`)
  - effect (`:331-333`)
  - Generate gate (`:389-392`) and its `keyConfigured` dependency (`:401`)
  - sidebar props `keyConfigured` and `onOpenKey` (`:517-518`)
  - `<KeyDialog …/>` (`:547-552`)
- `components/studio/sidebar.tsx`: props `keyConfigured` and `onOpenKey` (`:43-44,129-130`), the `KeyRound` import (`:8`), and the key status row (`:255-274`).
- `tests/credentials.test.mjs` (whole file). `tests/upload.test.mjs` asserts "server credentials" naming (`:38,65,83-98`); review it.
- Docs: `AGENTS.md:26-29,41-43,70-71` ("Missing key (the key dialog opens)"), `components/studio/AGENTS.md:32`, `README.md`.

### 6b. Device cookie (`generation/device.ts`, `proxy.ts`)
- `generation/device.ts`: `DEVICE_COOKIE="hf_device"`, options (httpOnly, 400 days), `mintDeviceId`, `parseDeviceId`, `resolveDeviceId` (`:1-30`).
- `proxy.ts` (Next 16 proxy/middleware) mints the cookie on every page request without one. Its matcher excludes `api`, `_next/static`, `_next/image`, `favicon.ico` and image extensions (`:10-24`).
- **Nothing reads `hf_device`.** No component, action or route references it, so both files can be deleted with no UI impact. If accounts replace it, a proxy can be reintroduced for auth.

### 6c. Browser persistence (IndexedDB + zustand `persist` → localStorage)
- **`lib/studio/kv.ts`**: IndexedDB DB `"studio"`, store `"kv"` (`:7-8`); `open` / `run` (`:10-36`); `memoryKv` fallback for SSR/tests (`:38-48`); `defaultKv()` singleton (`:52-62`). Only consumer: `history.ts`.
- **`lib/studio/history.ts`**: key `"history.v1"` (`:28`), cap 400 records (`:29`); `loadHistory(kv=defaultKv())` (`:31-35`), `saveHistory(records, kv)` (`:37-42`), `isRunRecord` guard (`:44-55`). It also exports `aspectFromSettings` (`:58-63`), which is pure; keep it. Consumer: `use-runs.ts:11-16,52,66`.
- **`generation/stores/browser-storage.ts`**: `browserStorage<T>()`, a per-call localStorage `PersistStorage` with try/catch (`:5-42`). Consumers:
  - `generation/stores/active.ts` → `useActive` `"hf.active.v2"` (surface, model, batch) (`:21-60`). Read by `layouts/studio.tsx:280-282` and `generation/plane.ts`.
  - `generation/stores/settings.ts` → `useSettings` `"hf.settings.v1"` (`byModel`) (`:11-40`). Read by `layouts/studio.tsx:283-284`.
  - `lib/studio/projects.ts` → `useProjects` `"hf.projects.v1"` (`:28-64`). Read by `layouts/studio.tsx:285`; mutations at `:396` (`touch`), `:507` and `:530` (`create`), `:509` (`rename`), `:511` (`remove`).
  - `lib/studio/uploads.ts` → `useUploads` `"hf.uploads.v1"` (max 200, `:31-47`). Read by `layouts/studio.tsx:286`; written by `uploadAsset` (`:54-66`), which is passed as `library.onUpload` at `layouts/studio.tsx:483`. `remove` is unused.
  - `generation/stores/media.ts` → `useImageMedia` / `useVideoMedia` (`"hf.imageMedia.v1"`, `"hf.videoMedia.v1"`) and `generation/stores/prompt.ts` → `useImagePrompt` / `useVideoPrompt` (`"hf.imagePrompt.v1"`, `"hf.videoPrompt.v1"`). These are **dead code**: they are consumed only by `generation/plane.ts` `assemblePlane()`, which nothing imports. The studio keeps prompt and media in `useState` (`layouts/studio.tsx:273-274`).
- **Components touching persistence:** none in `components/studio/*`. All of it flows through `layouts/studio.tsx` (props `projects`, `library.items`, `galleryItems`) and `useRuns`.
- `useActive.batch` / `setBatch` / `MAX_BATCH` (`active.ts:11-39`) are unused by the UI.
- Project and history rows are linked only by `RunRecord.projectId`. Deleting a project leaves its runs with a dangling `projectId`; they stay visible in All.
- Also unused: `@tanstack/react-virtual` (in `package.json`, imported nowhere) and `components/ui/label.tsx`.

---

## 7. Placeholders and scaffold copy to replace before shipping

### 7a. Placeholder assets (`public/presets/placeholder-1..4.svg`, each 600×800 = 3:4 gradient art)
| Reference | Use | Target aspect of the slot |
|---|---|---|
| `layouts/studio.tsx:66-70` `HERO_FALLBACKS` = [`placeholder-2`, `placeholder-1`, `placeholder-3`] | Home hero (`:116,142`) until 3 ready **image** outputs exist; also the `images` of the feed empty state (`:208,231`) | Hero: sides ≈156×88, center ≈187×106 (≈16:9). Empty state: sides ≈169×85 (≈2:1), center ≈184×105 (≈16:9). Supply 16:9 (e.g. 1280×720); object-cover crops |
| `components/studio/template-picker.tsx:25-30` `THUMBS` (1–4) | `TEMPLATES[*].images` | Single card ≈424×240 at 900px (≈16:9; full width at 1 column). Triptych frames ≈137×240 (≈9:16), unused |
| `registry.json:461-478` | Registry file list (template repo only) | n/a |

The 3:4 portrait placeholders do not match any slot they fill. Videos never feed the hero, project covers or mosaics, because video items have `src=""` (`lib/studio/gallery-items.ts:38`) and those consumers filter on `src !== ""`.

### 7b. Mock templates
The four `TEMPLATES` entries are listed in full in 3.11. Replace them with product presets: title, subtitle, prompt, `kind`, and optionally `modelId`/`settings` (the hook exists at `layouts/studio.tsx:416-425`). Use real media at 16:9. The layouts AGENTS.md requires at least 2 presets. `category` is unused.

### 7c. Scaffold and Higgsfield-branded copy (product-facing)
- `app/layout.tsx:28-29`: title "Studio"; description "Generate images and video with Higgsfield models."
- `layouts/studio.tsx:268-269`: `title="Studio"` (sidebar name and the "S" logo initial); `headline="Turn any idea into images and video"`.
- `layouts/studio.tsx:142`: hero alt "Recent Studio outputs".
- `layouts/studio.tsx:159-162`: tabs "Explore" and "My Projects".
- `layouts/studio.tsx:232-237`: "All Generations" (also `:541`), "No generations yet", "No generations in {title}", "Describe an idea below, then generate the first result."
- `layouts/studio.tsx:479`: "Could not change the image role."
- `studio-prompt-box.tsx`: placeholder "Describe the scene you imagine..." (`:515`); "Video"/"Image" modes; "Generate"/"Cancel"/"Cancelling…"; "Add references", "Model", "All settings", "Attached media", "Clear attachments", "Cannot play", slot labels.
- `template-picker.tsx`: "Try" (`:138,219`), "Use template: {title}".
- `my-projects.tsx`: "New project", "All Generations", "{n} generations", "{n} generations  •  Private".
- `project-dialogs.tsx`: all copy in 3.13.
- `sidebar.tsx`: "Home", "All Generations", "Projects", "New project", "No projects yet.", "Collapse sidebar"/"Expand sidebar", key-row copy.
- `generation-card.tsx`: "Generating", "Generation unavailable", "The generation did not produce previewable media.", "More actions", "Open generation: {prompt}" / "Open generation".
- `gallery/gallery-tile.tsx`: "Download", "Copy prompt", "Share", "Delete", "Remove".
- `gallery/justified-gallery.tsx:35`: default title "Your generations"; "Loading".
- `gallery/density-control.tsx`: "Tile size", "Tile density".
- `generation-detail.tsx`: "Generation", "Close", "Prompt", "Model", "Type", "Video"/"Image", "Created", "Download", "Copy prompt", "Share", "Generated media".
- `settings-dialog.tsx`: "{model} settings", "Applied to the next generation.", "This model has no settings.".
- `asset-library.tsx`: all copy in 3.3.
- `key-dialog.tsx`, `no-models.tsx`: remove (developer and BYOK copy).
- Run and error copy in section 5, especially the Higgsfield/sidebar/API-key messages in `route.ts`, `credentials.ts` and `upload-contract.ts`.
- Model icons `/public/model-icons/*.svg` are third-party model brand marks shown in the Model pill. They are not placeholders, but note them.

---

## 8. Design tokens

### 8.1 Semantic tokens, resolved for `data-theme="default-dark"`
Chain: shadcn var (`styles/theme.css:14-45`) → Quanta semantic (`styles/quanta/theme/color.css:146-279`) → primitive (`styles/quanta/primitives/color.css`).

| shadcn var | Quanta semantic | Primitive | Value |
|---|---|---|---|
| `--background` | `background-primary` | `cool-400` | `#131416` |
| `--foreground` | `text-primary` | `grey-050` | `#ffffff` |
| `--card` | `background-secondary` | `cool-300` | `#1c1e21` |
| `--card-foreground` | `text-primary` | `grey-050` | `#ffffff` |
| `--popover` | `background-tertiary` | `cool-200` | `#2a2d32` |
| `--popover-foreground` | `text-primary` | | `#ffffff` |
| `--primary` | `brand-primary` | `lime-500` | **`#d1fe17`** |
| `--primary-foreground` | `button-label-brand` | `grey-550` | `#1a1a1a` |
| `--secondary` | `transparent-light-05` | | `#ffffff0d` |
| `--secondary-foreground` | `text-primary` | | `#ffffff` |
| `--muted` | `transparent-light-05` | | `#ffffff0d` |
| `--muted-foreground` | `text-secondary` | `grey-300` | `#828282` |
| `--accent` | `overlay-hover` | `transparent-light-05` | `#ffffff0d` |
| `--accent-foreground` | `text-primary` | | `#ffffff` |
| `--destructive` | `state-error-fg` | `red-600` | `#fa0019` |
| `--border` | `border-default` | `transparent-light-10` | `#ffffff1a` |
| `--input` | `border-strong` | `transparent-light-20` | `#ffffff33` |
| `--ring` | `border-focus` | `lime-500` | `#d1fe17` |
| `--chart-1..5` | brand primary/blue/pink/orange/cyan | lime-500 / blue-500 / pink-500 / orange-600 / cyan-500 | `#d1fe17` / `#0256fe` / `#ff005b` / `#ff4f03` / `#9ce6f3` |
| `--radius` | `--hf-radius-300` | | `0.75rem` (12px) |
| `--sidebar` | `background-secondary` | `cool-300` | `#1c1e21` |
| `--sidebar-foreground` | `text-primary` | | `#ffffff` |
| `--sidebar-primary` / `-primary-foreground` | brand-primary / button-label-brand | | `#d1fe17` / `#1a1a1a` |
| `--sidebar-accent` / `-accent-foreground` | overlay-hover / text-primary | | `#ffffff0d` / `#ffffff` |
| `--sidebar-border` / `--sidebar-ring` | border-default / border-focus | | `#ffffff1a` / `#d1fe17` |

`styles/theme.css:46` sets `color-scheme: dark`.

Other dark semantic tokens used directly through `q-*` utilities:
- background: `background-secondary-strong` `#23262a`, `background-glass` `#23262abf` (modals, dropdowns), `background-elevated-start/end` `#383e46` / `#18191c`, `background-inverse` `#ffffff`
- text: `text-tertiary` `#626262`, `text-inverse` `#1a1a1a` (Generate label), `text-link` `#d1fe17`, `text-disabled` `#484e56`, `text-danger` `#fa0019`, `text-success` `#2eb844`, `text-warning` `#dfab01`, `text-info` `#5b91fe`, `text-on-overlay-secondary` `#ffffff80`
- icons: `icon-primary` `#ffffff`, `icon-secondary` `#828282`, `icon-tertiary` `#626262`
- border: `border-subtle` `#ffffff0d`, `border-error` `#fa0019`
- overlays: `overlay-scrim` `#00000080`, `overlay-hover` `#ffffff0d`, `overlay-dim-strong` `#ffffff33`
- states: `state-error-bg` `#5c000f`, `state-error-fg-soft` `#ff5462`, `state-success-fg` `#2eb844`, `state-warning-fg` `#dfab01`, `state-info-fg` `#0256fe`
- lime alphas `transparent-lime-05..90` (`#d1fe170d` … `#d1fe17e5`); `-40` and `-15` feed `.studio-glow`.

**Lime check:** `--hf-color-lime-500: #d1fe17` exactly (`styles/quanta/primitives/color.css:82`). The ramp is 100 `#fbffec`, 200 `#f6ffd1`, 300 `#eeffa7`, 400 `#e1fe68`, **500 `#d1fe17`**, 600 `#b2d814`, 700 `#8ead10`, 800 `#6d840c`, 900 `#4f6109`, 1000 `#344006`, 1100 `#1f2603`, 1200 `#111402`. The Generate CTA is **not flat lime**: it lays a 191° gradient of `rgb(255,255,20)` (`#ffff14`, yellow sheen) over solid `rgb(209,254,23)` = `#d1fe17` (`styles/studio/prompt-box.css:355-357`). The same sheen exists as the token `--hf-gradient-special-gloss-lime-angled` (`primitives/gradient.css:16`), but the CSS writes it out literally.

Neutral ramps: `cool-050..400` = `#5c626a, #484e56, #383e46, #2a2d32, #23262a, #1c1e21, #18191c, #131416`. Grey `050..600` = `#fff, #f4f4f4, #cecece, #a8a8a8, #7f7f7f(250), #828282(300), #626262, #565656, #454545, #323232, #2a2a2a, #1a1a1a, #000`; note that 250 is darker than 300.

### 8.2 Fonts
- Loaded in `app/layout.tsx:9-24` through `next/font/google`: Space Grotesk (`--font-space-grotesk`), Inter (`--font-inter`), IBM Plex Mono 400/500 (`--font-ibm-plex-mono`).
- Mapped in `styles/theme.css:9-12`: grotesk-base and primary-base → Space Grotesk; secondary-base → Inter; mono-base → IBM Plex Mono.
- Families (`styles/quanta/theme/typography.css:9-12`): `--hf-type-family-grotesk` (+ sans-serif), `-primary` (+ system-ui, sans-serif), `-secondary` (+ system-ui, sans-serif), `-mono` (+ ui-monospace, monospace).
- `primitives/typography.css:17-20` hand-points primary to "Space Grotesk"; upstream Quanta uses "Inter Display" (comment `:7-13`).
- Tailwind: `font-sans` = Inter (secondary), `font-heading` = Space Grotesk, `font-mono` = Plex Mono (`app/globals.css:11-13`). Body text is Inter.
- Weights (`primitives/typography.css:22-26`): regular 400, medium 500, semi-bold 600, bold 700, black 900.
- Letter spacing (`:28-32`): loose 0.00625rem, none 0, slight −0.025rem, tight −0.075rem, wide 0.0125rem.

### 8.3 Type scale (`styles/quanta/theme/typography.css`)
Base sizes (`:14-28`), in rem: 050 .625 (10px), 100 .75 (12), 200 .875 (14), 300 1 (16), 400 1 (16), 500 1.125 (18), 600 1.25 (20), 700 1.5 (24), 800 1.75 (28), 900 1.875 (30), 1000 2 (32), 1100 2.25 (36), 1200 2.5 (40), 1300 3 (48), 1400 3.5 (56).
- At ≥768px (`:46-71`): 400 → 18, 500 → 20, 600 → 22, 700 → 26, 800 → 30, 900 → 34, 1000 → 36, 1100 → 42, 1200 → 48, 1300 → 56, 1400 → 64.
- At ≥1280px (`:73-94`): 600 → 24, 700 → 28, 800 → 32, 900 → 36, 1000 → 40, 1100 → 48, 1200 → 56, 1300 → 64, 1400 → 72.
- Line heights (`--hf-type-line-height-*`) mirror the same steps.

Utility families (`styles/quanta/tailwind/typography.css`), shown as family / size step / weight / line-height step / letter spacing:
- `text-q-display-{sm,md,lg}-{medium,semi-bold,bold}`: primary, 1100 / 1200 / 1300, tight
- `text-q-headline-{sm,md,lg}-*`: primary, 800 / 900 / 1000, tight
- `text-q-title-{sm,md,lg}-*`: primary, 500 / 600 / 700, slight
- `text-q-accent-{2xs,xs,sm,md,lg,xl,2xl..6xl}-bold`: grotesk bold, 100 / 500 / 600 / 700 / 800 / 900 / 1000..1400, tight
- `text-q-body-{sm,md,lg}-{regular,medium,semi-bold}`: Inter, 200 / 300 / 400; lh 500 / 600 / 600; sm uses loose spacing
- `text-q-label-{xs,sm,md,lg}-*`: Inter, 100 / 200 / 300 / 400
- `text-q-caption-{xs,sm}-{regular,medium,semi-bold}`: Inter, 050 / 100; xs uses wide spacing
- `text-q-mono-{xs,sm,md,lg}-*`: Plex Mono, 050 / 100 / 200 / 300

Used in the template:
- `accent-lg-bold`: hero h1, 28 → 30 → 32px, uppercase
- `headline-sm-semi-bold`: gallery title and empty-state title
- `headline-md-semi-bold`: NoModels
- `body-sm-regular/medium`: prompt field and dialog text
- `caption-sm-regular/medium/semi-bold`: chips, errors, counters
- `caption-xs-semi-bold`: mode labels
- `label-md-semi-bold`: asset empty-state title
- Plain Tailwind sizes (`text-sm`, `text-xs`) also appear in the cards, sidebar and lightbox.

### 8.4 Radii (`primitives/radius.css:9-19`)
`0`, 050 2px, 100 4px, 150 6px, 200 8px (pill), 250 10px, 300 12px (`--radius`, upload tile, CTA), 400 16px (mode button, asset cards), 500 20px (dock surface, active mode), 600 24px (dock, mode rail, modal), full 9999px.

Tailwind `rounded-*` derive from `--radius` (section 2): `rounded-lg` 12, `rounded-xl` 16.8, `rounded-2xl` 21.6px. `rounded-q-*` utilities map 1:1 to the tokens.

### 8.5 Spacing (`primitives/spacing.css:9-27`)
In px: 0, 050 = 2, 100 = 4, 150 = 6, 200 = 8, 250 = 10, 300 = 12, 400 = 16, 500 = 20, 600 = 24, 650 = 24, 700 = 28, 800 = 32, 1000 = 40, 1200 = 48, 1400 = 56, 1600 = 64, 2000 = 80, 2400 = 96. Tailwind utilities are `*-q-*`.

### 8.6 Motion (`primitives/motion.css`)
- Durations (`:9-29`): instant 0, fast 100, normal 200, slow 300, slower 500, plus 80, 120, 130, 150, 160, 180, 220, 260, 350, 400, 700, 800, 1000, 1200, 1300, 1400 ms.
- Easings (`:30-39`): in `(0.4,0,1,1)`, out `(0,0,0.2,1)`, in-out `(0.4,0,0.2,1)`, linear, spring `(0.34,1.56,0.64,1)`, out-expo `(0.16,1,0.3,1)`, emphasized `(0.32,0.72,0,1)`, swift `(0.2,0,0,1)`, in-out-circ `(0.65,0,0.35,1)`, out-quart `(0.22,1,0.36,1)`.
- Tailwind names: `duration-q-*`, `ease-q-*`.

Where motion is used:
- Modal: 200ms ease-out in (opacity, scale 0.96), 150ms ease-in out (`components/modal.css:63-76`).
- Dropdown/Select: 180ms out-expo in (scale 0.94 plus a 4px side offset), 130ms ease-in out (`components/dropdown.css:41-76`).
- Select chevron 180ms out-expo; close disc 150ms ease-out; tabs 200ms ease-out.
- CTA: 140ms `cubic-bezier(0.23,1,0.32,1)` (literal), press 100ms scale 0.98.
- `.studio-glow` 2600ms ease-in-out infinite.
- CardActions fade 150ms; sidebar width 200ms; TemplateCard lift 200ms; Home tab content `animate-in fade-in-0 duration-300` (`layouts/studio.tsx:166`); lightbox fade 150ms; tooltip tw-animate.

Reduced motion:
- Handled: modal, dropdown, select and close (1ms); tabs (instant, no panel animation); CTA (1ms, no transform); `.studio-glow` (static at 0.7); TemplateCard lift (`motion-reduce:`); hover video (no play).
- **Not handled:** `Loader2 animate-spin` (tiles, upload, gallery "Loading"); sidebar width transition; `scrollIntoView({behavior:"smooth"})` (`layouts/studio.tsx:180-183`); Home tab fade; lightbox fade and video `autoPlay`; tooltip animations.

### 8.7 Z-index scale (`primitives/z-index.css:9-16`, Tailwind `z-q-*`)
base 0, dropdown 10, sticky 20, overlay 30, modal 40, popover 50, toast 60, tooltip 70.

Actual usage:
- Select and Menu positioners: `z-q-dropdown` (10)
- `.q-modal-backdrop`: 30
- `.q-modal`: 40
- Tooltip positioner: raw `z-50`
- `GenerationDetailModal` backdrop and popup: raw `z-50`
- Local stacking: tile open button `z-[1]`, CardActions and failed-tile Remove `z-[2]`, dock chip controls `z-10`, hero center frame `z-[1]`, sidebar row overlay `z-[1]` / action `z-[2]`

### 8.8 Shadows, opacity, icons, breakpoints
- Shadows (`primitives/shadow.css:9-17`): `inset-tile`, `inset-tile-hairline`, `modal`, `overlay`, `raised`, `raised-sm` (`0 2px 4px 0 #0000001a`), `sheen`, `thumb-soft`, `thumb-strong`. The dock, mode rail and CTA use literal composite shadows (`prompt-box.css:33-35,69-71,361-364`).
- Opacity (`primitives/opacity.css:9-10`): disabled 0.5, muted 0.4.
- Icon sizes (`primitives/icon.css:9-13`): xs 12, sm 16, md 20, lg 24, xl 28.
- Breakpoints: Quanta mobile 20rem, tablet 48rem, desktop 80rem, wide 120rem (`primitives/breakpoint.css:9-12`). Tailwind exposes `q-tablet` / `q-desktop` / `q-wide` (`tailwind/breakpoint.css`). The components actually use Tailwind `sm` 640 / `md` 768 / `lg` 1024 and the prompt-box `max-width:640px` query.
- Component knobs (`styles/quanta/components.css`): `--q-modal-width-{xs..2xl}`, `--q-menu-min/max-width`, `--q-sidebar-width` 14.9375rem (unused by `StudioSidebar`, which uses w-64 / w-14), and others.
- Focus ring utility `q-focus-ring`: `0 0 0 2px var(--background)` plus `0 0 0 4px` lime (`styles/quanta/index.css:40-46`).

---

## 9. `useRuns` contract (`lib/studio/use-runs.ts`)

### 9.1 Types (quoted)
```ts
// lib/studio/history.ts:5-26
export type RunStatus = "running" | "completed" | "failed"

/** One generation request and everything the studio shows about it. */
export interface RunRecord {
  id: string
  /** Platform request id; running rows resume their poll from it after a reload. */
  requestId: string
  surface: Surface            // "image" | "video"
  modelId: string
  modelLabel: string
  prompt: string
  settings: Record<string, unknown>
  /** Owning project, when generated from a project view. */
  projectId?: string
  /** Aspect ratio as width / height, used by the justified feed before media loads. */
  aspect: number
  status: RunStatus
  /** Result URLs: one video or N images. */
  urls: string[]
  error?: string
  createdAt: number
}
```
```ts
// generation/catalog/types.ts:55-61
export type GenerationPlane = {
  model: string
  inputMode?: string
  prompt: { text: string }
  media: Partial<Record<MediaRole, MediaItem[]>>
  settings: Record<string, unknown>
}
// generation/platform.ts:27-39
export type GenerationStatus = {
  status: string; requestId: string
  images?: Array<{ url: string }>; video?: { url: string }; error?: unknown
}
export type StatusResult =
  | { requestId: string; status: GenerationStatus }
  | { requestId: string; error: string }
```

### 9.2 Signature and return value (inferred; no explicit annotation, `use-runs.ts:23,123`)
```ts
export function useRuns(): {
  records: RunRecord[]                 // newest first (submit prepends)
  running: RunRecord[]                 // records.filter(status === "running"), recomputed each render
  loaded: boolean                      // true after loadHistory resolves (unused by the studio)
  error: string | null                 // last submit/cancel error (shown in the dock)
  submit: (plane: GenerationPlane, projectId?: string) => Promise<RunRecord | null>
  cancel: (id: string) => Promise<void>
  remove: (id: string) => void
  setError: React.Dispatch<React.SetStateAction<string | null>>  // unused by the studio
}
```

### 9.3 Behaviour
- **Mount** (`:50-63`): `loadHistory()` → `setRecords(stored)` → `setLoaded(true)` → `watch(record)` for every record with status `running` (resumes polls after a reload). Unmount calls `stopWatching()`, which clears the module-level poller.
- **Persist** (`:65-67`): once loaded, every `records` change calls `saveHistory(records)`. That rewrites the whole array, capped at 400.
- **`submit`** (`:69-101`):
  1. `setError(null)`; `getModel(plane.model)`.
  2. `await submitGeneration(plane)` (server action) → `{requestId, …}`.
  3. Build a `RunRecord`: `id = requestId`, surface/model/label from the catalog, `prompt = plane.prompt.text`, `settings = plane.settings` (the parse happens server-side, so these are the raw client values), `projectId?`, `aspect = aspectFromSettings(settings)`, `status:"running"`, `urls:[]`, `createdAt: Date.now()`.
  4. Prepend it, `watch(record)`, return the record.
  5. On throw: `setError(message)` and return `null`. **No record is created.**
- **`watch`** (`:34-48`): `watchRequest(requestId, {deadline: createdAt + 15 min})`. Resolve → `update(id, settle(status))`. Reject → `update(id, {status:"failed", error: message})`.
- **`settle`** (`:126-141`):
  - `urls = images[].url ∪ video.url`
  - `completed` with urls → `{status:"completed", urls}`
  - `nsfw` → failed "Blocked by the content filter"
  - `canceled` → failed "Canceled"
  - anything else → failed `describeError(error)` (string, or `.detail`, or `.message`) ?? "The generation did not produce media"
- **`cancel(id)`** (`:103-115`): no-op unless the record is `running`. Calls `await cancelGeneration({requestIds:[requestId]})`, then `update(id, {status:"failed", error:"Canceled"})`. On throw, `setError(message)`. The watch keeps running and later settles "canceled" → "Canceled" again. `cancel` closes over `records`.
- **`remove(id)`** (`:117-119`): local filter only. No platform call, no confirmation, and the poll is not stopped.

### 9.4 Collaborators
- **`generation/actions.ts`** (`"use server"`):
  - `submitGeneration(plane)` (`:41-49`): `parseSettings` → `toPlatform` → `client.submit(path, body)` → `QueuedGeneration {status, requestId, statusUrl, cancelUrl}`.
  - `getGenerationStatuses({requestIds})` (`:55-72`): fans out server-side, returns `StatusResult[]`; a per-id error becomes `{requestId, error}`.
  - `cancelGeneration({requestIds})` (`:74-77`): only the first id is used.
- **`generation/poll.ts`** (module singleton):
  - `watchRequest(id, {deadline})` (`:30-52`) dedupes through `inflight`.
  - One `getGenerationStatuses` call per `POLL_INTERVAL_MS = 4000` for all waiting ids (`:7, 65-87`). Server actions are serialized per client, so this batching avoids starving submit.
  - `TERMINAL = completed|failed|nsfw|canceled` (`:5`).
  - `deliver` (`:89-100`) rejects per-request errors; `sweep` (`:104-111`) applies deadlines.
  - `MAX_MISSES = 3` consecutive round failures → `settleAll(reject)` (`:11, 80-82, 113-118`).
  - `stopWatching` (`:57-63`) drops all watches without settling them.
- **`lib/studio/history.ts`**: `loadHistory(kv = defaultKv())`, `saveHistory(records, kv)`. Takes a `Kv` (`{get<T>(key), set(key,value)}`, `kv.ts:2-5`). This is the injection seam for swapping the backend. Also `aspectFromSettings` (`:58-63`): reads `settings.aspect_ratio ?? aspectRatio ?? ratio`, splits on `:`, `x` or `/`, and returns w/h, else 1 (so `"auto"` gives 1).
- **Consumers in `layouts/studio.tsx`**: `runs.records` (`:336,340`), `runs.running` (`:377,411`), `runs.error` (`:494`), `runs.submit` (`:394`), `runs.cancel` (`:411`), `runs.remove` (`:543`). Derived: `runToGalleryItems` / `runToLibraryItems` (`lib/studio/gallery-items.ts:7-53`).

### 9.5 What must survive the MongoDB swap
1. The return shape above, including `submit` resolving to the created `RunRecord` (the caller switches to the All view and touches the project) or `null` with `error` set.
2. Newest-first `records`.
3. `running` derived from status.
4. `id === requestId`.
5. The `settle` copy mapping.
6. Failed and canceled runs stay in `records` and are never filtered out (AGENTS.md:48-50).
7. Resume-on-mount for `running` rows.
8. `loaded`, which should finally drive a loading state.

Today's whole-array `saveHistory` needs to become per-record create, update and delete. `projects.ts` `create` is **synchronous** and its return value is used immediately (`layouts/studio.tsx:507,530`: `projectsStore.create(name).id`). The sidebar and MyProjects `onCreateProject` props already accept `Promise<void>`.

---

## 10. Gallery internals (`components/studio/gallery/*`)

**Pipeline:** `RunRecord[]` → `runToGalleryItems` (`lib/studio/gallery-items.ts:7-40`) → `GalleryItem[]` → `UserGenerations` → `JustifiedGallery` → `useJustifiedGallery` → `JustifiedLayoutEngine` → rows → `GalleryTile`.

**Item mapping:**
- A running or failed run becomes one placeholder item: `id = run.id`, status `generating` or `failed`, `src:""`, `failureLabel = run.error` (`:21-32`).
- A completed run becomes one item per URL: `id = runId` when there is a single URL, else `` `${runId}:${i}` ``. Images get `src = url`; videos get `src = ""` and `videoSrc = url` (`:33-40`).
- Every item carries `groupId:"all"` and `groupLabel:""`.

**Justified algorithm inputs:**
- `width / height` per item. Only the ratio matters: `width = run.aspect`, `height = 1` (`gallery-items.ts:12-13`). `run.aspect` comes from the requested setting at submit time (`aspectFromSettings`), **not from the real media dimensions**. `"auto"` or a missing setting gives 1:1, and media is cropped with `object-cover`.
- `containerWidth` = viewport `clientWidth`, measured with a ResizeObserver (`use-justified-gallery.ts:106-117`).
- `targetRowHeight` = `DENSITY_ROW_HEIGHTS[density]`.
- `gap = 6` (`:27`), used both between tiles and between rows.
- `grouped = false` from `UserGenerations`.
- Engine defaults that the hook does not override (`justified-engine.ts:86-94`): `headerHeight 44`, `groupGap 20` (both only matter when grouped), `maxRowHeight 460`.

**Algorithm** (`justified-engine.ts:171-275`):
- Greedily add items while `targetRowHeight·Σaspect + gap·(n−1) < containerWidth`. When the row reaches or exceeds the width, flush it with `h = (W − gap·(n−1)) / Σaspect`, set each tile width to `h·aspect`, and advance x by `w + gap`.
- The trailing partial row is never taller than the target and is left-aligned.
- Every row is clamped to `maxRowHeight` (460, `:221-227`). At density index 4 (580px target), rows taller than 460 are clamped and **no longer fill the width** (a right-edge gap).
- Not grouped: one continuous group, no header rows.
- Output: `Layout {rows: LayoutRow[] (y, height, key "r-{n}", tiles: TileRect{index,item,x,width,height}), totalHeight, itemCount}`, plus per-item `itemTops` / `itemHeights` for anchoring.

**Density control:** `DENSITY_ROW_HEIGHTS = [220, 280, 360, 460, 580]`, `DEFAULT_DENSITY = 1` (280px) (`use-justified-gallery.ts:24-25`). The slider runs 0..4, step 1; left is smaller and denser. Held in local state; not persisted.

**Virtualization by rows:**
- The engine lays out **all** items with an O(n) pass on each change of items, width or density (the engine `useMemo` at `use-justified-gallery.ts:74-85`).
- `getWindow(scrollTop, viewportHeight, OVERSCAN_PX=400)` binary-searches rows for `[startRow, endRow)` (`justified-engine.ts:286-328`).
- Scroll is passive and rAF-throttled. `readRange` only `setRange`s when the row window actually changes (`use-justified-gallery.ts:91-103, 169-199`).
- Only rows in the window render. Each tile is absolutely positioned (`left: x, top: row.y, width, height`) inside a sizer of `totalHeight`, keyed by `item.id` (`justified-gallery.tsx:76-108`).
- Tile load tier: a row intersecting the true viewport → `"full"` (img `loading="eager"`); an overscan row → `"near"` (`lazy`). `"far"` is defined but unused (`justified-gallery.tsx:94-96`).
- `.studio-tile` adds `content-visibility:auto`. `GalleryTile` is `memo`.

**Rule about loaded media (AGENTS.md:48-50; `gallery-tile.tsx:16-20`; `use-justified-gallery.ts:167-168`):** "Never unmount loaded media because scroll velocity is high; never filter out failed terminal runs the user already saw generating." In code this means:
- The only unmounting is positional windowing with a 400px overscan.
- There is no velocity-based swap to placeholders or skeletons, and none should be added.
- Mounted `<img>` / `<video>` elements stay mounted for as long as their row is in the window, so finished generations never flash back to placeholders.
- A `bg-card` placeholder sits under each image. It does not replace the image.
- Failed and generating items are always included; the studio never filters statuses.

**Scroll anchoring:** every scroll frame stores `engine.findAnchor(scrollTop)` = `{index, offset}` (the last item whose top ≤ scrollTop, `justified-engine.ts:347-364`). After any re-layout (width, density or items), `useLayoutEffect` restores `scrollTop = getItemTop(index) + offset` when `index > 0` (`use-justified-gallery.ts:121-131`). `setDensity` refreshes the anchor first (`:201-211`). Caveat: the anchor is by **index**, so a new run prepended at index 0 shifts the anchored content by one item when the user is scrolled down.

**Infinite scroll (dormant):** `hasMore`, `loadingMore`, `onLoadMore`.
- Triggers when `scrollTop + clientHeight ≥ totalHeight − 1200` (`INFINITE_MARGIN`, `:31`) on scroll, and also in an effect so short pages keep filling the viewport (`:155-165`).
- Deduped by `feedEdgeKey = "{length}:{firstId}:{lastId}"`, so an empty or duplicate page cannot drain the cursor, and guarded by a `loadingRef` (`:134-151`).
- `onLoadMore` errors are swallowed; the owner must surface them.
- This is the natural hook for MongoDB cursor pagination. Appending older items does not disturb the anchor.

**Hover video:** muted, looping, `playsInline`. Plays on `mouseenter` / `focus` of the open button and pauses and rewinds on leave or blur. Disabled under reduced motion. With no poster, the video stays visible and shows its first frame through `preload="metadata"` (`gallery-tile.tsx:58-116`).

---

## 11. Studio layout (`layouts/studio.tsx`)

### 11.1 Tree
```
<div class="flex h-dvh overflow-hidden bg-background">          :500
  <StudioSidebar …/>                                              :501-519
  <main class="relative flex min-w-0 flex-1 flex-col">            :520
    view.kind === "home"
      ? <HomeState>                                               :95-193
          dots layer (HERO_DOTS 14px, mask HERO_DOTS_MASK, h-600) :120-129
          glow layer (HERO_GLOW, neutral grey radial, h-600)      :130-134
          <div px-6 pt-16 pb-16 gap-12>
            <div ref=promptRef>
              <HeroComposition images=heroImages(items) alt="Recent Studio outputs"/>
              <h1 text-q-accent-lg-bold uppercase max-w-640>{headline}</h1>
              <StudioPromptBox {...dock}/>   (plain skin, w-830 max-w-full, inline)
            <div max-w-900>
              <Tabs segmented pill> Explore | My Projects             :151-165
              <div key=tab animate-in fade-in-0 300ms>
                tab=projects → <MyProjects projects generations=items …/>
                else         → <ExamplePresets items=TEMPLATES onUse=…/> (+ smooth scroll to dock)
      : <FeedState>                                               :195-258
          <div px-4 pt-4 style.paddingBottom=dockHeight+24>
            <UserGenerations items=visibleItems title emptyState onDelete/>
          <div absolute bottom-0 inset-x-0 pt-24 pb-4 px-4 pointer-events-none
               bg: linear-gradient(transparent → var(--background))>  (the bottom scrim)
            <div ref=dockRef max-w-900> <StudioPromptBox {...dock} skin="glass" className="pointer-events-auto w-full"/>
  <KeyDialog …/>                                                  :547-552
```
- Home scrolls as one page (`overflow-y-auto` on the HomeState root).
- The feed view is `overflow-hidden`; the gallery owns its scroll.
- The dock height is measured with a ResizeObserver (initial 180) so the feed padding clears the floating dock (`:209-219`).
- Full-bleed shell with no `max-w` (as AGENTS.md requires). No app header.

### 11.2 View state
- `StudioView = {kind:"home"} | {kind:"all"} | {kind:"project"; projectId}` (`sidebar.tsx:29-30`), held in `useState` (`:271`).
- Transitions:
  - Sidebar rows call `onViewChange`.
  - Creating a project (sidebar or MyProjects) goes to `{project, id}` (`:506-508, 527-532`).
  - Deleting the current project goes to `{all}` (`:510-514`).
  - A successful submit from Home goes to `{all}` (`:395`).
  - The All Generations card goes to `{all}`; a project card goes to `{project}`.
- Project view filters `galleryItems` by `projectId` through the `byProject` map (`:343-350, 372-375`); there is no second history. Title is `selectedProject?.name ?? "All Generations"` (`:541`).
- The HomeState tab (`explore` | `projects`) is local and resets to Explore each time Home mounts (`:114`).
- Sidebar `collapsed` is local `useState` (`:272`).

### 11.3 Derived data
- `galleryItems = records.flatMap(runToGalleryItems)` (`:335-338`).
- `libraryItems = [...uploads, ...records.flatMap(runToLibraryItems)]` (`:339-342`). Generation items get `name = prompt.slice(0,60)`; an empty prompt renders a blank title in the library because `?? kind` does not catch `""`.
- `projects: MyProjectsProject[]` (`:351-367`): `generationCount` = distinct runIds; `cover` = the first ready image item.
- `heroImages(items)` (`:83-93`): the first 3 ready items with a non-empty `src` (images only), padded with `HERO_FALLBACKS`. Feed empty states use `heroImages(galleryItems)` from all items, not the project's (`:208, 540`).

### 11.4 Dock state flow (one controlled state shared by Home and Feed)
- **Local `useState`:** `prompt` (`:273`), `media: MediaItem[]` (`:274`), `canceling` (`:277`), `localError` (`:278`). Prompt and media are **not** cleared after a submit and **not** persisted.
- **Persisted stores:**
  - `useActive` gives `surface` and `model` (`:280-282`); `setModel(id)` also derives the surface.
  - `useSettings.byModel[model.id]` gives `settings` (`:283-284, 307-310`).
- **Derived:**
  - `surfaces` = `["video","image"]` filtered by availability (`:289-295`).
  - `models` = models for the current surface (`:296-299`).
  - `model` = `getModel(modelId)`, falling back to the first model of the surface (`:300-306`).
  - `inputMode = inferInputMode(model, media)` (`:311`).
- **`prepared`** (`:312-329`): builds a `GenerationPlane {model, inputMode, prompt:{text: trimmed}, media: groupMedia(media), settings: parseSettings(model, settings)}` and dry-runs `toPlatform(plane)` to validate. Result is `{plane, error:null}` or `{plane:null, error}`.
- **Gates:**
  - `generating = running.length > 0` (`:377`). Only one batch in flight is possible from the UI because the CTA turns into Cancel.
  - `canGenerate = plane !== null && (prompt.trim() || media.length)` (`:378-379`).
  - `missingRequiredInput` (`:380-384`) only suppresses the inline error.
- **Handlers:**
  - `handleGenerate` (`:386-406`): guard → clear localError → key gate (template-only) → `runs.submit(plane, projectId?)` → switch Home to All and `projectsStore.touch(projectId)`.
  - `handleCancel` (`:408-414`): cancels **all** running runs in parallel; `canceling` stays true until all settle.
  - `handleUseTemplate` (`:416-425`): sets the prompt. If the template has a valid `modelId`, switches model and applies its settings. Otherwise, if the template kind differs from the current surface, switches to that surface's first model. Media is not cleared.
  - `changeSurface` (`:428-431`): selects the first model of that surface.
- **`dock: DockProps`** (`:433-497`), where `DockProps` = `StudioPromptBoxProps` minus `className` and `skin` (`:78-81`):
  - `onModelChange` clears `localError`.
  - `onSettingChange(key, value)` calls `setSetting(model.id, {[key]: value})`.
  - `onMediaAdd(role, selection)`: maps `AssetSelection` → `MediaItem`; `"auto"` goes to `mergeReferences(media, selected, roles)`, a specific role goes to `mergeMedia(media, role, …, roles[role])`. These helpers throw on limit violations. The throw propagates to `AssetLibraryModal.commit`, which shows it in the modal.
  - `onMediaRemove` filters by id.
  - `onMediaRoleChange` calls `changeImageRole`; a thrown error sets `localError`.
  - `library = {items: libraryItems, onUpload: uploadAsset}`.
  - `generateDisabled = !canGenerate`; `disabledReason = prepared.error`.
  - `error = localError ?? runs.error ?? (!missingRequiredInput && media.length>0 ? prepared.error : null)`.
- The same `dock` object goes to the inline dock (Home) and the floating dock (Feed). Values survive view switches, but the `StudioPromptBox` instance remounts, so internal UI state such as an open dialog resets.

### 11.5 Presets section
Explore renders all of `TEMPLATES` in `ExamplePresets` with no category filter. `onUse` → `handleUseTemplate` → `scrollIntoView({behavior:"smooth", block:"center"})` on the prompt block (`:176-186`).

---

## 12. Gaps and bugs found while reading (for planning)

1. **Disabled-reason tooltip is unreachable.** The Generate `title` sits on a `pointer-events:none` disabled button (`prompt-box.css:380-384`). The "+" pill's title has the same problem.
2. **Pills have no keyboard focus ring** (`prompt-box.css:177-206`).
3. **The failed-tile "Remove" is hover-only** with no `focus-within` (`gallery-tile.tsx:148`). **The boolean settings switch has no accessible name** (`settings-dialog.tsx:138-154`).
4. The prompt textarea does not auto-grow, and `overflow:hidden` means long prompts are clipped.
5. There is no initial-load state (`loaded` is unused), so the empty state and placeholders flash before history arrives.
6. Submit failures create no tile; they surface only in the dock error line, which clears on the next model change or attach.
7. Rename dialog: the name resets to "" after the first rename (`project-dialogs.tsx:46,55`).
8. MyProjects: the "All Generations" count is a tile count, project cards count runs; "1 generations" is never singularized; the "Private" label is fake.
9. Videos never appear in the hero, project covers or mosaics (`src:""`). The lightbox shows no blurred backdrop for videos, and lightbox video autoplays regardless of reduced motion.
10. Feed tile aspect comes from the requested setting, not the media; `auto` gives a square crop. At density 580, rows are clamped to 460 and do not fill the width.
11. Index-based scroll anchor: prepending a new run while scrolled down shifts the visible content by one tile.
12. Lime used outside the CTA and active states: the TemplateCard "Try" button, the lightbox "Download", the sidebar logo tile, the AssetLibrary "Add" button and selected badge, the boolean switch, and the slider. AGENTS.md reserves lime for the CTA and active states.
13. The sidebar is not responsive (no auto-collapse); at phone width it takes 256 + 20px.
14. Copy, share and download give no feedback (no toast system).
15. Tile delete has no confirmation and is local only.
16. Dead code and unused pieces: `generation/plane.ts` with the media and prompt stores, `useActive.batch`, `MAX_INLINE_SETTINGS`, `TemplateItem.category`, triptych images, `LoadTier "far"`, `GenerationDetail.aspectRatio`, `components/ui/label.tsx`, the `@tanstack/react-virtual` dependency, `useUploads.remove`, `useRuns.setError`.
17. Possible z-index conflict: Select/Menu positioners (10) vs `.q-modal` (40). Verify in a browser.
