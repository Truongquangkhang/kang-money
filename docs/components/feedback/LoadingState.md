# Component: LoadingState

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-27
**Category:** feedback

> Skeleton placeholder component shown during the initial `Loading` screen state, before budget data arrives.

---

## 1. Purpose

`LoadingState` renders animated skeleton placeholders that mimic the shape of the content they temporarily replace, so the layout does not visibly jump when real data arrives. It is the canonical first-paint experience for any screen that fetches its own data.

On the Budget Overview screen it is the content displayed while `screenState = loading` — that is, after `ScreenOpened` fires but before `BudgetDataReceived` populates the envelope list and totals (see `docs/screens/BudgetOverview.md` §6 State Machine). The Budget Overview renders `LoadingState` inside a root wrapper with `data-testid` `budget-overview-loading`; the skeleton itself carries `data-testid` `skeleton-loader`.

This component is **presentational only**. It owns no data, fires no events, and has no internal state transitions. It does not detect timeouts, retries, or errors — the parent screen decides when to mount and unmount it (it is replaced by the ready content on `BudgetDataReceived`, or by `ErrorState` on `ApiErrorReceived`).

> **Re-fetch loading (month change) — out of scope for this version.** When the user changes month (`Ready → Loading`), the Budget Overview may show a lighter inline loading treatment rather than the full skeleton. This component specifies the **initial** full-screen load only. See §6 Open behavior note.

---

## 2. Anatomy

The `full` variant mirrors the desktop two-column Budget Overview layout (`BudgetOverview.md` §7.1): a page header, a filter tabs row, the envelope table (group rows + envelope rows with three value columns), and the right-hand detail panel card.

```
+--------------------------------------------------------------------------+
|  [skeleton-loader]  role="status"  aria-busy="true"                      |
|                                                                          |
|  +-- Header skeleton --------------------------------------------------+ |
|  |  [◄]   ▭▭▭▭▭▭▭▭ (month label)   [►]                                 | |
|  +--------------------------------------------------------------------+ |
|                                                                          |
|  +-- Filter tabs skeleton --------------------------------------------+ |
|  |  ▭▭▭  ▭▭▭▭  ▭▭▭▭▭  ▭▭▭▭  ▭▭▭▭▭                                     | |
|  +--------------------------------------------------------------------+ |
|                                                                          |
|  +-- Table skeleton (~65%) ----------+  +-- Panel skeleton (~35%) ----+ |
|  |  ▮ ▭▭▭▭▭▭ (group row)             |  |  ▭▭▭▭▭▭▭▭▭▭ (title)         | |
|  |    ▭▭▭▭▭   ▭▭▭▭   ▭▭▭▭   ▭▭▭▭     |  |  ▭▭▭▭▭▭                     | |
|  |    Name    Asgn   Actv   Avail   |  |                             | |
|  |    ▭▭▭▭▭   ▭▭▭▭   ▭▭▭▭   ▭▭▭▭     |  |  ▭▭▭▭▭▭▭▭▭▭▭▭▭▭▭▭▭▭▭        | |
|  |    ▭▭▭▭▭   ▭▭▭▭   ▭▭▭▭   ▭▭▭▭     |  |  ▭▭▭▭▭▭▭▭▭▭▭▭               | |
|  |  ▮ ▭▭▭▭▭▭ (group row)             |  |  ▭▭▭▭▭▭▭▭▭▭▭▭▭▭▭▭           | |
|  |    ▭▭▭▭▭   ▭▭▭▭   ▭▭▭▭   ▭▭▭▭     |  |                             | |
|  |    ▭▭▭▭▭   ▭▭▭▭   ▭▭▭▭   ▭▭▭▭     |  |  ▭▭▭▭▭▭▭▭ (button)          | |
|  +-----------------------------------+  +-----------------------------+ |
|                                                                          |
+--------------------------------------------------------------------------+

Legend:  ▭ = skeleton text/value bar   ▮ = skeleton icon/chevron block
```

The `table` variant renders only the left column (header is optional, see Props). The `panel` variant renders only the right-hand detail panel card. On mobile / narrow layouts the panel skeleton stacks below the table skeleton, matching `BudgetOverview.md` §7.1 (Web Mobile / Narrow).

---

## 3. Props

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| `variant` | `'table' \| 'panel' \| 'full'` | No | `'full'` | Which skeleton shape to render. `full` = header + filter tabs + table + detail panel; `table` = envelope table region only; `panel` = detail panel card only. *Design proposal — see note below.* |
| `rows` | `number` | No | `6` | Number of skeleton **envelope rows** to render in the table region. Ignored when `variant = 'panel'`. *Design proposal.* |
| `groups` | `number` | No | `2` | Number of skeleton **group header rows** to render, with `rows` envelope rows distributed beneath them. Ignored when `variant = 'panel'`. *Design proposal.* |
| `animated` | `boolean` | No | `true` | Whether the shimmer animation plays. When `false`, placeholders render as static blocks. Always forced to `false` under `prefers-reduced-motion` (see §7). *Design proposal.* |
| `showHeader` | `boolean` | No | `true` | Whether the month-navigator header skeleton is rendered. Useful for embedding the `table` variant where a real header is already present. *Design proposal.* |
| `label` | `string` | No | `'Đang tải…'` | Visually-hidden accessible label announced by assistive tech (see §7, §9). *Design proposal.* |
| `data-testid` | `string` | No | `'skeleton-loader'` | Test hook for the skeleton root. **Hard contract** for Budget Overview: must be `skeleton-loader` (`BudgetOverview.md` §13.6). |

> **Contract vs. proposal.** The only hard contract from `BudgetOverview.md` is that the skeleton root carries `data-testid="skeleton-loader"` and is rendered inside the screen's `budget-overview-loading` wrapper, shaped to mimic header + filter tabs + envelope table + detail panel. All other props (`variant`, `rows`, `groups`, `animated`, `showHeader`, `label`) are **design proposals** for this component's own API and may be adjusted during implementation without changing the screen contract.

---

## 4. Variants

| Variant | Renders | Used by |
|---|---|---|
| `full` | Header + filter tabs + table region + detail panel card | Budget Overview initial load (desktop and mobile) |
| `table` | Envelope table region only (group rows + envelope rows × `rows`) | Screens that re-load only the list area; embedding where the header is real |
| `panel` | Detail panel card skeleton only | Selection-driven panels that load independently (future use) |

All variants share the same skeleton primitive (a shimmering rounded rectangle); they differ only in composition. The `full` variant is the default and the one the Budget Overview screen uses.

---

## 5. States

`LoadingState` is stateless from the screen's perspective — it is mounted, displayed, and unmounted by the parent. It has two visual presentations driven by environment, not by props alone:

| Presentation | Condition | Behavior |
|---|---|---|
| **Animated (shimmer)** | `animated = true` **and** `prefers-reduced-motion` is not set | Placeholders pulse with a left-to-right shimmer sweep |
| **Static** | `animated = false` **or** `prefers-reduced-motion: reduce` | Placeholders render as solid muted blocks, no motion |

There is no `error` or `empty` presentation. Those are owned by sibling components (`ErrorState.md`, `EmptyState.md`); the parent screen swaps `LoadingState` out for them on the relevant state transition.

---

## 6. Interaction Rules

- `LoadingState` is **non-interactive**. It contains no focusable controls, no buttons, and no click targets. Pointer events on the skeleton do nothing.
- Skeleton placeholders must **not** be selectable, tab-navigable, or announced as actionable elements.
- The component does not manage its own visibility — the parent screen mounts it while `screenState = loading` and unmounts it on `BudgetDataReceived` (`Ready`) or `ApiErrorReceived` (`Error`).
- The skeleton dimensions should match the real content's layout box as closely as possible to avoid layout shift when real data swaps in.

> **Open behavior note (month re-fetch):** On `Ready → Loading` (user changed month), the Budget Overview is **not** required to render the full skeleton; a lighter inline indicator is permitted. Whether `LoadingState` (e.g. the `table` variant) is reused for that case is deferred to the screen implementation and is not specified here.

---

## 7. Accessibility

- The skeleton root has `role="status"` and `aria-live="polite"` so assistive tech announces that content is loading without interrupting the user.
- The root sets `aria-busy="true"` while mounted. The parent screen region it occupies should reflect `aria-busy="true"` during `Loading` and `aria-busy="false"` once `Ready`.
- A visually-hidden text label (the `label` prop, default `Đang tải…`) provides the spoken announcement. Screen readers announce the label, not the individual placeholder bars.
- Individual skeleton placeholder bars are decorative and **must** be hidden from the accessibility tree (`aria-hidden="true"`). Only the single status label is exposed.
- **Reduced motion:** when `prefers-reduced-motion: reduce` is set, the shimmer animation is disabled regardless of the `animated` prop. Placeholders render as static muted blocks. This is a hard requirement, not a preference.
- No element inside `LoadingState` is focusable; it never appears in the tab order.

---

## 8. Design Tokens

References `docs/Theme.md` for canonical tokens. The skeleton surface / shimmer tokens are **TBD** — `docs/Theme.md` does not yet define them and must be added before implementation.

| Element | Token |
|---|---|
| Skeleton placeholder base fill | `color.bg.skeleton` *(TBD — not yet defined in `docs/Theme.md`)* |
| Skeleton shimmer highlight (sweep) | `color.bg.skeletonShimmer` *(TBD)* |
| Skeleton card background | `color.bg.surface` |
| Page background behind skeleton | `color.bg.page` |
| Group header row skeleton tint | `color.bg.subtleTint` |
| Visually-hidden label text | `color.text.secondary` |
| Placeholder corner radius | `radius.sm` *(TBD — confirm in `docs/Theme.md`)* |
| Shimmer animation duration | `motion.duration.skeleton` *(TBD — proposed ~1.2s, ease-in-out, infinite)* |

> All `TBD` tokens are blockers for implementation and should be resolved when `docs/Theme.md` is authored. Until then, `color.bg.skeleton` is the placeholder name used throughout this spec.

---

## 9. Content / Formatting

- `LoadingState` renders **no real data** — every visible bar is a placeholder. It must never display partial or stale budget values.
- No monetary values, dates, or month labels are shown. There is therefore no VND formatting in this component (see `BudgetOverview.md` §9 for the formatting rules that apply once `Ready`).
- The only textual content is the visually-hidden accessible label, defaulting to the Vietnamese string `Đang tải…` ("Loading…"). Copy is single-language (Vietnamese) consistent with the app's VND-only, Vietnamese-locale scope.
- Placeholder bar widths should be varied (not all uniform) to read as plausible content rather than a grid of identical blocks — e.g. envelope-name bars wider than value-column bars.

---

## 10. Platform Notes

### Web Desktop
- Renders the two-column composition (table skeleton ~65% / panel skeleton ~35%), matching `BudgetOverview.md` §7.1. The panel skeleton is top-aligned in the right column.

### Web Mobile / Narrow
- Single column. The panel skeleton stacks **below** the table skeleton (no bottom sheet), mirroring the real narrow layout.

### Android / iOS
- The detail panel renders as a bottom sheet when active on native (`BudgetOverview.md` §11), but during initial load nothing is selected, so the `full` skeleton shows only the table region; the `panel` skeleton is not rendered on first load.
- Respect the OS-level reduced-motion setting in addition to the CSS `prefers-reduced-motion` media query.

---

## 11. Test IDs

| Element | data-testid | Description |
|---|---|---|
| Skeleton root | `skeleton-loader` | Root of the skeleton placeholders. **Hard contract** — rendered inside the screen's `budget-overview-loading` wrapper during `Loading` |
| Header skeleton | `skeleton-header` | Month-navigator placeholder region (rendered when `showHeader = true`) |
| Filter tabs skeleton | `skeleton-filter-tabs` | Placeholder row for the filter tabs |
| Table skeleton | `skeleton-table` | Envelope table placeholder region |
| Group row skeleton | `skeleton-group-row` | A single skeleton group header row |
| Envelope row skeleton | `skeleton-row` | A single skeleton envelope row (group + value columns) |
| Detail panel skeleton | `skeleton-panel` | Detail panel card placeholder (rendered for `full` and `panel` variants) |
| Visually-hidden label | `skeleton-label` | The `role="status"` accessible loading label |

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | 1.0 | Initial spec | Finance App Team |
