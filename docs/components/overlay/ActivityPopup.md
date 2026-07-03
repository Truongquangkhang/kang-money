# Component: ActivityPopup

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-27
**Category:** overlay

> A popover that lists the transactions for a selected envelope-month, opened from a non-zero Activity cell in the Budget Overview envelope table.

---

## 1. Purpose

The Activity Popup is a read-and-route overlay. When the user clicks an Activity cell with a non-zero value in the Budget Overview envelope table (`docs/screens/BudgetOverview.md`), this component lists every transaction assigned to that envelope for the active budget month, sorted by date descending.

It exists to answer one question — **"what made up this number?"** — without leaving the Budget Overview screen. Transaction history is not a primary view on Budget Overview; the Activity Popup is the only path to it from that screen.

The popup is a **router, not an editor**: it displays transactions but does not edit them inline. Tapping a transaction row hands off to `TransactionForm` (`docs/screens/TransactionForm.md`) via `onSelectTransaction`. The popup owns its own presentation, sorting, and dismissal; the parent owns data fetching and selection state.

---

## 2. Anatomy

```
+--------------------------------------------------------------+
|  Activity                                            [Close]  |  ← title · close button
|  Bills                                                        |  ← subtitle (category group name)
+--------------------------------------------------------------+
|  Account | Date       | Payee        | Memo        | Amount   |
|----------|------------|--------------|-------------|----------|
|  VCB     | 27/06/2026 | EVN HCMC     | Electric... | -850.000đ|
|  MoMo    | 24/06/2026 | Internet FPT | Monthly fee | -250.000đ|
|  Cash    | 18/06/2026 | Refund       | Overpaid    | +50.000đ |
+--------------------------------------------------------------+

Empty state (transactions = [], not loading):

+--------------------------------------------------------------+
|  Activity                                            [Close]  |
|  Bills                                                        |
+--------------------------------------------------------------+
|                                                              |
|              No transactions this month                      |  ← muted, centered
|                                                              |
+--------------------------------------------------------------+
```

| Element | Content |
|---|---|
| Title | `Activity` |
| Subtitle | Category group name of the selected envelope (e.g. `Bills`) |
| Transaction table | 5 columns — Account · Date · Payee · Memo · Amount |
| Close button | `Close` — dismisses the popup |
| Empty state | `No transactions this month`, muted and centered |

---

## 3. Props

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| `envelopeId` | `string` | Yes | — | ID of the envelope whose transactions are shown. Identifies which envelope-month the popup represents. |
| `month` | `YYYY-MM` | Yes | — | The active budget month for the listed transactions. |
| `transactions` | `Transaction[]` | Yes | `[]` | Transaction list for this envelope-month. Parent passes `[]` on open and populates it when the fetch resolves (see §5). The component sorts by date descending for display. |
| `onClose` | `() => void` | Yes | — | Called when the user dismisses the popup (Close button, click-outside, or Escape). Dismissing does not affect the selected envelope. |
| `onSelectTransaction` | `(transactionId: string) => void` | Yes | — | Called when a transaction row is tapped; opens `TransactionForm` for that transaction. |

> The component does not fetch data. It receives `transactions` from the parent (`activityPopup.transactions` in the Budget Overview reducer) and never mutates them.

---

## 4. Variants

The component renders the same content in three platform-driven presentation variants. Behavior and props are identical across variants; only positioning and dismissal affordances differ (see §10).

| Variant | Platform | Presentation |
|---|---|---|
| `popover` | Web Desktop | Anchored popover positioned against the clicked Activity cell. |
| `bottom-sheet` | Android / iOS | Full-width sheet that slides up from the bottom of the screen. |
| `inline` | Web Mobile / Narrow | Stacked inline below the triggering element; no overlay, no sheet. |

The variant is selected by the host platform, not by a prop. There is no visual size variant — the table width adapts to its container in all variants.

---

## 5. States

| State | Condition | Display |
|---|---|---|
| Loading | Popup just opened; parent set `transactions = []` and is fetching (event `ActivityDataReceived` not yet received) | Header (`Activity` + subtitle) rendered; table area shows a loading placeholder in place of rows. |
| Populated | `transactions` is non-empty | Header + transaction table with one row per transaction, sorted by date descending. |
| Empty | Fetch resolved and `transactions` is empty | Header + centered muted message `No transactions this month` (`data-testid` `activity-popup-empty`). |

> Loading and Empty share an empty `transactions` array. The parent distinguishes them by fetch lifecycle: while the fetch is in flight the loading placeholder is shown; once `ActivityDataReceived` arrives with an empty list, the empty state is shown. The popup remains open and the header stays visible across all three states.

---

## 6. Interaction Rules

### Opening
- Opens only when the user clicks an Activity cell whose value is non-zero. A zero-value Activity cell does nothing (enforced by the parent — see Budget Overview §8).
- On open, the parent sets `activityPopup = { envelopeId, month, transactions: [] }` and begins fetching. The component renders the loading state until `transactions` is populated.

### Single-instance rule
- Only one Activity Popup is open at a time.
- Clicking a second Activity cell closes the current popup and opens the new one for the newly clicked envelope-month.

### Dismissal
- Clicking the **Close** button dismisses the popup (`onClose`).
- Clicking **outside** the popup — anywhere on the screen — dismisses it (`onClose`).
- Pressing **Escape** dismisses it (`onClose`).
- Dismissing the popup does **not** change `selectedEnvelopeId` or otherwise affect the selected envelope / Detail Panel.

### Row activation
- Each transaction row is tappable. Activating a row calls `onSelectTransaction(transactionId)`, which navigates to `TransactionForm` for that transaction.
- Activating a row does not implicitly close the popup via `onClose`; navigation away is the parent's responsibility.

### Sorting
- Transactions are always displayed sorted by **date descending** (most recent first), regardless of the order in which the parent supplies them.

---

## 7. Accessibility

- **Role:** The popup container uses `role="dialog"` with `aria-modal` appropriate to the variant (modal for bottom-sheet; non-modal popover on desktop). The transaction list uses `role="listbox"` with each transaction row as `role="option"` (or an equivalent `list` / `listitem` + button pattern) so rows are announced as an activatable collection.
- **Labelling:** The dialog is labelled by its title (`Activity`) and described by its subtitle (the category group name) via `aria-labelledby` / `aria-describedby`.
- **Focus management:** On open, focus moves into the popup (Close button or first transaction row). Focus is trapped within the popup while it is open in the modal variants. On close, focus returns to the Activity cell that triggered it.
- **Escape:** `Escape` closes the popup from anywhere within it.
- **Keyboard activation:** Each transaction row is reachable by keyboard (Tab / arrow keys per the listbox pattern) and activatable with `Enter` / `Space`, firing `onSelectTransaction`.
- **Amount semantics:** Amount color (negative vs. positive) is not the sole indicator — the leading `-` sign and the value itself convey expense vs. income for non-color-perceiving users.
- **Empty state:** The empty message is in the accessibility tree and announced when the popup resolves to empty.

---

## 8. Design Tokens

Reference: `docs/Theme.md`.

| Element | Token |
|---|---|
| Popup background | `color.bg.overlay` |
| Popup shadow | `color.shadow.md` |
| Title text | `color.text.primary` |
| Subtitle text | `color.text.secondary` |
| Table header text | `color.text.secondary` |
| Amount — negative (expense) | `color.text.negative` |
| Amount — positive (income / adjustment) | `color.text.positive` |
| Close button | `color.icon.secondary` / `color.text.secondary` |
| Empty state text | `color.text.secondary` |

---

## 9. Content / Formatting

All monetary values are VND integers (no decimals), formatted in Vietnamese locale (consistent with Budget Overview §9).

### Transaction row columns

| Column | Content | Formatting |
|---|---|---|
| Account | Source account name | Plain text, e.g. `VCB`, `MoMo`, `Cash` |
| Date | Transaction date | `DD/MM/YYYY` — e.g. `27/06/2026` |
| Payee | Payee / merchant name | Plain text |
| Memo | User memo or imported description | Plain text, **truncated at 40 chars** (ellipsis appended) |
| Amount | Transaction amount in VND | See amount formatting below |

### Amount formatting

| Value type | Format | Example | Color |
|---|---|---|---|
| Expense (negative) | Leading `-`, dot-separated thousands, `đ` suffix | `-850.000đ` | `color.text.negative` |
| Income / positive adjustment | Dot-separated thousands, `đ` suffix | `50.000đ` | `color.text.positive` |

### Fixed strings

| Element | String |
|---|---|
| Title | `Activity` |
| Close button | `Close` |
| Empty state | `No transactions this month` |

> Memo truncation is display-only — the full memo is preserved in the underlying transaction and shown in `TransactionForm`.

---

## 10. Platform Notes

### Web Desktop
- Renders as a **popover** anchored to the Activity cell that triggered it.
- Click-outside dismisses; the popover is positioned relative to the cell.

### Android / iOS
- Renders as a **bottom sheet** — full width, sliding up from the bottom of the screen.
- Tapping outside the sheet (on the scrim) dismisses it.

### Web Mobile / Narrow
- Renders **inline**, stacked directly below the triggering element — **not** a bottom sheet and not an overlay.
- The surrounding page flows around it; dismissal still occurs via Close, Escape, or interacting outside the inline region.

> Across all platforms the content (title, subtitle, 5-column table, empty state) and prop contract are identical; only the presentation container differs.

---

## 11. Test IDs

| Element | data-testid | Description |
|---|---|---|
| Popup container | `activity-popup` | Root popover / bottom sheet / inline container |
| Transaction row | `activity-row-{transactionId}` | Per-transaction row; activating it fires `onSelectTransaction` |
| Close button | `activity-close` | `Close` button |
| Empty state | `activity-popup-empty` | `No transactions this month` message |

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | 1.0 | Initial spec | Finance App Team |
