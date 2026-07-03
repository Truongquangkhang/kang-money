# Component: AlertBar

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-27
**Category:** feedback

> A single-line banner shown in the Budget Overview header that warns the user when their budget is unbalanced — either overspent (danger) or with money left to assign (info) — and offers a one-tap action to resolve it.

---

## 1. Purpose

The AlertBar communicates the **budget balance status** of the current month at a glance and gives the user a direct path to fix it. In envelope budgeting every đồng must be assigned to exactly one envelope, so the only "healthy" state is a balanced budget (`totalAssigned === totalAvailableMoney`). When the budget is out of balance the AlertBar surfaces the imbalance and its exact amount, and exposes a single action button to rebalance.

The AlertBar is **derived state** — it does not own the budget data. Its parent (the Budget Overview screen) decides which variant to render, or whether to render it at all, based on `totalAssigned` vs `totalAvailableMoney`. The component itself is presentational: it renders the icon, message, amount, and action it is given.

Only **one** AlertBar is ever visible at a time. The two variants (`danger`, `info`) are mutually exclusive, and when the budget is balanced the component is not rendered at all.

---

## 2. Anatomy

The AlertBar is a single horizontal bar with four slots: a leading status **icon**, the **message** text, the formatted **amount**, and a trailing **action button**. The message and amount occupy the flexible center; the icon is fixed-leading and the action is fixed-trailing.

```
+-----------------------------------------------------------------------+
| [icon]  message text .......................  amount   [ Action ]     |
|  (!)    You have assigned more than you have  600.000đ  [ Fix This ]   |
+-----------------------------------------------------------------------+
   ^         ^                                    ^          ^
   icon      message (flex, grows)                amount     action button
   (fixed)                                        (fixed)    (fixed, trailing)
```

| Slot | Source | Notes |
|---|---|---|
| Icon | Implied by `variant` | `danger` → warning/alert glyph; `info` → info glyph. Not a prop — derived from `variant`. |
| Message | `message` prop | Single line; truncates with ellipsis before colliding with the amount. |
| Amount | `amount` prop | Formatted as VND (see §9). For `danger` this is the absolute overspent amount; for `info` the unassigned amount. |
| Action button | `action` prop | Label + `onPress`. Always rendered (action is required). |

> The icon slot is driven entirely by `variant` and is not independently configurable in v1.0.

---

## 3. Props

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| `variant` | `'danger' \| 'info'` | Yes | — | Selects the visual treatment, icon, and `data-testid`. `danger` = overspent; `info` = unassigned money. The two are mutually exclusive. |
| `amount` | `number` | Yes | — | VND integer (no decimals). For `danger`, the **absolute value** of the overspent amount (always rendered positive). For `info`, the unassigned amount. Formatted per §9. |
| `message` | `string` | Yes | — | The human-readable message. Provided by the caller. Conventionally `You have assigned more than you have` (danger) / `You have money left to assign` (info). |
| `action` | `{ label: string, onPress: () => void }` | Yes | — | The single trailing action. `label` is the button text (e.g. `Fix This`, `Assign Now`); `onPress` is invoked on tap/click/activation. |

> **Future consideration — dismissibility.** The contract does not include a dismiss affordance. The budget imbalance is a persistent condition (it remains true until the user rebalances), so a dismissible AlertBar would re-appear on the next render and provide little value. If a future release introduces dismissible alerts, add optional `dismissible?: boolean` and `onDismiss?: () => void` props rather than overloading `action`. Out of scope for v1.0.

---

## 4. Variants

Exactly two variants. They are **mutually exclusive** — the parent renders at most one, chosen by comparing `totalAssigned` to `totalAvailableMoney`.

| Variant | Render condition | Icon | Message (convention) | Amount | Action (convention) | Background token | `data-testid` |
|---|---|---|---|---|---|---|---|
| `danger` | `totalAssigned > totalAvailableMoney` (overspent) | Warning glyph | `You have assigned more than you have` | Absolute value of overspent amount (e.g. `600.000đ`) | `{ label: "Fix This", onPress }` | `color.bg.dangerSubtle` | `alert-bar-danger` |
| `info` | `totalAssigned < totalAvailableMoney` (money left) | Info glyph | `You have money left to assign` | Unassigned amount | `{ label: "Assign Now", onPress }` | `color.bg.infoSubtle` | `alert-bar-info` |

| Condition | Result |
|---|---|
| `totalAssigned > totalAvailableMoney` | Render `danger` variant |
| `totalAssigned < totalAvailableMoney` | Render `info` variant |
| `totalAssigned === totalAvailableMoney` (balanced) | **Do not render** — the component is absent from the DOM |

> Balance is a **rendering decision made by the parent**, not a variant of this component. AlertBar has no "balanced" or "hidden" variant; it simply is not mounted when the budget is balanced.

---

## 5. States

The AlertBar is a presentational component with no internal lifecycle state (no loading/error/data-fetch states of its own). The "states" below are visual states of its elements.

| State | Applies to | Description |
|---|---|---|
| Default | Whole bar | Normal rendered state for the active variant. |
| Action — default | Action button | Resting button appearance. |
| Action — hover | Action button | Web only — pointer hover emphasis. |
| Action — pressed / active | Action button | Down state on tap/click. |
| Action — focus-visible | Action button | Keyboard focus ring (see §7). |
| Message — truncated | Message text | When the message is too long for the available width it truncates with an ellipsis; the full text remains in the accessible name. |

> **Not applicable:** loading, error, empty, and disabled states. The action button is always enabled in v1.0 — if there is nothing to fix, the AlertBar is not rendered at all (balanced budget).

---

## 6. Interaction Rules

- Tapping / clicking the action button invokes `action.onPress`. The AlertBar does **not** mutate budget state itself; `onPress` is owned by the parent (e.g. `triggerRebalanceFlow` for danger, `focusFirstUnderfundedEnvelope` for info).
- The AlertBar does not navigate, open modals, or dismiss itself. Any such behavior lives in the caller's `onPress`.
- The bar is **not** itself clickable — only the action button is interactive. Tapping the message, amount, or icon does nothing.
- When the underlying budget totals change, the parent re-evaluates the render condition (§4): it may swap variant (`danger` ↔ `info`), update the `amount`, or unmount the bar entirely (on reaching balance). The AlertBar reflects whatever props it is given on each render; it holds no stale amount.
- There is no dismiss action in v1.0 (see §3 future consideration). The only way the bar disappears is for the budget to become balanced.

---

## 7. Accessibility

- **Role:** The AlertBar conveys an important status change and SHOULD expose it to assistive tech. The `danger` variant uses `role="alert"` (assertive — overspending is urgent). The `info` variant uses `role="status"` (polite — informational, non-urgent).
- **Accessible name / content:** The announced text combines the message and the formatted amount, e.g. `You have assigned more than you have, 600.000đ`. The amount must be announced as a readable VND figure, not raw digits (see §9). If the message is visually truncated, the full text is still present for screen readers.
- **Icon:** The status icon is decorative (the variant meaning is carried by the text) and is hidden from assistive tech (`aria-hidden`). Color alone is never the sole signal — the message text always states the condition.
- **Action button:** A real button element with an accessible label equal to `action.label` (`Fix This` / `Assign Now`). It is keyboard-focusable, activatable with `Enter` / `Space`, and shows a visible focus ring (`focus-visible`). On native platforms it is a touch target of at least 44×44 pt.
- **Contrast:** Text and the action button meet WCAG AA contrast against the variant's subtle background (`color.bg.dangerSubtle` / `color.bg.infoSubtle`).
- **Live-region behavior:** Because the bar can appear, swap variant, or update its amount, the parent should mount it within a region whose role (`alert` / `status`) triggers an announcement when it appears or its content changes.

---

## 8. Design Tokens

Tokens reference `docs/Theme.md` (canonical source — colors, typography, spacing). Where a token is not yet finalized in the theme it is marked TBD.

| Element | Token |
|---|---|
| Bar background — `danger` | `color.bg.dangerSubtle` |
| Bar background — `info` | `color.bg.infoSubtle` |
| Message text — `danger` | `color.text.danger` |
| Message text — `info` | `color.text.info` |
| Amount text — `danger` | `color.text.danger` |
| Amount text — `info` | `color.text.info` |
| Icon — `danger` | `color.icon.danger` |
| Icon — `info` | `color.icon.info` |
| Action button background — `danger` | `color.bg.danger` |
| Action button text — `danger` | `color.text.onDanger` |
| Action button background — `info` | `color.bg.info` |
| Action button text — `info` | `color.text.onInfo` |
| Action button focus ring | `color.border.active` |
| Bar border / hairline | `color.border.subtle` (TBD) |
| Horizontal padding | `space.4` (TBD) |
| Vertical padding | `space.2` (TBD) |
| Icon ↔ message gap | `space.2` (TBD) |
| Amount ↔ action gap | `space.3` (TBD) |
| Corner radius | `radius.md` (TBD) |

---

## 9. Content / Formatting

All monetary values are VND integers (no decimals), formatted per the Vietnamese locale conventions shared with the Budget Overview screen.

| Value type | Format | Example |
|---|---|---|
| Amount (VND) | Dot-separated thousands, `đ` suffix | `600.000đ` |
| Amount — always positive | The `amount` prop is always rendered positive; `danger` callers pass the **absolute value** of the overspent amount | `600.000đ` (not `-600.000đ`) |
| Large amounts (≥ 1 billion) | Same convention as Budget Overview (§9): abbreviated `{N}B` for exact billions, else full dot-separated format | `2B` / `1.200.000.000đ` |

### Standard copy

| Variant | Message | Action label |
|---|---|---|
| `danger` | `You have assigned more than you have` | `Fix This` |
| `info` | `You have money left to assign` | `Assign Now` |

> Copy is supplied by the caller via `message` and `action.label`; the strings above are the canonical defaults used by the Budget Overview screen. The amount is always displayed as a separate, positive VND figure — it is not interpolated into the message string.

---

## 10. Platform Notes

### Web Desktop
- Renders as a full-width bar in the page header, below the month navigator (see Budget Overview §7.2). Icon leading, action button trailing on the same line.

### Web Mobile / Narrow
- Full-width bar. If the message, amount, and action cannot fit on one line, the action button may wrap to a second line beneath the message+amount while keeping the bar as a single visual block. Message truncates before the amount is clipped.

### Android / iOS
- Renders inline in the header area (not a toast or snackbar — the condition is persistent, not transient). The action button is a native button meeting the 44×44 pt minimum touch target.
- The status announcement (`role="alert"` / `role="status"` equivalents) maps to the platform accessibility API so the alert is read when it appears.

---

## 11. Test IDs

| Element | data-testid | Description |
|---|---|---|
| Bar root (generic) | `alert-bar` | Root element of the AlertBar regardless of variant. Always present when the bar is rendered. |
| Bar root — danger | `alert-bar-danger` | Present (in addition to / in place of the generic id, per the screen contract) when `variant = 'danger'`. |
| Bar root — info | `alert-bar-info` | Present when `variant = 'info'`. |
| Message | `alert-bar-message` | The message text element. |
| Amount | `alert-bar-amount` | The formatted VND amount element. |
| Action button | `alert-bar-action` | The trailing action button (`Fix This` / `Assign Now`). |
| Icon | `alert-bar-icon` | The leading status icon (decorative). |

> The Budget Overview screen references both the generic `alert-bar` id (its §4) and the per-variant ids `alert-bar-danger` / `alert-bar-info` (its §13). Implementations should expose the generic id on the root and the matching variant id so both test selectors resolve.

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | 1.0 | Initial spec | Finance App Team |
