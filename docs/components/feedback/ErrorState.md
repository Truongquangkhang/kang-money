# Component: ErrorState

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-27
**Category:** feedback

> Full content-area error display shown when a screen's initial data fetch fails, surfacing a retry action the parent wires to re-trigger the fetch.

---

## 1. Purpose

`ErrorState` renders the recovery surface a screen shows when its initial data fetch fails. It replaces the entire content area (skeleton, table, panels — everything) with a single, focused block: an icon, a title, an explanatory message, and a primary retry action.

The component is **presentational and stateless**. It owns no fetch logic, no retry timer, and no error classification. The parent screen decides *when* to render it (typically in its `Error` state) and supplies the copy and the retry handler. `ErrorState` only displays what it is given and calls back when the user asks to retry.

Its canonical consumer is the **Budget Overview** screen (`docs/screens/BudgetOverview.md`), which renders `ErrorState` during its `Error` state — entered on event `ApiErrorReceived` when the initial budget data fetch fails. Pressing the retry action is what moves Budget Overview from `Error` back to `Loading` to re-fetch.

---

## 2. Anatomy

```
+-------------------------------------------------------+
|                                                       |
|                         (!)                  <- icon  |
|                                                       |
|              Couldn't load your budget       <- title |
|                                                       |
|     We couldn't reach the server. Check your          |
|     connection and try again.              <- message |
|                                                       |
|                  [  Try Again  ]           <- action  |
|                                                       |
+-------------------------------------------------------+
```

Vertical stack, centered within the content area. Top to bottom: **icon** → **title** → **message** → **retry action**. The action is the only interactive element and the default focus target.

---

## 3. Props

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| `title` | `string` | Yes | — | Short headline naming what failed (e.g. `Couldn't load your budget`). Rendered as the heading. |
| `message` | `string` | Yes | — | One- or two-sentence explanation and suggested next step. Plain text; no markup. |
| `action` | `{ label: string, onPress: () => void }` | No | `{ label: "Try Again", onPress: undefined }` | Primary retry action. `onPress` is invoked when the user activates the button; the parent wires it to re-trigger the data fetch. When omitted, no button is rendered (display-only error). |
| `icon` | `ReactNode \| string` | No | Negative alert glyph (`!` in a circle) | Optional leading icon. Pass a node to override; pass `null` to suppress the icon entirely. |
| `variant` | `'inline' \| 'fullscreen'` | No | `'fullscreen'` | Layout density. `fullscreen` fills the content area and centers vertically; `inline` renders a compact block in normal flow (e.g. inside a card or popover). |

> The Budget Overview screen renders this component with `variant = 'fullscreen'` and `action.label = "Try Again"`, binding `action.onPress` to the dispatch that re-fetches budget data for the current month.

---

## 4. Variants

| Variant | Use | Layout |
|---|---|---|
| `fullscreen` (default) | Initial fetch failure that blanks the whole screen — the Budget Overview `Error` state | Fills the content area; the icon/title/message/action stack is centered both horizontally and vertically. |
| `inline` | A localized failure inside an already-rendered surface (e.g. a popover or sub-panel could not load) | Compact, top-aligned block in normal document flow; no vertical centering, reduced spacing. |

Both variants use identical anatomy and the same retry behavior; only sizing, spacing, and vertical alignment differ.

---

## 5. States

| State | Trigger | Appearance |
|---|---|---|
| `default` | Component mounted with a `title`, `message`, and an `action` | Icon · title · message · enabled retry button. The retry button receives focus on mount. |
| `no-action` | Rendered without an `action` prop | Icon · title · message only. No button; nothing receives focus. |
| `retry-pending` | After `action.onPress` fires, while the parent transitions away (e.g. Budget Overview `Error → Loading`) | The parent is expected to unmount `ErrorState` and render its loading view. `ErrorState` does not show its own spinner; it stays visible (button may be momentarily disabled by the parent) until replaced. |

> `ErrorState` has no internal lifecycle. "Retry-pending" describes the brief window owned by the parent between the press and the unmount, not an internal state the component manages.

---

## 6. Interaction Rules

- Activating the retry action calls `action.onPress`. `ErrorState` performs no work of its own — it neither re-fetches nor changes its own props.
- In Budget Overview, `action.onPress` is bound to the dispatch that takes the screen from `Error` to `Loading` (re-fetch). On the next mount cycle the screen renders its `Loading` view and `ErrorState` is unmounted.
- The retry button is activatable by click/tap and by keyboard (`Enter` and `Space`).
- When `action` is omitted, the component is display-only and has no interactive elements.
- The component does not auto-retry, does not poll, and does not debounce. Rapid repeated presses are the parent's responsibility to guard (the parent should disable re-entry once a re-fetch is in flight).
- Resizing or switching `variant` does not change behavior — only layout.

---

## 7. Accessibility

- The root container has `role="alert"` and `aria-live="assertive"` so assistive technology announces the failure immediately when it appears.
- On mount, focus moves to the retry button so a keyboard or screen-reader user can recover without hunting for the control. When `action` is omitted, focus moves to the alert container instead (made focusable via `tabindex="-1"`).
- The retry button is a real `<button>` (or platform equivalent), keyboard-activatable with `Enter` and `Space`, and reachable in the tab order.
- The `title` is rendered as a heading and is referenced by the alert region (`aria-labelledby`); the `message` is referenced via `aria-describedby`.
- The icon is decorative — marked `aria-hidden="true"` — because the title and message already convey the meaning. Color is never the sole signal: the failure is conveyed by text, not only by the negative color token.
- The retry button has an accessible name equal to `action.label` (e.g. "Try Again"). If `action.label` is non-descriptive, supply an `aria-label` via the parent.

---

## 8. Design Tokens

Tokens reference `docs/Theme.md`.

| Element | Token |
|---|---|
| Container background | `color.bg.page` (fullscreen) / `color.bg.surface` (inline) |
| Icon color | `color.icon.secondary` |
| Title text | `color.text.primary` |
| Message text | `color.text.secondary` |
| Negative accent (icon emphasis / leading rule) | `color.text.negative` |
| Retry button background | `color.bg.primary` |
| Retry button label | `color.text.onPrimary` |
| Retry button focus ring | `color.border.active` |
| Vertical stack gap | `space.md` |
| Container padding (fullscreen) | `space.xl` |
| Container padding (inline) | `space.md` |

> The icon uses `color.icon.secondary` as its base; `color.text.negative` is reserved for emphasizing the error semantics (e.g. tinting the alert glyph) so the surface reads as a failure rather than a neutral empty state.

---

## 9. Content / Formatting

- **Title:** sentence case, short, names what failed from the user's point of view — `Couldn't load your budget`, not `HTTP 500`. No trailing period.
- **Message:** one or two plain sentences. State what happened and what to do next — e.g. `We couldn't reach the server. Check your connection and try again.` No technical error codes, stack traces, or raw API messages in user-facing copy.
- **Action label:** an imperative verb phrase — `Try Again` (default) or `Retry`. Keep it under ~3 words.
- The component renders no monetary values, so VND formatting rules (`docs/screens/BudgetOverview.md` §9) do not apply here. Any amounts referenced in copy must still follow the app's VND formatting (dot-separated thousands, `đ` suffix) if a future message ever includes one.
- All copy is supplied by the parent; `ErrorState` ships no hardcoded strings except the default `action.label` (`Try Again`).

---

## 10. Platform Notes

### Web Desktop
- `fullscreen` fills the screen's content area (within the max-width, side-padded container) and centers the stack vertically.

### Web Mobile / Narrow
- `fullscreen` fills the full-width content column; the stack stays horizontally centered. Action button stretches to a comfortable tap width with adequate vertical padding.

### Android / iOS
- The retry control is a native-styled primary button meeting the platform minimum touch target (≥ 44pt). Focus management maps to the platform accessibility focus (VoiceOver / TalkBack lands on the alert, then the retry control).
- The `role="alert"` / `aria-live="assertive"` semantics map to the platform live-region equivalent so the failure is announced on appearance.

---

## 11. Test IDs

| Element | data-testid | Description |
|---|---|---|
| Root container | `error-state` | The error display itself. Budget Overview renders this inside its `budget-overview-error` root wrapper. |
| Icon | `error-state-icon` | Leading alert glyph (decorative). |
| Title | `error-state-title` | Heading text from the `title` prop. |
| Message | `error-state-message` | Explanatory text from the `message` prop. |
| Retry action | `error-state-action` | Primary retry button; activating it calls `action.onPress`. Absent when `action` is omitted. |

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | 1.0 | Initial spec | Finance App Team |
