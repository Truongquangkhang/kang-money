# Theme — Design Tokens

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-27
**Assigned To:** Finance App Team

> This document is the **canonical source of design tokens** for the personal expense management app — colors, typography, spacing, radius, elevation, z-index, breakpoints, and motion. Every screen and component spec references token names defined here (e.g. `color.bg.surface`); none should hard-code raw hex, px, or ms values.

> **Single theme (Light) for the current release.** A dark theme is out of scope; token names are namespaced so a dark palette can be added later without renaming tokens. **VND-only** app — no locale/currency tokens are needed here (number formatting lives in each spec's §9).

---

## 1. Purpose

Tokens are the shared vocabulary between design and code. A spec says "Available positive → `color.text.positive`"; this document says what `color.text.positive` actually is. Implementations expose each token as a CSS custom property (`--color-text-positive`) or platform equivalent, so a single change here propagates everywhere.

**Rules**
- Specs reference tokens by **dot path** (`color.bg.danger`). Code uses the **CSS variable** form (`--color-bg-danger`).
- Never reference a raw value in a spec or component. If you need a value that has no token, add the token here first.
- Semantic tokens (`color.text.negative`) are preferred over primitive tokens (`palette.red.600`). Specs should only ever use semantic tokens.

---

## 2. Naming Convention

```
{category}.{role}.{variant}
```

| Segment | Examples |
|---|---|
| `category` | `color`, `space`, `radius`, `shadow`, `font`, `z`, `motion` |
| `role` (color only) | `bg` (background), `text`, `border`, `icon` |
| `variant` | `primary`, `secondary`, `surface`, `danger`, `dangerSubtle`, `onDanger`, … |

`onX` text tokens (`color.text.onDanger`) are the foreground color to use **on top of** the matching `color.bg.X` fill — they guarantee contrast.

---

## 3. Color Palette (primitives)

Internal scale that semantic tokens map onto. **Specs must not reference these directly** — use the semantic tokens in §4.

| Token | Hex |
|---|---|
| `palette.gray.0` | `#FFFFFF` |
| `palette.gray.25` | `#FCFCFD` |
| `palette.gray.50` | `#F5F6F8` |
| `palette.gray.100` | `#F0F2F5` |
| `palette.gray.200` | `#EAECF0` |
| `palette.gray.300` | `#D0D5DD` |
| `palette.gray.500` | `#667085` |
| `palette.gray.700` | `#344054` |
| `palette.gray.900` | `#101828` |
| `palette.blue.50` | `#EFF6FF` |
| `palette.blue.100` | `#E8F0FE` |
| `palette.blue.600` | `#1A73E8` |
| `palette.blue.700` | `#175CD3` |
| `palette.green.50` | `#ECFDF3` |
| `palette.green.600` | `#067647` |
| `palette.red.50` | `#FEF3F2` |
| `palette.red.600` | `#D92D20` |
| `palette.red.700` | `#B42318` |

---

## 4. Semantic Color Tokens

These are the tokens specs reference. Each row lists the dot path, CSS variable, resolved value, and primary usage seen in the specs.

### 4.1 Background — `color.bg.*`

| Token | CSS var | Value | Used for |
|---|---|---|---|
| `color.bg.page` | `--color-bg-page` | `#F5F6F8` | Screen page background (`BudgetOverview` §10) |
| `color.bg.surface` | `--color-bg-surface` | `#FFFFFF` | Cards, panels, default table rows, detail panel |
| `color.bg.subtleTint` | `--color-bg-subtleTint` | `#F0F2F5` | Group header rows in `EnvelopeTable` |
| `color.bg.selectedRow` | `--color-bg-selectedRow` | `#E8F0FE` | Highlighted selected envelope row |
| `color.bg.primary` | `--color-bg-primary` | `#1A73E8` | Primary buttons (e.g. Modal Save / Assign) |
| `color.bg.danger` | `--color-bg-danger` | `#D92D20` | Destructive button fill (Delete) |
| `color.bg.dangerSubtle` | `--color-bg-dangerSubtle` | `#FEF3F2` | `AlertBar` danger background (overspent) |
| `color.bg.info` | `--color-bg-info` | `#1570EF` | Info button fill |
| `color.bg.infoSubtle` | `--color-bg-infoSubtle` | `#EFF6FF` | `AlertBar` info background (unassigned money) |
| `color.bg.overlay` | `--color-bg-overlay` | `#FFFFFF` | Popover / bottom-sheet surface (`ActivityPopup`) |
| `color.bg.scrim` | `--color-bg-scrim` | `rgba(16, 24, 40, 0.50)` | Modal backdrop / dimming layer behind dialogs & sheets |
| `color.bg.skeleton` | `--color-bg-skeleton` | `#E4E7EC` | Skeleton placeholder fill (`LoadingState`) |
| `color.bg.skeletonShimmer` | `--color-bg-skeletonShimmer` | `#F2F4F7` | Skeleton shimmer highlight band |

> `color.bg.overlay` is the **content surface** of a popover/sheet, not the dim layer. The dim layer is `color.bg.scrim`.

### 4.2 Text — `color.text.*`

| Token | CSS var | Value | Used for |
|---|---|---|---|
| `color.text.primary` | `--color-text-primary` | `#101828` | Default text, envelope/group names |
| `color.text.secondary` | `--color-text-secondary` | `#667085` | Muted text, `available = 0`, inactive filter tab, empty-state copy |
| `color.text.positive` | `--color-text-positive` | `#067647` | `available > 0`, income amounts (green) |
| `color.text.negative` | `--color-text-negative` | `#D92D20` | `available < 0`, expense amounts (red) |
| `color.text.danger` | `--color-text-danger` | `#B42318` | Danger emphasis text / danger alert copy |
| `color.text.info` | `--color-text-info` | `#175CD3` | Info emphasis text / info alert copy |
| `color.text.onPrimary` | `--color-text-onPrimary` | `#FFFFFF` | Text on `color.bg.primary` |
| `color.text.onDanger` | `--color-text-onDanger` | `#FFFFFF` | Text on `color.bg.danger` (Delete button label) |
| `color.text.onInfo` | `--color-text-onInfo` | `#FFFFFF` | Text on `color.bg.info` |

> **Money color semantics** are fixed by the budgeting domain: green (`positive`) = money you still have to spend; red (`negative`) = overspent or an expense outflow; muted (`secondary`) = exactly zero / fully assigned. See `EnvelopeTable.md` §5 and `BudgetOverview.md` §7.4.

### 4.3 Border — `color.border.*`

| Token | CSS var | Value | Used for |
|---|---|---|---|
| `color.border.default` | `--color-border-default` | `#D0D5DD` | Inputs, buttons, card outlines |
| `color.border.subtle` | `--color-border-subtle` | `#EAECF0` | Table row dividers, low-emphasis separators |
| `color.border.active` | `--color-border-active` | `#1A73E8` | Active inline-edit cell border, active filter-tab underline, focus ring base |

### 4.4 Icon — `color.icon.*`

| Token | CSS var | Value | Used for |
|---|---|---|---|
| `color.icon.secondary` | `--color-icon-secondary` | `#667085` | Month navigator arrows, edit ✏️, chevrons |
| `color.icon.danger` | `--color-icon-danger` | `#D92D20` | Danger / overspent iconography |
| `color.icon.info` | `--color-icon-info` | `#1570EF` | Info alert iconography |

---

## 5. Typography — `font.*`

Vietnamese requires full diacritic coverage (e.g. ữ, ậ, ỡ). **Inter** is the primary family (complete Vietnamese subset); the system stack is the fallback.

| Token | CSS var | Value |
|---|---|---|
| `font.family.base` | `--font-family-base` | `"Inter", -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif` |
| `font.family.mono` | `--font-family-mono` | `"JetBrains Mono", "SF Mono", Menlo, Consolas, monospace` |

### 5.1 Type scale

| Token | CSS var | Size / Line-height | Used for |
|---|---|---|---|
| `font.size.xs` | `--font-size-xs` | `12px / 16px` | Status badges, table column headers, captions |
| `font.size.sm` | `--font-size-sm` | `14px / 20px` | Secondary text, memo, dates |
| `font.size.md` | `--font-size-md` | `16px / 24px` | Body, table cell values (default) |
| `font.size.lg` | `--font-size-lg` | `18px / 26px` | Detail Panel envelope name, section titles |
| `font.size.xl` | `--font-size-xl` | `22px / 30px` | Month label in page header |
| `font.size.2xl` | `--font-size-2xl` | `28px / 36px` | Empty-state / error-state titles |

### 5.2 Weights

| Token | CSS var | Value |
|---|---|---|
| `font.weight.regular` | `--font-weight-regular` | `400` |
| `font.weight.medium` | `--font-weight-medium` | `500` |
| `font.weight.semibold` | `--font-weight-semibold` | `600` |
| `font.weight.bold` | `--font-weight-bold` | `700` |

> Group names and the month label use `semibold`; envelope names and body use `regular`. Monetary values use `medium` for legibility of digits.

---

## 6. Spacing — `space.*`

4px base unit. Used for padding, gaps, and margins.

| Token | CSS var | Value |
|---|---|---|
| `space.xs` | `--space-xs` | `4px` |
| `space.sm` | `--space-sm` | `8px` |
| `space.md` | `--space-md` | `16px` |
| `space.lg` | `--space-lg` | `24px` |
| `space.xl` | `--space-xl` | `32px` |
| `space.2xl` | `--space-2xl` | `48px` |

> The desktop two-column grid gap (`BudgetOverview` §7.1, "gap 24px") is `space.lg`.

> **Alias note:** some early component specs reference `spacing.sm` / `spacing.md` / `spacing.xl`. These are **aliases of the `space.*` scale** (`spacing.sm` → `space.sm`, etc.). `space.*` is canonical; `spacing.*` is accepted but deprecated and should be migrated.

---

## 7. Radius — `radius.*`

| Token | CSS var | Value | Used for |
|---|---|---|---|
| `radius.sm` | `--radius-sm` | `4px` | Inputs, inline-edit field, status badges |
| `radius.md` | `--radius-md` | `8px` | Cards, panels, buttons, popover surface |
| `radius.lg` | `--radius-lg` | `12px` | Bottom sheets, modals |
| `radius.full` | `--radius-full` | `9999px` | Pills (status badges), avatar/circular elements |

---

## 8. Elevation / Shadow — `shadow.*`

| Token | CSS var | Value | Used for |
|---|---|---|---|
| `shadow.sm` | `--shadow-sm` | `0 1px 2px rgba(16,24,40,0.06)` | Resting cards, sticky detail panel |
| `shadow.md` | `--shadow-md` | `0 4px 12px rgba(16,24,40,0.12)` | Popover / `ActivityPopup`, dropdowns |
| `shadow.lg` | `--shadow-lg` | `0 12px 32px rgba(16,24,40,0.20)` | Modals, bottom sheets |

> **Alias note:** overlay specs reference `color.shadow.md`. This is an **alias of `shadow.md`** — shadows live in the `shadow.*` namespace (canonical), not under `color.*`. Migrate `color.shadow.md` → `shadow.md`.

---

## 9. Z-Index — `z.*`

Single layering scale so overlays stack predictably.

| Token | CSS var | Value | Layer |
|---|---|---|---|
| `z.base` | `--z-base` | `0` | Normal content flow |
| `z.sticky` | `--z-sticky` | `100` | Sticky detail panel, sticky table header |
| `z.dropdown` | `--z-dropdown` | `200` | Month picker, inline-edit overlay |
| `z.popover` | `--z-popover` | `300` | `ActivityPopup` (desktop popover) |
| `z.scrim` | `--z-scrim` | `400` | Modal / bottom-sheet backdrop |
| `z.modal` | `--z-modal` | `500` | `Modal` dialog, mobile bottom sheets |
| `z.toast` | `--z-toast` | `600` | Transient notifications (above everything) |

---

## 10. Breakpoints — `breakpoint.*`

Drives the responsive layout switch in `BudgetOverview` §7.1 (two-column desktop ↔ single-column mobile) and the platform variants in the overlay specs.

| Token | CSS var | Value | Meaning |
|---|---|---|---|
| `breakpoint.mobile` | `--breakpoint-mobile` | `< 768px` | Single column; detail panel & popups render inline (web mobile) |
| `breakpoint.tablet` | `--breakpoint-tablet` | `768px–1023px` | Single column, wider gutters |
| `breakpoint.desktop` | `--breakpoint-desktop` | `≥ 1024px` | Two-column ~65% / ~35% grid; sticky detail panel; popover overlays |

> Native Android / iOS use platform sheets regardless of width (see each overlay spec's Platform Notes).

---

## 11. Motion — `motion.*`

| Token | CSS var | Value | Used for |
|---|---|---|---|
| `motion.duration.fast` | `--motion-duration-fast` | `120ms` | Hover, inline-edit focus, row highlight |
| `motion.duration.base` | `--motion-duration-base` | `200ms` | Popover open/close, tab switch |
| `motion.duration.slow` | `--motion-duration-slow` | `300ms` | Bottom-sheet slide-up, modal enter |
| `motion.easing.standard` | `--motion-easing-standard` | `cubic-bezier(0.2, 0, 0, 1)` | Most transitions |
| `motion.easing.decelerate` | `--motion-easing-decelerate` | `cubic-bezier(0, 0, 0, 1)` | Entrances (sheets, popovers) |
| `motion.skeletonShimmer` | `--motion-skeletonShimmer` | `1500ms` linear infinite | `LoadingState` shimmer sweep |

> **Reduced motion:** when `prefers-reduced-motion: reduce` is set, disable the skeleton shimmer (`LoadingState` §7) and replace slide/scale transitions with instant or opacity-only changes. Honor this everywhere animation is used.

---

## 12. Token Reference Index

Every token currently referenced by a screen or component spec, with the file(s) that use it. Use this to confirm a token exists before referencing it.

| Token | Referenced by |
|---|---|
| `color.bg.page` | BudgetOverview |
| `color.bg.surface` | BudgetOverview, EnvelopeTable, EmptyState, ErrorState |
| `color.bg.subtleTint` | BudgetOverview, EnvelopeTable |
| `color.bg.selectedRow` | BudgetOverview, EnvelopeTable |
| `color.bg.primary` | Modal, AlertBar |
| `color.bg.danger` | BudgetOverview, Modal |
| `color.bg.dangerSubtle` | BudgetOverview, AlertBar |
| `color.bg.info` | AlertBar |
| `color.bg.infoSubtle` | BudgetOverview, AlertBar |
| `color.bg.overlay` | BudgetOverview, ActivityPopup |
| `color.bg.scrim` | Modal *(new — backdrop)* |
| `color.bg.skeleton` | LoadingState |
| `color.bg.skeletonShimmer` | LoadingState |
| `color.text.primary` | BudgetOverview, EnvelopeTable, InlineEdit |
| `color.text.secondary` | BudgetOverview, EnvelopeTable, EmptyState, ErrorState |
| `color.text.positive` | BudgetOverview, EnvelopeTable, ActivityPopup |
| `color.text.negative` | BudgetOverview, EnvelopeTable, ActivityPopup |
| `color.text.danger` | AlertBar, Modal |
| `color.text.info` | AlertBar |
| `color.text.onPrimary` | Modal, AlertBar |
| `color.text.onDanger` | BudgetOverview, Modal |
| `color.text.onInfo` | AlertBar |
| `color.border.default` | InlineEdit, Modal |
| `color.border.subtle` | EnvelopeTable |
| `color.border.active` | BudgetOverview, EnvelopeTable, InlineEdit |
| `color.icon.secondary` | BudgetOverview, ErrorState |
| `color.icon.danger` | ErrorState, AlertBar |
| `color.icon.info` | AlertBar |
| `shadow.md` (alias `color.shadow.md`) | BudgetOverview, ActivityPopup, Modal |
| `space.* / spacing.*` | EnvelopeTable, Modal, ActivityPopup, EmptyState |
| `radius.sm`, `radius.md` | (component cards / inputs) |

---

## 13. Open Questions

1. **Dark theme** — out of scope for this release. When added, semantic tokens keep their names and only resolved values change. No spec changes required. → Tracked for a future version.
2. **Focus-ring token** — focus styling currently reuses `color.border.active`. Should there be a dedicated `color.border.focus` + ring width/offset tokens for accessibility? → Affects all interactive components.
3. **Alias migration** — `spacing.*` and `color.shadow.md` are accepted aliases. Decide a deprecation date and migrate referencing specs to `space.*` / `shadow.md`. → Affects EnvelopeTable, Modal, ActivityPopup, EmptyState.

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | **1.0** | Initial design-token spec. Defined the full semantic color system (bg / text / border / icon), primitive palette, typography (Inter, Vietnamese diacritic coverage), spacing, radius, elevation/shadow, z-index layering, breakpoints, and motion. Catalogued every token referenced by BudgetOverview and the 8 component specs. Reconciled two inconsistencies: `spacing.*` documented as deprecated alias of `space.*`; `color.shadow.md` documented as alias of `shadow.md`. Added `color.bg.scrim` for modal/sheet backdrops. | Finance App Team |
