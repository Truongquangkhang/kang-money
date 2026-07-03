# Screen: Budget Overview

**Status:** In Design
**Version:** 1.1
**Last Updated:** 2026-06-27
**Assigned To:** Finance App Team

> **Shared Budget (Add Member) — Not in scope.** Multi-user / shared budget features are excluded from the current release. The "Add Member" button is not rendered. All budget data is single-user only.

> **Target & Auto-Assign — Not in scope.** Per-envelope spending targets and auto-assign suggestions are excluded from the current release. The Detail Panel does not render a Target section or Auto-Assign section.

> **Multi-currency — Not in scope.** The app supports VND only. All monetary values are stored, computed, and displayed in VND. No currency selector is rendered anywhere on this screen.

> This spec is the target behavior contract for the Budget Overview screen of the personal expense management app.

---

## 1. Purpose

The Budget Overview screen is the central screen of the envelope budgeting app. It lets users allocate their available money across spending envelopes (hũ) before they spend — every đồng must be assigned to exactly one envelope. The screen owns the full monthly budget view as two coordinated panels: **(left)** the envelope list for the current month, **(right)** the detail and management panel for the selected envelope.

The right-hand Detail Panel is **selection-driven**: on load nothing is selected and the panel shows an empty-state hint. Clicking an envelope row activates the panel for that envelope; the panel always reflects the currently selected envelope.

This screen is for **budget allocation and envelope management only**. Transaction history is accessible via the Activity Popup (§7.3), not as a primary view on this screen.

---

## 2. Dependencies

| Document | Purpose |
|---|---|
| `docs/Theme.md` | Canonical design tokens — colors, typography, spacing |
| `docs/components/feedback/AlertBar.md` | Overspent / unassigned money warning banner |
| `docs/components/feedback/LoadingState.md` | Skeleton display during initial data load |
| `docs/components/feedback/ErrorState.md` | Content-area error display during `Error` state |
| `docs/components/feedback/EmptyState.md` | Empty envelope list and empty filter result states |
| `docs/components/input/InlineEdit.md` | Inline rename of envelope name in Detail Panel |
| `docs/components/overlay/Modal.md` | Confirmation dialog for destructive actions (Delete envelope with transactions) |
| `docs/components/overlay/ActivityPopup.md` | Popover listing transactions for a selected envelope-month |
| `docs/components/data/EnvelopeTable.md` | The three-column (Assigned / Activity / Available) scrollable budget table |
| `docs/screens/TransactionForm.md` | Destination when user taps a transaction row inside the Activity Popup |

> **Envelope (hũ)** is the product term for a budget category. "Category" and "envelope" are used interchangeably in this spec; the UI renders "hũ".

---

## 3. Inputs

N/A — top-level screen that owns its own data fetch; no parent caller injects props.

---

## 4. Component Usage

### AlertBar — Overspent warning

| Prop | Value |
|---|---|
| `variant` | `danger` |
| `amount` | Absolute value of the overspent amount (e.g. `600.000đ`) |
| `message` | `You have assigned more than you have` |
| `action` | `{ label: "Fix This", onPress: triggerRebalanceFlow }` |

Shown in the header area when `totalAssigned > totalAvailableMoney`. Hidden when the budget is balanced. `data-testid`: `alert-bar`.

### AlertBar — Unassigned money notice

| Prop | Value |
|---|---|
| `variant` | `info` |
| `amount` | Unassigned amount |
| `message` | `You have money left to assign` |
| `action` | `{ label: "Assign Now", onPress: focusFirstUnderfundedEnvelope }` |

Shown when `totalAssigned < totalAvailableMoney`. Mutually exclusive with the danger variant — only one AlertBar is visible at a time.

### FilterTabs

A row of filter tabs above the envelope table. Each tab is a text label that filters the visible envelope rows.

| Tab value | Label | Envelope filter |
|---|---|---|
| `all` | All | All envelopes |
| `overspent` | Overspent | `available < 0` |
| `underfunded` | Underfunded | `available >= 0 && available < assignedLastMonth * 0.5` (heuristic — TBD) |
| `overfunded` | Overfunded | `available > assignedThisMonth * 2` (heuristic — TBD) |
| `money_available` | Money Available | `available > 0` |
| `snoozed` | Snoozed | Envelopes in `hidden` state |

`data-testid`: `filter-tab-{value}` per tab, `filter-tabs` for the row container.

### EnvelopeTable — Budget table

The main budget table. Renders group rows and envelope rows with three value columns.

| Prop | Value |
|---|---|
| `envelopes` | Filtered, grouped envelope data for the current month |
| `selectedEnvelopeId` | Currently selected envelope ID (drives row highlight and Detail Panel) |
| `onSelectEnvelope` | Called when user clicks an envelope row; sets `selectedEnvelopeId` |
| `onSelectActivity` | Called when user clicks an Activity cell with a non-zero value; opens Activity Popup |
| `onEditAssigned` | Called when user clicks an Assigned cell; activates inline amount edit |
| `month` | Current budget month (`YYYY-MM`) |

Column structure:

| Column | Header | Content | Editable |
|---|---|---|---|
| 1 | (none) | Envelope name + group label | No (edit via Detail Panel) |
| 2 | Assigned | `assignedThisMonth` formatted as VND | Yes — inline click-to-edit |
| 3 | Activity | `activityThisMonth` formatted as VND (negative = spending) | No — click opens Activity Popup |
| 4 | Available | `available` formatted as VND, colored by state | No |

`data-testid`: `envelope-table`.

### ActivityPopup — Transaction list

| Prop | Value |
|---|---|
| `envelopeId` | Selected envelope ID |
| `month` | Current budget month (`YYYY-MM`) |
| `transactions` | Transaction list for this envelope-month |
| `onClose` | Dismisses the popup |
| `onSelectTransaction` | Opens `TransactionForm` for the tapped row |

Rendered as a popover anchored to the Activity cell that triggered it. Owned by `ActivityPopup.md`; this screen provides only the prop bindings above.

Structure of each transaction row (owned by `ActivityPopup`):

| Column | Content |
|---|---|
| Account | Source account name (e.g. `VCB`, `MoMo`, `Cash`) |
| Date | Transaction date (`DD/MM/YYYY`) |
| Payee | Payee name or merchant |
| Memo | User memo or imported description (truncated at 40 chars) |
| Amount | Amount in VND (negative = expense, colored `color.text.negative`) |

`data-testid`: `activity-popup`, `activity-row-{transactionId}`, `activity-close`.

### InlineEdit — Assigned amount

| Prop | Value |
|---|---|
| `value` | Current `assignedThisMonth` for the envelope (raw integer, VND) |
| `format` | `currency-vnd` |
| `onCommit` | Persists the new assigned value; triggers re-calculation of `available` and budget balance |
| `onCancel` | Reverts to the previous value |
| `min` | `0` |

Used in: Assigned column cells. The cell switches to an input on click/tap; `Enter` or blur commits.

### InlineEdit — Envelope name (Detail Panel)

| Prop | Value |
|---|---|
| `value` | Current envelope name |
| `maxLength` | `50` |
| `onCommit` | Persists the new name |
| `onCancel` | Reverts |

Used in: Detail Panel header. The envelope name becomes an editable text field when the user clicks the name or the edit icon (✏️).

### Modal — Delete confirmation

| Prop | Value |
|---|---|
| `title` | `Delete envelope "{envelopeName}"?` |
| `body` | `This envelope has {N} transactions. Please reassign them to another envelope before deleting.` (when transactions exist) or `This action cannot be undone.` (when no transactions) |
| `primaryAction` | `{ label: "Reassign Transactions", onPress: openTransactionReassignFlow }` (when transactions exist) or `{ label: "Delete", onPress: deleteEnvelope, variant: "danger" }` |
| `secondaryAction` | `{ label: "Cancel", onPress: closeModal }` |

Opened when user presses **Delete** in the Detail Panel edit form. If the envelope has no transactions, the primary action is the destructive Delete directly.

---

## 5. Acceptance Criteria

### Month Navigation
- [ ] Header shows the current budget month as `{Month} {YYYY}` (e.g. `June 2026`)
- [ ] `←` / `→` arrows navigate to the previous / next month; the budget table and Detail Panel reload for the new month
- [ ] The month picker (tapping the month label) opens a date picker constrained to month granularity; selecting a month navigates to it

### Alert Bar
- [ ] When `totalAssigned > totalAvailableMoney`: the danger AlertBar shows the overspent amount and a `Fix This` action; no info AlertBar is shown
- [ ] When `totalAssigned < totalAvailableMoney`: the info AlertBar shows the unassigned amount; no danger AlertBar is shown
- [ ] When `totalAssigned === totalAvailableMoney`: no AlertBar is shown

### Filter Tabs
- [ ] All tabs are shown above the envelope table; `All` is active by default
- [ ] Activating a tab filters the envelope rows; the group row is hidden if all its envelopes are filtered out
- [ ] The `Snoozed` tab shows only envelopes the user has explicitly hidden; hidden envelopes do not appear in any other tab
- [ ] Switching tabs does not change the selected envelope or the Detail Panel

### Envelope Table
- [ ] One group header row per `CategoryGroup`, collapsible; collapsed groups hide all child envelope rows
- [ ] Each envelope row shows: name, `assignedThisMonth`, `activityThisMonth`, `available`
- [ ] `available` cell color: green when `> 0`, default (muted) when `= 0`, red when `< 0`
- [ ] Clicking an Assigned cell activates inline edit for that cell only; `Enter` or blur commits; `Escape` cancels
- [ ] Clicking an Activity cell with a non-zero value opens the Activity Popup for that envelope-month
- [ ] Clicking an Activity cell with a zero value does nothing
- [ ] Clicking an envelope name or row (outside the Assigned / Activity cells) selects the envelope and opens the Detail Panel

### Activity Popup
- [ ] Popup is anchored to the Activity cell that was clicked
- [ ] Lists all transactions assigned to that envelope in the current month, sorted by date descending
- [ ] Each row shows: Account, Date, Payee, Memo, Amount
- [ ] Expense amounts are displayed in red; income / positive adjustments in green
- [ ] Clicking a transaction row opens `TransactionForm` for that transaction
- [ ] Clicking outside the popup or the Close button dismisses it without affecting the selected envelope

### Detail Panel — Edit Envelope
- [ ] When an envelope is selected the Detail Panel shows the envelope name with an edit icon (✏️)
- [ ] Clicking the name or ✏️ activates inline name edit with **Save** / **Cancel** actions
- [ ] **Hide** button: hides the envelope (moves it to the `Snoozed` filter); if `available > 0` a warning is shown: `This envelope still has {amount}. The money will not be lost but the envelope will not appear this month.`
- [ ] **Delete** button: opens the Delete confirmation Modal (see §4 Modal); if the envelope has transactions the user must reassign them before deletion completes; if no transactions the deletion is immediate after confirmation
- [ ] **Cancel** button: dismisses the edit form, reverts name field, leaves the envelope unchanged

### Detail Panel — Available Balance Breakdown
- [ ] Shows a collapsible `Available Balance` section with the breakdown rows defined in §7.5
- [ ] The `Assigned This Month` row value is clickable and activates the same inline edit as the Assigned column cell
- [ ] All amounts shown in VND

### Envelope Accumulation
- [ ] `available` at end of month carries over to the next month automatically — it is never reset to 0
- [ ] An envelope that accumulates over multiple months shows `available` equal to the sum of all prior months' balances minus spending, plus the current month's assignment

### Empty States
- [ ] No envelopes: shows `EmptyState` with prompt `+ Add Category Group` and onboarding copy
- [ ] Filter returns no envelopes: shows inline `No envelopes match this filter` below the filter tabs
- [ ] Activity Popup with no transactions: shows `No transactions this month` inside the popup

---

## 6. State Machine

### Screen State

```
Loading → Ready
Loading → Error
Error   → Loading   (on retry)
```

### Reducer Fields

| Field | Type | Initial value | Description |
|---|---|---|---|
| `screenState` | `loading \| ready \| error` | `loading` | Overall screen lifecycle state |
| `currentMonth` | `YYYY-MM` | Current calendar month | Active budget month |
| `totalAvailableMoney` | `number` | `0` | Total money available to assign (sum of all account balances) |
| `totalAssigned` | `number` | `0` | Sum of `assignedThisMonth` across all envelopes |
| `envelopes` | `Envelope[]` | `[]` | All envelopes with their computed values for `currentMonth` |
| `activeFilter` | `FilterTab` | `all` | Currently active filter tab |
| `selectedEnvelopeId` | `string \| null` | `null` | ID of the envelope shown in the Detail Panel |
| `activityPopup` | `{ envelopeId, month, transactions } \| null` | `null` | State for the Activity Popup; `null` when closed |
| `detailPanelMode` | `view \| editing` | `view` | Whether the Detail Panel is in view or edit mode |

> `selectedEnvelopeId`, `activityPopup`, and `detailPanelMode` are reset when `currentMonth` changes.

### Events

| Event | Trigger | Kind |
|---|---|---|
| `ScreenOpened` | Platform mounts the screen | system |
| `BudgetDataReceived` | Initial budget data for the current month arrives | system |
| `ApiErrorReceived` | Initial data fetch fails | system |
| `UserChangedMonth` | User navigates to a different month | user |
| `MonthDataReceived` | Budget data for the new month arrives after `UserChangedMonth` | system |
| `UserSelectedEnvelope` | User clicks an envelope row | user |
| `UserOpenedActivityPopup` | User clicks a non-zero Activity cell | user |
| `ActivityDataReceived` | Transaction list for the selected envelope-month arrives | system |
| `UserClosedActivityPopup` | User closes the Activity Popup | user |
| `UserStartedEditEnvelope` | User clicks envelope name or ✏️ in Detail Panel | user |
| `UserCommittedEnvelopeName` | User saves renamed envelope | user |
| `UserHidEnvelope` | User confirms Hide in Detail Panel | user |
| `UserDeletedEnvelope` | User confirms Delete (after reassigning transactions if required) | user |
| `UserEditedAssigned` | User commits a new Assigned value for an envelope | user |

### Transitions

| Current state | Event | Next state | Notes |
|---|---|---|---|
| `Loading` | `BudgetDataReceived` | `Ready` | Populate envelopes, totals; no envelope selected |
| `Loading` | `ApiErrorReceived` | `Error` | |
| `Error` | `ScreenOpened` | `Loading` | Retry dispatches re-fetch |
| `Ready` | `UserChangedMonth` | `Loading` | Re-fetch for the new month; reset `selectedEnvelopeId`, `activityPopup`, `detailPanelMode` |
| `Ready` | `UserSelectedEnvelope` | `Ready` | Set `selectedEnvelopeId`; set `detailPanelMode = view`; close `activityPopup` |
| `Ready` | `UserOpenedActivityPopup` | `Ready` | Set `activityPopup = { envelopeId, month, transactions: [] }`; begin fetching transactions |
| `Ready` | `ActivityDataReceived` | `Ready` | Populate `activityPopup.transactions` |
| `Ready` | `UserClosedActivityPopup` | `Ready` | Set `activityPopup = null` |
| `Ready` | `UserStartedEditEnvelope` | `Ready` | Set `detailPanelMode = editing` |
| `Ready` | `UserCommittedEnvelopeName` | `Ready` | Persist new name; set `detailPanelMode = view` |
| `Ready` | `UserHidEnvelope` | `Ready` | Mark envelope `hidden`; set `selectedEnvelopeId = null` |
| `Ready` | `UserDeletedEnvelope` | `Ready` | Remove envelope from list; set `selectedEnvelopeId = null` |
| `Ready` | `UserEditedAssigned` | `Ready` | Update `assignedThisMonth` for the envelope; recompute `available` and `totalAssigned` |

> Filter tab changes, inline edit open/cancel, and modal open/close are local UI state with no reducer event. Behavior is specified in §8.

---

## 7. Display Rules

### 7.1 Layout

Max-width-constrained content area with horizontal padding. The screen is divided into two columns.

#### Web Desktop

```
Content area (max-width, side padding)
+-- Page header (full width) — Month navigator + AlertBar
|
+-- Two-column grid (~65% / ~35%, gap 24px)
    +-- Left column
    |   +-- Filter tabs row
    |   +-- Toolbar row (+ Add Category Group · Undo · Redo · toggle view)
    |   +-- EnvelopeTable
    |       +-- Group header row (collapsible)
    |       +-- Envelope row × N
    |           Columns: Name | Assigned | Activity | Available
    |
    +-- Right column (sticky, top-aligned)
        +-- Detail Panel card
            +-- Empty state: "Select an envelope to see details"
            +-- Active state (envelope selected):
                +-- Header: envelope name (inline-editable) + ✏️
                +-- Edit form (when detailPanelMode = editing):
                |   +-- Name input + [Hide] [Delete] [Cancel] [Save]
                +-- Available Balance section (collapsible)
                +-- Notes textarea
```

#### Web Mobile / Narrow

Single column. The Detail Panel renders **inline below the envelope table** (not a bottom sheet). It is empty until an envelope is selected.

```
Content area (full width)
+-- Page header — Month navigator + AlertBar
+-- Filter tabs
+-- Toolbar
+-- EnvelopeTable (full width)
+-- Detail Panel card (full width, below table)
    +-- Empty state until envelope selected
    +-- Active: header · edit form · balance breakdown · notes
```

### 7.2 Page Header

Always visible at the top of the screen.

| Element | Content |
|---|---|
| `←` arrow | Navigate to the previous month |
| Month label | `{Month} {YYYY}` — tapping opens the month picker |
| `→` arrow | Navigate to the next month |
| AlertBar | Shown below the month navigator when budget is unbalanced (see §4) |

### 7.3 Activity Popup

Opens when the user clicks a non-zero Activity cell. Positioned as a popover anchored to the clicked cell (top-aligned on desktop; bottom sheet on mobile — see §11).

| Element | Content |
|---|---|
| Title | `Activity` |
| Subtitle | Category group name (e.g. `Bills`) |
| Table columns | Account · Date · Payee · Memo · Amount |
| Close button | `Close` — dismisses the popup |

Each transaction row is tappable and navigates to `TransactionForm` for editing. Expense amounts are colored `color.text.negative`; income / adjustments are colored `color.text.positive`.

**Empty state:** `No transactions this month` (muted, centered). `data-testid`: `activity-popup-empty`.

### 7.4 Envelope Table — Row States

#### Group header row

| Element | Content |
|---|---|
| Expand/collapse | Chevron `▾` (expanded) / `▸` (collapsed) |
| Group name | Bold, primary text |
| Assigned total | Sum of `assignedThisMonth` for all envelopes in the group |
| Activity total | Sum of `activityThisMonth` for all envelopes in the group |
| Available total | Sum of `available` for all envelopes in the group |

#### Envelope row

| Element | Content |
|---|---|
| Name | Envelope name, indented beneath the group header |
| Status badge | `Fully Spent` (muted pill) when `activityThisMonth ≠ 0 && available = 0`; `Overspent {amount}` (red pill) when `available < 0` |
| Assigned cell | `assignedThisMonth` in VND; click to inline-edit |
| Activity cell | `activityThisMonth` in VND; shown in `color.text.negative` when negative; click opens Activity Popup if value ≠ 0 |
| Available cell | `available` in VND; colored by threshold (see §5 AC) |

#### Available cell color thresholds

| Condition | Color token |
|---|---|
| `available > 0` | `color.text.positive` (green) |
| `available = 0` | `color.text.secondary` (muted) |
| `available < 0` | `color.text.negative` (red) |

### 7.5 Detail Panel — Available Balance Breakdown

Shown in a collapsible section labeled `Available Balance`. Collapsed by default when first selecting an envelope; user preference persists for the session.

| Row label | Value | Notes |
|---|---|---|
| Carried Over From Last Month | `availableCarriedOver` | Sum of `available` from all prior months; may span multiple months of accumulation |
| Assigned This Month | `assignedThisMonth` | Inline-editable (same as the Assigned column cell) |
| Cash Spending | `cashSpending` | Total cash/bank/e-wallet spending this month for this envelope (negative) |
| Credit Spending | `creditSpending` | Total credit card spending this month for this envelope (negative) |

**Formula:** `available = availableCarriedOver + assignedThisMonth + cashSpending + creditSpending`

where `cashSpending` and `creditSpending` are negative numbers representing outflows.

### 7.6 Detail Panel — Edit Form

Visible when `detailPanelMode = editing`. Rendered inline in the Detail Panel header area, replacing the static name display.

| Element | Type | Behavior |
|---|---|---|
| Name input | Text field, pre-filled with current name | Editable; max 50 chars |
| **Hide** | Secondary button | Hides the envelope; shows warning if `available > 0` (see §5 AC) |
| **Delete** | Danger button | Opens Delete confirmation Modal (§4) |
| **Cancel** | Ghost button | Dismisses edit form; reverts input |
| **Save** | Primary button | Commits the new name; dismisses edit form |

**Hide** and **Delete** are always rendered regardless of whether the name has been changed. **Save** is disabled when the name field is empty.

---

## 8. Interaction Rules

### Month Navigation
- Clicking `←` / `→` fires `UserChangedMonth` with the adjacent month. Fast-clicking is debounced — a second click before the data arrives queues the next month.
- The current calendar month is the default on first load. The user can navigate to past months freely; future months beyond the current month are accessible for forward planning.

### Filter Tabs
- Clicking a tab sets `activeFilter` locally. The envelope table re-filters immediately with no network call.
- The selected envelope and Detail Panel are preserved when switching tabs. If the selected envelope is filtered out, the Detail Panel remains visible (the row just disappears from the list).

### Assigned Cell Inline Edit
- Clicking an Assigned cell opens an inline numeric input in that cell only. All other cells remain non-editable.
- Only one cell can be in edit mode at a time — clicking another Assigned cell commits the current edit first, then opens the new one.
- `Enter` or clicking outside the cell commits the new value and fires `UserEditedAssigned`.
- `Escape` cancels the edit and reverts the cell to its previous value.
- Input is VND integer only (no decimals). Comma separators are rendered during input for readability.
- Entering `0` is valid — it removes the allocation for that envelope this month.
- Negative values are not allowed; the input rejects them.

### Activity Cell Click
- Clicking an Activity cell with `activityThisMonth = 0` does nothing (no popup, no state change).
- Clicking an Activity cell with `activityThisMonth ≠ 0` fires `UserOpenedActivityPopup` and begins loading transactions for that envelope-month.
- Only one Activity Popup can be open at a time. Clicking a second Activity cell closes the current popup and opens the new one.
- Clicking outside the Activity Popup (anywhere on the screen) closes it.

### Envelope Row Selection
- Clicking an envelope name or any part of the row outside the Assigned and Activity cells selects the envelope and fires `UserSelectedEnvelope`.
- The selected row is highlighted with `color.bg.selectedRow` (design token TBD).
- Clicking the same envelope row again while it is selected does nothing (no deselect).

### Detail Panel — Edit Mode
- Clicking the envelope name or ✏️ sets `detailPanelMode = editing`.
- **Hide:** if `available > 0`, shows an inline warning message before the action completes — the warning is shown within the Detail Panel, not a modal. The user confirms by clicking **Hide Anyway** on the warning or cancels with **Cancel**.
- **Delete:** always opens the Delete confirmation Modal regardless of `available`. If the envelope has transactions, the modal body instructs the user to reassign them first (see §4 Modal).
- **Save:** commits the name change. If the name is unchanged from the original, the request is still sent (idempotent).
- **Cancel:** resets the name input to the original value and sets `detailPanelMode = view`.

### Group Row Expand / Collapse
- Clicking a group header row toggles its expanded state locally. Group collapse state persists per session (not persisted to the server).
- When a group is collapsed, its envelope rows are hidden. The group's aggregated totals remain visible in the group header row.

### Envelope Accumulation (multi-month carry-over)
- `available` for a given envelope-month is computed server-side as the running balance from the envelope's creation date through the current month.
- The screen does **not** compute carry-over locally — it renders `available` as received from the API.
- When displaying the Available Balance Breakdown (§7.5), `availableCarriedOver` represents all prior months' net balance as a single figure.

---

## 9. Number Formatting

All monetary values are VND integers (no decimals). The app uses Vietnamese locale formatting throughout.

| Value type | Format | Example |
|---|---|---|
| VND amounts (positive) | Dot-separated thousands, `đ` suffix | `1.500.000đ` |
| VND amounts (negative) | Leading `-`, dot-separated thousands, `đ` suffix | `-50.000đ` |
| VND amounts (zero) | `0đ` | `0đ` |
| Large amounts (≥ 1 billion) | Abbreviated: `{N}B` for exact billions, else full format | `2B` / `1.200.000.000đ` |
| Inline edit input | Comma-separated during typing (auto-formatted); stored as integer | `1,500,000` → stored as `1500000` |
| Percentage (utilization, etc.) | `XX%` — no decimal for whole numbers; one decimal otherwise | `70%`, `85.5%` |
| Dates in Activity Popup | `DD/MM/YYYY` | `27/06/2026` |
| Month in header | `{Month} {YYYY}` (full month name) | `June 2026` |

---

## 10. Design Tokens

| Element | Token |
|---|---|
| Page background | `color.bg.page` |
| Left panel card background | `color.bg.surface` |
| Detail Panel card background | `color.bg.surface` |
| Envelope row — default | `color.bg.surface` |
| Envelope row — selected | `color.bg.selectedRow` |
| Group header row background | `color.bg.subtleTint` |
| Available positive | `color.text.positive` |
| Available zero | `color.text.secondary` |
| Available negative | `color.text.negative` |
| AlertBar danger background | `color.bg.dangerSubtle` |
| AlertBar info background | `color.bg.infoSubtle` |
| Assigned cell (edit active) border | `color.border.active` |
| Activity Popup background | `color.bg.overlay` |
| Activity Popup shadow | `color.shadow.md` |
| Danger button (Delete) | `color.bg.danger`, `color.text.onDanger` |
| Month navigator arrow | `color.icon.secondary` |
| Filter tab (active) | `color.text.primary`, `color.border.active` underline |
| Filter tab (inactive) | `color.text.secondary` |

---

## 11. Platform Notes

### Android / iOS
- The Activity Popup renders as a **bottom sheet** (not a popover) on Android and iOS — full width, slides up from the bottom.
- The Detail Panel renders as a **bottom sheet** on Android and iOS when an envelope is selected; tapping outside the sheet dismisses it and clears `selectedEnvelopeId`.
- Month navigation arrows are replaced by a swipe gesture (left/right) on the budget table area.

### Web Desktop
- The Detail Panel is sticky in the right column and does not scroll with the envelope list.
- The Activity Popup is a popover anchored to the Activity cell.

### Web Mobile
- Both the Detail Panel and Activity Popup render inline (stacked below the triggering element) — no bottom sheet.

---

## 12. Open Questions

1. **Underfunded / Overfunded thresholds** — the heuristics in §4 FilterTabs are placeholders. What is the exact business rule for labeling an envelope underfunded or overfunded? → Blocks finalizing the filter tab behavior.
2. **Auto-reassign on Delete** — when the user deletes an envelope with transactions, does the app show a reassignment UI inside the Modal, or does it navigate to a separate reassignment screen? → Affects §4 Modal and §8 interaction flow.
3. **Forward planning** — can users navigate to future months and pre-assign envelopes before income arrives? If so, does the balance validation (AlertBar) apply to future months? → Affects §6 state machine and §8 month navigation.
4. **VCB import auto-categorization** — when importing a VCB statement file, are transactions automatically assigned to envelopes (e.g. by keyword match or AI), or does the user categorize manually? → Affects the Transaction import flow (separate screen spec); referenced here because import is the primary data-entry path.

---

## 13. Test IDs

### 1. Page Header

| Element | `data-testid` | Description |
|---|---|---|
| Month previous button | `month-prev` | `←` arrow |
| Month next button | `month-next` | `→` arrow |
| Month label | `month-label` | `{Month} {YYYY}` |
| Alert bar (danger) | `alert-bar-danger` | Overspent warning |
| Alert bar (info) | `alert-bar-info` | Unassigned money notice |

### 2. Filter & Toolbar

| Element | `data-testid` | Description |
|---|---|---|
| Filter tab container | `filter-tabs` | Row of filter tabs |
| Filter tab (per tab) | `filter-tab-{value}` | e.g. `filter-tab-all`, `filter-tab-overspent` |
| Add group button | `add-group-btn` | `+ Add Category Group` |

### 3. Envelope Table

| Element | `data-testid` | Description |
|---|---|---|
| Table root | `envelope-table` | Root table element |
| Group row | `group-row-{groupId}` | Category group header row |
| Group expand/collapse | `group-toggle-{groupId}` | Chevron button |
| Envelope row | `envelope-row-{envelopeId}` | Per-envelope table row |
| Assigned cell | `assigned-cell-{envelopeId}` | Clickable assigned amount cell |
| Assigned input | `assigned-input-{envelopeId}` | Inline input when cell is in edit mode |
| Activity cell | `activity-cell-{envelopeId}` | Clickable activity amount cell |
| Available cell | `available-cell-{envelopeId}` | Available balance cell |
| Empty state (no envelopes) | `envelope-table-empty` | Shown when no envelopes exist |
| Empty state (filter) | `filter-empty` | Shown when active filter matches no envelopes |

### 4. Activity Popup

| Element | `data-testid` | Description |
|---|---|---|
| Popup container | `activity-popup` | Root popover / bottom sheet |
| Transaction row | `activity-row-{transactionId}` | Per-transaction row |
| Close button | `activity-close` | `Close` button |
| Empty state | `activity-popup-empty` | No transactions message |

### 5. Detail Panel

| Element | `data-testid` | Description |
|---|---|---|
| Panel container | `detail-panel` | Right column panel card |
| Empty state | `detail-panel-empty` | `Select an envelope to see details` hint |
| Envelope name (view) | `envelope-name` | Static name display |
| Edit button | `envelope-edit-btn` | ✏️ icon that triggers edit mode |
| Name input (edit) | `envelope-name-input` | Inline name text field |
| Hide button | `envelope-hide-btn` | **Hide** button |
| Delete button | `envelope-delete-btn` | **Delete** button |
| Cancel edit button | `envelope-cancel-btn` | **Cancel** button |
| Save name button | `envelope-save-btn` | **Save** button |
| Balance breakdown container | `balance-breakdown` | Collapsible section |
| Carried over row | `balance-carried-over` | `Carried Over From Last Month` row |
| Assigned this month row | `balance-assigned` | `Assigned This Month` row (also inline-editable) |
| Cash spending row | `balance-cash-spending` | `Cash Spending` row |
| Credit spending row | `balance-credit-spending` | `Credit Spending` row |
| Notes textarea | `envelope-notes` | Free-text notes field |

### 6. Layout States

| Element | `data-testid` | Description |
|---|---|---|
| Screen root (ready) | `budget-overview` | Root wrapper in the ready state |
| Screen root (loading) | `budget-overview-loading` | Root wrapper during initial `Loading` state |
| Screen root (error) | `budget-overview-error` | Root wrapper when initial data fetch fails |
| Loading skeleton | `skeleton-loader` | Skeleton placeholders during `Loading` state |
| Error state | `error-state` | Full-screen error display in `Error` state |

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | **1.1** | Removed Add Member button (not in scope). Removed Target and Auto-Assign from Detail Panel (not in scope). Added §7.3 Activity Popup display rules and full §4 ActivityPopup component spec (based on image: Account / Date / Payee / Memo / Amount columns). Rewrote Detail Panel edit form (§7.6, §4 InlineEdit, §4 Modal) — rename / hide / delete actions now specified in full per image showing inline edit with Hide / Delete / Cancel / OK buttons. Added envelope multi-month accumulation rules (§5 AC, §8, §7.5). Scoped app to VND only throughout (§1, §4, §9). Closed open questions 10.3 (mobile layout — not in scope) and 10.4 (multi-currency — decided: VND only); renumbered remaining questions. | Finance App Team |
| 2026-06-27 | **1.0** | Initial spec — Budget Overview screen. Sections: Purpose, Dependencies, Component Usage (AlertBar, FilterTabs, EnvelopeTable), Acceptance Criteria, State Machine, Display Rules (layout, header, pool table, detail panel), Interaction Rules, Number Formatting, Design Tokens, Test IDs, Open Questions. Reference: YNAB Budget screen (Jun 2026). | Finance App Team |
