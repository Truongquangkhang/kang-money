# Component: EmptyState

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-27
**Category:** feedback

> A reusable feedback component that fills a content area when there is nothing to show — either because no data exists yet (onboarding) or because the current filter/query matched nothing.

---

## 1. Purpose

`EmptyState` is the canonical "nothing here" surface across the app. It communicates *why* an area is empty and, where appropriate, offers the single next action that resolves the emptiness. It is intentionally minimal: a title, a short body, an optional illustration/icon, and an optional call-to-action.

On the Budget Overview screen (`docs/screens/BudgetOverview.md`) the component appears in two distinct, deliberately different forms:

1. **No envelopes exist at all** — a first-run / onboarding moment. The screen renders a centered, illustrated `EmptyState` with onboarding copy and a `+ Add Category Group` CTA. `data-testid`: `envelope-table-empty`.
2. **A filter matched no envelopes** — a transient, recoverable moment. The screen renders a lightweight, single-line inline `EmptyState` reading `No envelopes match this filter`, placed directly below the filter tabs. `data-testid`: `filter-empty`.

The two cases are the same component driven by the `variant` prop (`full` vs `inline`); they share no copy and have very different visual weight.

> **Note — Activity Popup empty message.** The Activity Popup shows its own empty message (`No transactions this month`, `data-testid`: `activity-popup-empty`). `EmptyState` (`inline` variant) **may** be used to render it, but the popup **owns** that copy and `data-testid` — `EmptyState` is purely the presentational shell. This spec does not define the Activity Popup empty copy.

This component is presentation-only. It owns no data fetching, no filter logic, and no state machine — the parent screen decides *when* to render it and supplies all copy and actions via props.

---

## 2. Anatomy

### `full` variant (onboarding — no envelopes)

```
+-------------------------------------------------------+
|                                                       |
|                    ┌───────────┐                      |
|                    │           │                      |
|                    │    🗂️     │   ← illustration/icon |
|                    │           │                      |
|                    └───────────┘                      |
|                                                       |
|              No categories yet                        |   ← title
|                                                       |
|     Create your first category group to start         |   ← body copy
|     giving every đồng a job.                           |
|                                                       |
|              ┌─────────────────────────┐              |
|              │  + Add Category Group   │              |   ← CTA (optional)
|              └─────────────────────────┘              |
|                                                       |
+-------------------------------------------------------+
        (centered vertically + horizontally)
```

### `inline` variant (filter returned nothing)

```
  No envelopes match this filter
  └─ single muted line · left-aligned · no illustration · no CTA
```

| Slot | `full` | `inline` |
|---|---|---|
| Illustration / icon | Rendered, centered above title | Not rendered |
| Title | Rendered, prominent | Optional (usually omitted; body line stands alone) |
| Body copy | Rendered | Rendered as the single muted line |
| CTA (`action`) | Rendered as a primary button below body | Not rendered by default |

---

## 3. Props

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| `variant` | `'full' \| 'inline'` | No | `'full'` | `full` = centered, illustrated onboarding block with optional CTA. `inline` = single muted line, no illustration, no CTA by default. |
| `title` | `string` | No | — | Heading line. Required in practice for `full`; usually omitted for `inline`. Ignored if empty. |
| `message` | `string` | Yes | — | Body copy. The primary line of explanation. Alias: `body` (accepted as a synonym; `message` is canonical). |
| `body` | `string` | No | — | Synonym for `message`. If both are supplied, `message` wins. |
| `action` | `{ label: string, onPress: () => void }` | No | — | Optional CTA. When present and `variant = full`, renders a primary button. Ignored when `variant = inline` unless explicitly passed (see §6). |
| `icon` | `string \| node` | No | — | Icon glyph or element shown above the title in `full`. Ignored in `inline`. |
| `illustration` | `node` | No | — | Larger illustration element, used in `full` in place of `icon` when richer art is desired. If both `icon` and `illustration` are supplied, `illustration` wins. Ignored in `inline`. |

> `EmptyState` renders nothing meaningful without at least a `message`. Passing neither `message` nor `body` is a developer error.

---

## 4. Variants

| Variant | Usage | Layout | Illustration | CTA |
|---|---|---|---|---|
| `full` | First-run / no-data onboarding moments (e.g. no envelopes exist). | Centered both axes within its container; generous vertical padding. | Yes — `icon` or `illustration` above the title. | Optional, primary button below body. |
| `inline` | Recoverable empty results inside an already-populated screen (e.g. a filter matched nothing). | Left-aligned, compact, single line; sits in the content flow below the filter tabs. | No. | None by default. |

### Budget Overview bindings

| Usage | `variant` | `title` | `message` | `action` | `data-testid` |
|---|---|---|---|---|---|
| No envelopes exist | `full` | `No categories yet` | `Create your first category group to start giving every đồng a job.` | `{ label: "+ Add Category Group", onPress: openAddGroupFlow }` | `envelope-table-empty` |
| Filter matched nothing | `inline` | — | `No envelopes match this filter` | — | `filter-empty` |

> Copy above is the contract from `docs/screens/BudgetOverview.md` §5 (Empty States). The `+ Add Category Group` CTA shares the same target action as the toolbar `add-group-btn` button.

---

## 5. States

`EmptyState` is stateless. It has no internal lifecycle, no loading/error/ready phases of its own, and no reducer. It is rendered or not rendered by the parent.

| Condition | Result |
|---|---|
| `message`/`body` present, `variant = full`, no `action` | Illustration + title + body, centered. No button. |
| `message`/`body` present, `variant = full`, `action` present | Illustration + title + body + primary CTA button, centered. |
| `message`/`body` present, `variant = inline` | Single muted line in content flow. |
| `action.onPress` in flight | `EmptyState` does not track this. If the action triggers loading, the parent unmounts `EmptyState` and renders `LoadingState`. |
| No `message` and no `body` | Developer error — nothing useful renders. Avoid. |

> There is no hover/pressed state for the component as a whole. The CTA button inherits its own interactive states from the app's primary button (see `Modal.md` / Theme button tokens).

---

## 6. Interaction Rules

- `EmptyState` itself is non-interactive; the only interactive element is the optional CTA button.
- **CTA press:** clicking/tapping the CTA fires `action.onPress`. The component does not debounce or guard against double-press — the parent owns idempotency (the Budget Overview `+ Add Category Group` flow opens a single group-creation flow).
- **`inline` + `action`:** by default the `inline` variant does not render a CTA. If an `action` is explicitly passed to an `inline` instance, it renders as a subtle text-link after the message rather than a full button. The Budget Overview `filter-empty` usage passes no `action`.
- **No dismiss:** `EmptyState` has no close affordance. Emptiness is resolved by data changing (an envelope is added, the filter is changed), at which point the parent stops rendering it.
- **Filter recovery:** the `filter-empty` instance disappears automatically when the user switches to a filter tab that matches rows — no action required from within `EmptyState`.

---

## 7. Accessibility

- The `full` variant container uses `role="status"` (a polite live region) so screen readers announce the onboarding state when it appears without stealing focus.
- The `inline` variant is also announced via `role="status"` / `aria-live="polite"` so a screen-reader user hears that the filter returned nothing when they switch tabs.
- The `title` renders as a heading at the appropriate level for its container (the Budget Overview left column treats it as a section-level heading). It is exposed as the accessible name of the region via `aria-labelledby`.
- The illustration/icon is decorative and is marked `aria-hidden="true"`; the title and body carry all meaning. No information is conveyed by the image alone.
- The CTA is a real `<button>` (or platform equivalent), fully keyboard-focusable and activatable with `Enter` and `Space`. It carries a descriptive accessible label matching its visible text (`+ Add Category Group`).
- Color is never the sole signal — muted copy uses `color.text.secondary`, but the message text itself states the situation.
- Touch target for the CTA meets the platform minimum (≥ 44×44 pt on mobile).

---

## 8. Design Tokens

References `docs/Theme.md` for all canonical tokens.

| Element | Token |
|---|---|
| Container background (`full`) | `color.bg.surface` |
| Container background (`inline`) | transparent (inherits parent) |
| Title text | `color.text.primary` |
| Body copy (`full`) | `color.text.secondary` |
| Body copy / single line (`inline`) | `color.text.secondary` |
| Illustration / icon tint | `color.icon.secondary` |
| CTA button background | `color.bg.primary` |
| CTA button text | `color.text.onPrimary` |
| CTA (inline text-link) text | `color.text.primary` |
| Vertical padding (`full`) | `spacing.xl` (top + bottom) |
| Gap between slots (`full`) | `spacing.md` |
| Inline line padding | `spacing.sm` vertical |
| Title typography | `type.heading.sm` |
| Body / inline typography | `type.body.md` |

> `color.bg.primary` / `color.text.onPrimary` / `spacing.*` / `type.*` are expected canonical tokens in `docs/Theme.md`. Where a token is not yet defined there, treat the entry above as the proposed binding (TBD against Theme).

---

## 9. Content / Formatting

- **No monetary values** are rendered by `EmptyState` — it carries no amounts, so the VND formatting rules of the screen spec do not apply here.
- Copy is sentence case, no terminal period on the `inline` single line (`No envelopes match this filter`), terminal period allowed on `full` multi-sentence body.
- Vietnamese diacritics are preserved verbatim in copy (e.g. `đồng`, `hũ`). The component does not transform or normalize text.
- The `full` title is short (≤ 4 words recommended); the body is one or two short sentences. Long copy is the parent's responsibility to avoid.
- CTA labels are imperative and lead with the action (`+ Add Category Group`). The leading `+` glyph is part of the label string, not a separate icon.
- The product term for a budget category is **envelope (hũ)**; onboarding copy may use either "category" or "envelope" to match the surrounding screen — the Budget Overview onboarding copy uses "category group" to match its toolbar button.

---

## 10. Platform Notes

### Web Desktop
- `full` variant fills the left-column content area where the `EnvelopeTable` would be, centered within that column's width.
- `inline` variant sits directly below the filter tabs row, left-aligned to the table's content edge.

### Web Mobile / Narrow
- `full` variant spans full width with reduced vertical padding; illustration scales down but remains centered.
- `inline` variant unchanged — a single muted line in the content flow.

### Android / iOS
- `full` variant is centered in the available content region. The CTA button respects the platform minimum touch target and uses the native primary-button styling.
- `inline` variant renders as a muted line below the filter tabs; no bottom sheet, no overlay.
- When `EmptyState` backs the Activity Popup's empty message (see §1 note), it renders inside the bottom-sheet body on mobile; the popup owns positioning and copy.

---

## 11. Test IDs

`EmptyState` exposes a single root `data-testid`, supplied by the parent (it has no fixed id of its own — the parent passes the contextually correct one). Inner elements derive from the root where applicable.

| Element | `data-testid` | Description |
|---|---|---|
| Root (no envelopes, Budget Overview) | `envelope-table-empty` | `full` variant shown when no envelopes exist |
| Root (filter, Budget Overview) | `filter-empty` | `inline` variant shown when the active filter matches no envelopes |
| Root (Activity Popup, optional backing) | `activity-popup-empty` | `inline` variant when used to render the popup's empty message (copy owned by the popup) |
| CTA button | `{root}-action` | The optional call-to-action button (e.g. `envelope-table-empty-action`) |
| Title | `{root}-title` | Heading line (`full` variant) |
| Body / message | `{root}-message` | Body copy or the single inline line |

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | **1.0** | Initial spec | Finance App Team |
