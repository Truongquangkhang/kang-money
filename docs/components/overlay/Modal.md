# Component: Modal

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-27
**Category:** overlay

> A centered, focus-trapped confirmation dialog over a dimming scrim, used for blocking decisions such as confirming a destructive action.

---

## 1. Purpose

Modal is a general-purpose overlay dialog that interrupts the current flow to ask the user to confirm or cancel an action. It renders a dimming scrim over the screen and a centered dialog card containing a title, body content, and one or two action buttons.

Modal is used by the **Budget Overview** screen (`docs/screens/BudgetOverview.md`, §4 Modal) as the **Delete-envelope confirmation dialog**, opened from the Detail Panel **Delete** button. It supports two shapes of that dialog:

- **Envelope has transactions** — the user cannot delete directly; the primary action redirects them to reassign transactions first.
- **Envelope has no transactions** — the primary action is the destructive Delete itself, rendered with the `danger` variant.

Modal is **blocking**: while open, the underlying screen cannot be interacted with. It does not own application data; the caller supplies all content and action handlers.

---

## 2. Anatomy

```
+----------------------------------------------------------+
|                                                          |
|   scrim (color.bg.overlay) — dims & blocks the screen    |
|                                                          |
|        +--------------------------------------+          |
|        |  Delete envelope "Groceries"?    [×] |  <- title (aria-labelledby)
|        |--------------------------------------|          |
|        |                                      |          |
|        |  This envelope has 12 transactions.  |  <- body (string | node)
|        |  Please reassign them to another     |          |
|        |  envelope before deleting.           |          |
|        |                                      |          |
|        |--------------------------------------|          |
|        |        [ Cancel ]  [ Primary ]       |  <- actions (secondary | primary)
|        +--------------------------------------+          |
|              ^ dialog card (color.bg.surface,            |
|                color.shadow.md)                          |
+----------------------------------------------------------+
```

| Part | Required | Notes |
|---|---|---|
| Overlay scrim | Yes | Full-screen dimming layer (`color.bg.overlay`); click closes when `dismissible` |
| Dialog card | Yes | Centered surface card holding all content; carries the elevation shadow |
| Title | Yes | Single line; names the dialog and labels it for assistive tech |
| Close affordance (`×`) | Optional | Rendered only when `dismissible`; behaves as `onClose` |
| Body | Yes | Plain string or arbitrary node; the explanation / consequence text |
| Primary action button | Optional | Right-most; `default` or `danger` variant |
| Secondary action button | Optional | Left of primary; always the safe/cancel choice |

> If neither action is provided the dialog can only be dismissed via the close affordance, scrim, or Escape (and only when `dismissible`).

---

## 3. Props

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| `open` | `boolean` | Yes | `false` | Whether the modal is mounted/visible. Caller-controlled. |
| `title` | `string` | Yes | — | Dialog heading. Labels the dialog (`aria-labelledby`). e.g. `Delete envelope "Groceries"?` |
| `body` | `string \| node` | Yes | — | Body content. A string is rendered as a paragraph; a node is rendered as-is. |
| `primaryAction` | `{ label: string, onPress: () => void, variant?: 'default' \| 'danger' }` | No | `undefined` | Right-most action button. `variant: 'danger'` maps to the danger button tokens. Omit for a body-only / dismiss-only dialog. |
| `secondaryAction` | `{ label: string, onPress: () => void }` | No | `undefined` | Secondary button (left of primary). Always non-destructive. Typically `Cancel`. |
| `onClose` | `() => void` | Yes | — | Called when the dialog is dismissed by any means (close affordance, scrim, Escape). Alias: `onDismiss`. |
| `onDismiss` | `() => void` | No | `onClose` | Optional alias for `onClose`; provided for caller convenience. If both are set, `onClose` wins. |
| `dismissible` | `boolean` | No | `true` | When `true`, scrim-click, Escape, and the `×` affordance close the modal. When `false`, the dialog can only be resolved via its action buttons. |
| `size` | `'sm' \| 'md' \| 'lg'` | No | `'sm'` | Dialog card max-width. Confirmation dialogs use `sm`. |

> **Action `variant`.** `primaryAction.variant` defaults to `default`. Setting it to `danger` renders the button with the danger button tokens (`color.bg.danger` / `color.text.onDanger`) to signal an irreversible/destructive outcome. `secondaryAction` has no variant — it is always the safe choice.

### Budget Overview binding (the contract)

Used as the **Delete envelope** confirmation. Bindings depend on whether the selected envelope has transactions (`N` = transaction count, `{envelopeName}` = selected envelope name).

| Prop | Envelope has transactions (`N > 0`) | Envelope has no transactions (`N = 0`) |
|---|---|---|
| `title` | `Delete envelope "{envelopeName}"?` | `Delete envelope "{envelopeName}"?` |
| `body` | `This envelope has {N} transactions. Please reassign them to another envelope before deleting.` | `This action cannot be undone.` |
| `primaryAction` | `{ label: "Reassign Transactions", onPress: openTransactionReassignFlow }` | `{ label: "Delete", onPress: deleteEnvelope, variant: "danger" }` |
| `secondaryAction` | `{ label: "Cancel", onPress: closeModal }` | `{ label: "Cancel", onPress: closeModal }` |
| `onClose` | `closeModal` | `closeModal` |

> When the envelope has transactions, the primary action is **not** destructive — it routes the user to the reassignment flow (default variant). The actual deletion only becomes available once transactions have been reassigned. When the envelope has no transactions, the primary action is the destructive **Delete** itself (`danger` variant).

---

## 4. Variants

| Variant | Driven by | Description |
|---|---|---|
| Confirm (default) | `primaryAction.variant` omitted or `'default'` | Standard two-button confirm/cancel. Primary uses the default button tokens. Used for the *Reassign Transactions* shape. |
| Destructive | `primaryAction.variant = 'danger'` | Primary button uses danger tokens. Used for the direct *Delete* shape. Initial focus is placed on the **secondary** (safe) action — see §7. |
| Dismiss-only | no `primaryAction` / no `secondaryAction` | Body-only informational dialog; resolved by scrim/Escape/`×`. Requires `dismissible: true`. |

> Size is orthogonal to variant: any variant may be `sm` / `md` / `lg`. Confirmation dialogs use `sm`.

---

## 5. States

| State | Trigger | Behavior |
|---|---|---|
| Closed | `open = false` | Not rendered; scrim absent; underlying screen fully interactive. |
| Opening | `open` flips `false → true` | Scrim fades in; dialog card animates in; focus moves into the dialog (see §7). |
| Open | `open = true` | Scrim blocks the screen; focus trapped within the dialog. |
| Dismissing | scrim-click / Escape / `×` (when `dismissible`) | Calls `onClose`; dialog animates out; focus returns to the trigger element. |
| Action pending (optional) | host sets a pending flag during async `onPress` | Action buttons may render disabled/busy while the host resolves the action. Modal itself does not own async state; the host re-renders `open`/content. |

> Modal does not self-close on action press. Pressing a primary/secondary button calls the supplied `onPress`; it is the host's responsibility to set `open = false` (e.g. `closeModal`) as part of that handler.

---

## 6. Interaction Rules

- **Primary button** fires `primaryAction.onPress`. Modal does not auto-close — the host controls `open`.
- **Secondary button** fires `secondaryAction.onPress` (typically `closeModal`).
- **Escape key** fires `onClose` **only when `dismissible = true`**. When `dismissible = false`, Escape is ignored.
- **Scrim click** (clicking outside the dialog card) fires `onClose` **only when `dismissible = true`**. Clicks inside the dialog card never close it.
- **Close affordance (`×`)** is rendered only when `dismissible = true`; clicking it fires `onClose`.
- **Focus trap:** while open, Tab / Shift+Tab cycle only through focusable elements inside the dialog; focus cannot escape to the underlying screen.
- **Focus return:** on close, focus returns to the element that opened the modal (the Detail Panel **Delete** button, `data-testid` `envelope-delete-btn`).
- **Single modal:** only one Modal is open at a time. Opening a new modal requires the previous one to close.
- **Body scroll lock:** while open, scrolling of the underlying screen is locked; only the dialog body scrolls if its content overflows.

---

## 7. Accessibility

- Root dialog element uses `role="dialog"` and `aria-modal="true"`.
- The dialog is labelled by its title via `aria-labelledby` pointing at the title element (`modal-title`). If `body` is a plain string, it may additionally be referenced via `aria-describedby` (`modal-body`).
- **Focus trap** is mandatory while open: Tab order is confined to the dialog's focusable elements.
- **Initial focus** is placed on the **safest action**:
  - Destructive dialogs (`primaryAction.variant = 'danger'`) → initial focus on the **secondary** action (`Cancel`), never on the destructive button.
  - Non-destructive dialogs → initial focus on the **primary** action.
  - Dismiss-only dialogs → initial focus on the close affordance, or the dialog container if none.
- **Escape to dismiss** when `dismissible = true` (mirrors §6).
- **Focus return** to the triggering element on close (mirrors §6).
- The scrim is `aria-hidden` and not focusable; it is not announced as content.
- The danger button must not rely on color alone — its label text (`Delete`) conveys the destructive intent.

---

## 8. Design Tokens

References `docs/Theme.md` for canonical token values.

| Element | Token |
|---|---|
| Overlay scrim | `color.bg.overlay` |
| Dialog card background | `color.bg.surface` |
| Dialog card elevation shadow | `color.shadow.md` |
| Title text | `color.text.primary` |
| Body text | `color.text.secondary` |
| Close affordance icon | `color.icon.secondary` |
| Primary button (default variant) background | `color.bg.primary` |
| Primary button (default variant) text | `color.text.onPrimary` |
| Primary button (`danger` variant) background | `color.bg.danger` |
| Primary button (`danger` variant) text | `color.text.onDanger` |
| Secondary button background | `color.bg.surface` |
| Secondary button border | `color.border.default` |
| Secondary button text | `color.text.primary` |
| Focused control outline | `color.border.active` |

---

## 9. Content / Formatting

- **Title** is a single line ending in a question mark for confirmations: `Delete envelope "{envelopeName}"?`. The envelope name is wrapped in straight double quotes and interpolated verbatim (not truncated in the title).
- **Body** is a short, plain statement of consequence:
  - With transactions: `This envelope has {N} transactions. Please reassign them to another envelope before deleting.` — `{N}` is the transaction count as a plain integer (no `đ`, no thousands separators; it is a count, not money).
  - Without transactions: `This action cannot be undone.`
- **No monetary values** appear in this dialog. The app is VND-only; where money does appear in a Modal body (host-supplied node), it must follow the VND formatting rules in `docs/screens/BudgetOverview.md` §9 (dot-separated thousands, `đ` suffix).
- **Action labels** are short verb phrases in Title Case: `Reassign Transactions`, `Delete`, `Cancel`. The secondary action is `Cancel` for this dialog.
- Body strings render as a single paragraph; node bodies are the host's responsibility to format.

---

## 10. Platform Notes

### Web Desktop
- Centered dialog card over a full-viewport scrim. `size` controls max-width; the card never exceeds the viewport with margin.
- Scrim-click closes when `dismissible`.

### Web Mobile / Narrow
- Dialog card is full-width with side margins; remains centered (not a bottom sheet) so confirmation copy stays in view above the keyboard-free area.
- Body scrolls within the card if content overflows; action buttons stay pinned at the bottom of the card.

### Android / iOS
- Renders as a native-style centered alert dialog over the system scrim. Action buttons stack per platform convention; the destructive (`danger`) primary uses the platform destructive styling mapped to the danger tokens.
- Hardware/system back gesture is treated as Escape — closes only when `dismissible = true`.

---

## 11. Test IDs

| Element | data-testid | Description |
|---|---|---|
| Dialog root | `modal` | The `role="dialog"` container (the dialog card) |
| Title | `modal-title` | Title text element (target of `aria-labelledby`) |
| Body | `modal-body` | Body content region (target of `aria-describedby` when body is a string) |
| Primary action button | `modal-primary` | `primaryAction` button (e.g. `Reassign Transactions` / `Delete`) |
| Secondary action button | `modal-secondary` | `secondaryAction` button (e.g. `Cancel`) |
| Overlay scrim | `modal-scrim` | The dimming layer; click target for scrim-dismiss |
| Close affordance | `modal-close` | The `×` button; rendered only when `dismissible` |

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | 1.0 | Initial spec | Finance App Team |
