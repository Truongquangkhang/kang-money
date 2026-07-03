# Screen: All Accounts

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-30
**Assigned To:** Finance App Team

> **Split transactions — Not in scope.** A transaction belongs to exactly one category. The ability to split a single transaction across multiple categories is excluded from the current release. The split UI is not rendered on this screen.

> **Reconciliation wizard — Not in scope.** Marking individual transactions as reconciled is supported (the cleared icon cycles through three states: uncleared → cleared → reconciled). However, the full guided reconciliation flow (balance matching, lock-all, reconciliation report) is a separate screen and is excluded from this spec.

> **Scheduled / recurring transactions — Not in scope.** Auto-generated future transactions from recurring rules are excluded from the current release. No "upcoming" or "scheduled" row type is rendered.

> **Multi-currency — Not in scope.** The app supports VND only. All monetary values are stored, computed, and displayed in VND. No currency selector is rendered on this screen.

> This spec is the target behavior contract for the All Accounts screen of the personal expense management app.

---

## 1. Purpose

The All Accounts screen is the unified transaction ledger. It aggregates all transactions from every account the user has into a single scrollable table, ordered by date descending. The screen is the primary place to:

- review recent activity across all accounts,
- add and edit individual transactions,
- clear (mark as confirmed) incoming and outgoing transactions, and
- search and filter the transaction history.

The screen does **not** perform budget allocation — that is owned by the Budget Overview screen. Its relationship to budgeting is one-way: a transaction's category drives the Activity column in the Budget Overview, but the budget itself is not managed here.

---

## 2. Dependencies

| Document | Purpose |
|---|---|
| `docs/Theme.md` | Canonical design tokens — colors, typography, spacing |
| `docs/components/feedback/LoadingState.md` | Skeleton display during initial data load |
| `docs/components/feedback/ErrorState.md` | Content-area error display during `Error` state |
| `docs/components/feedback/EmptyState.md` | No-transactions and no-search-results states |
| `docs/components/data/TransactionTable.md` | The multi-column transaction table with sticky header and virtual scroll |
| `docs/components/input/InlineEdit.md` | Inline editing of individual fields within an open transaction row |
| `docs/components/overlay/Modal.md` | Delete confirmation dialog; File Import dialog |
| `docs/screens/BudgetOverview.md` | Consuming screen — transaction categories drive the Activity column there |

---

## 3. Inputs

N/A — top-level screen that owns its own data fetch; no parent caller injects props.

---

## 4. Component Usage

### BalanceSummaryBar

Displays the three key balance figures derived from all transactions across all accounts. Always visible; cannot be collapsed or dismissed.

| Prop | Value |
|---|---|
| `clearedBalance` | Sum of `amount` for all transactions with `cleared = "cleared"` or `cleared = "reconciled"` across all accounts |
| `unclearedBalance` | Sum of `amount` for all transactions with `cleared = "uncleared"` across all accounts |
| `workingBalance` | `clearedBalance + unclearedBalance` (computed; not persisted) |

The three values are laid out horizontally: `{clearedBalance} + {unclearedBalance} = {workingBalance}`. Each value has a label and icon beneath it (see §7.2).

`data-testid`: `balance-summary-bar`.

### Toolbar

A row of controls between the Balance Summary Bar and the Transaction Table. Two groups separated by a vertical rule.

**Left group:** `+ Add Transaction` · `File Import` | `Undo` · `Redo`

**Right group:** `View ▼` · Search input

Individual elements are specified in the Acceptance Criteria (§5) and Interaction Rules (§8). `data-testid`: `toolbar`.

### TransactionTable

The main data table. Renders a sticky header row and one row per transaction.

| Prop | Value |
|---|---|
| `transactions` | Filtered, sorted transaction array (post-search, post-view-filter) |
| `accounts` | Account name map keyed by `account_id` — used for Account column display |
| `sortColumn` | Active sort column key |
| `sortDirection` | `asc \| desc` |
| `selectedIds` | Set of selected transaction IDs (drives checkbox state) |
| `editingId` | ID of the row currently in inline edit mode, or `null` |
| `isAddingTransaction` | Whether the blank new-transaction row is shown |
| `onClickRow` | Called when a row body is clicked; sets `editingId` |
| `onToggleSelect` | Called when a row's checkbox is toggled |
| `onSelectAll` | Called when the header checkbox is clicked |
| `onToggleCleared` | Called when the cleared status icon is clicked |
| `onSort` | Called with `{ column, direction }` when a sortable header is clicked |

Column structure (full specification in §7.4): Checkbox · Flag · Attachment · Account · Date · Payee · Category · Memo · Outflow · Inflow · Cleared Status.

`data-testid`: `transaction-table`.

### InlineEdit — Transaction field

Used for each editable field when a row is in edit mode.

| Prop | Value |
|---|---|
| `field` | Field key: `date \| account \| payee \| category \| memo \| outflow \| inflow` |
| `value` | Current field value |
| `onCommit` | Persists the changed value |
| `onCancel` | Reverts to the previous value |

Payee and Category fields provide autocomplete from the user's existing payees and category list respectively. Date uses a date picker. Outflow and Inflow accept VND integer input; entering a value in one clears the other.

### Modal — Delete confirmation

| Prop | Value |
|---|---|
| `title` | `Delete {N} transaction{s}?` (singular: `Delete transaction?`) |
| `body` | `This action cannot be undone.` |
| `primaryAction` | `{ label: "Delete", onPress: confirmDelete, variant: "danger" }` |
| `secondaryAction` | `{ label: "Cancel", onPress: closeModal }` |

Opened when: (a) user clicks Delete in the Bulk Action Bar, or (b) user clicks the per-row Delete button inside an open edit row. The "Starting Balance" transaction has no Delete button — the trigger never fires for it.

### Modal — File Import

| Prop | Value |
|---|---|
| `title` | `Import Transactions` |
| `body` | File drop zone + format selector (CSV / OFX / QFX) + column mapping step |
| `primaryAction` | `{ label: "Import", onPress: runImport }` |
| `secondaryAction` | `{ label: "Cancel", onPress: closeModal }` |

Import deduplication: transactions with a matching `import_id` are skipped silently. After import, a toast reports: `{N} transactions imported, {M} duplicates skipped.`

---

## 5. Acceptance Criteria

### Balance Summary Bar
- [ ] Shows Cleared Balance, a `+` operator, Uncleared Balance, a `=` operator, and Working Balance in a single horizontal row
- [ ] Cleared Balance is green when ≥ 0, red when < 0; label reads "Cleared Balance" with a filled circle icon `⊙`
- [ ] Uncleared Balance is muted when = 0, red when < 0; label reads "Uncleared Balance" with an empty circle icon `○`
- [ ] Working Balance is green when ≥ 0, red when < 0; rendered in bold weight; label reads "Working Balance"
- [ ] All three values update immediately after any transaction create, edit, delete, or cleared toggle

### Toolbar
- [ ] `+ Add Transaction` is always enabled
- [ ] `File Import` is always enabled
- [ ] `Undo` is enabled only when the undo stack is non-empty; disabled (not hidden) otherwise
- [ ] `Redo` is enabled only when the redo stack is non-empty; disabled (not hidden) otherwise
- [ ] `Undo` and `Redo` are separated from `+ Add Transaction` and `File Import` by a visible vertical rule

### View Filter
- [ ] `View ▼` dropdown offers four options: All, Uncleared, Cleared, Reconciled
- [ ] Default active view on load is `All`
- [ ] Selecting a view immediately filters the transaction list; no network call is made
- [ ] When a view filter is active (not `All`), the active view label is shown inside the button (e.g. `Uncleared ▼`)

### Search
- [ ] Typing in the Search input filters the transaction list in real time (debounce 200 ms)
- [ ] Search is case-insensitive and matches partial strings
- [ ] Search scope: Payee name, Memo, Category name (full group + envelope label)
- [ ] Search and View filter are additive — both apply simultaneously to the transaction list
- [ ] Clearing the Search input (empty string or clicking `×`) immediately restores the full unfiltered list
- [ ] An `×` clear button appears inside the input whenever `searchQuery` is non-empty

### Transaction Table — Columns
- [ ] Table header is sticky; it remains visible when the list scrolls vertically
- [ ] Date column is sorted descending by default (newest first); a `▼` indicator is visible on the Date header
- [ ] Clicking a sortable column header sorts by that column; clicking the same header again reverses the direction
- [ ] Only one column can be sorted at a time; the sort indicator moves to the clicked column
- [ ] Outflow and Inflow cells are empty (not `0đ`) when the transaction amount is in the other direction
- [ ] Outflow and Inflow values are right-aligned and rendered in tabular-spaced numerics
- [ ] Category cell shows the format `{Group}: {🎯 EnvelopeName}` including the envelope emoji; special value: `Inflow: Ready to Assign`

### Adding a Transaction
- [ ] Clicking `+ Add Transaction` inserts a blank editable row at the top of the table, below the toolbar
- [ ] Focus is placed on the Date field; the value defaults to today's date
- [ ] Tab moves focus in field order: Date → Account → Payee → Category → Memo → Outflow → Inflow → Cleared Status
- [ ] A transaction cannot have both Outflow and Inflow values simultaneously; entering a value in one clears the other
- [ ] Pressing Enter on the last field or clicking a `Save` button commits the row
- [ ] Pressing Escape cancels and removes the blank row with no changes persisted
- [ ] A valid new transaction requires at minimum: a Date and at least one of Outflow or Inflow > 0
- [ ] The committed transaction is pushed to the undo stack and appears in the list at the correct sort position

### Editing a Transaction
- [ ] Clicking anywhere on a transaction row (outside the Checkbox and Cleared Status icon columns) opens that row in inline edit mode
- [ ] Only one row can be in edit mode at a time; clicking a second row commits the current edit first, then opens the new one
- [ ] The editing row expands to show all field inputs in place
- [ ] Pressing Enter commits the edit; pressing Escape cancels and reverts all fields to their previous values
- [ ] Clicking outside the open row (anywhere on the page that is not another transaction row) commits the edit
- [ ] The checkbox is disabled for a row that is currently in edit mode
- [ ] The "Starting Balance" transaction (payee = `Starting Balance`) has no Delete button in its edit row; all other fields remain editable

### Cleared Status Toggle
- [ ] Clicking the Cleared icon on a closed row (not in edit mode) toggles directly: `uncleared → cleared` or `cleared → uncleared`
- [ ] Toggling cleared status updates the Balance Summary Bar (Cleared Balance and Uncleared Balance) immediately
- [ ] Clicking the Cleared icon on a reconciled transaction does nothing; a tooltip reads "Reconciled transactions cannot be modified"
- [ ] The cleared icon is also available inside the open edit row and toggles without requiring a full row save

### Bulk Actions
- [ ] Checking any transaction checkbox enters bulk selection mode and shows the Bulk Action Bar above the table
- [ ] The Bulk Action Bar shows: `{N} transactions selected` · `Mark as Cleared` · `Mark as Uncleared` · `Delete` · `Cancel`
- [ ] The header checkbox selects all currently visible (post-filter) rows; if all visible rows are already selected, clicking it deselects all
- [ ] `Mark as Cleared` sets `cleared = "cleared"` for all selected transactions; all rows are deselected on completion
- [ ] `Mark as Uncleared` sets `cleared = "uncleared"` for all non-reconciled selected transactions; reconciled rows are skipped and a toast reads `{M} reconciled transactions were not modified`; all rows are deselected on completion
- [ ] Bulk `Delete` opens the Delete confirmation modal; on confirmation all selected transactions are deleted and all rows are deselected
- [ ] `Cancel` in the Bulk Action Bar deselects all rows without performing any action

### Undo / Redo
- [ ] Undo reverses the last user action: add, edit, delete, cleared toggle, bulk clear, bulk uncleared, bulk delete
- [ ] Redo re-applies the most recently undone action
- [ ] Committing any new action clears the redo stack
- [ ] `Ctrl/⌘ + Z` fires Undo; `Ctrl/⌘ + Shift + Z` fires Redo
- [ ] File Import is excluded from the undo stack and cannot be undone

### Empty States
- [ ] No transactions at all: shows EmptyState with copy "No transactions yet" and a `+ Add Transaction` call to action
- [ ] Search returns no results: shows "No transactions match your search" with a "Clear search" link below the table header
- [ ] View filter returns no results: shows "No {filterLabel} transactions" with a "Show all" link

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
| `transactions` | `Transaction[]` | `[]` | All transactions, unfiltered and unsorted |
| `accounts` | `Account[]` | `[]` | Account list — used for Account column display names |
| `clearedBalance` | `number` | `0` | Sum of `amount` for cleared and reconciled transactions |
| `unclearedBalance` | `number` | `0` | Sum of `amount` for uncleared transactions |
| `activeView` | `all \| uncleared \| cleared \| reconciled` | `all` | Active view filter |
| `searchQuery` | `string` | `""` | Active search string |
| `sortColumn` | `date \| payee \| category \| account \| outflow \| inflow` | `date` | Column driving the sort |
| `sortDirection` | `asc \| desc` | `desc` | Sort direction for `sortColumn` |
| `selectedTransactionIds` | `string[]` | `[]` | IDs of checked rows (bulk selection) |
| `editingTransactionId` | `string \| null` | `null` | ID of the row currently in inline edit mode |
| `isAddingTransaction` | `boolean` | `false` | Whether the blank new-transaction row is shown |
| `undoStack` | `UndoAction[]` | `[]` | Reversible action history |
| `redoStack` | `UndoAction[]` | `[]` | Re-applicable undone actions |
| `deleteModal` | `{ transactionIds: string[] } \| null` | `null` | Pending delete confirmation; `null` when modal is closed |
| `fileImportModalOpen` | `boolean` | `false` | Whether the File Import dialog is open |

> `workingBalance` is computed at render time as `clearedBalance + unclearedBalance`; it is not stored in the reducer.

> `selectedTransactionIds`, `editingTransactionId`, `isAddingTransaction`, `undoStack`, and `redoStack` are reset when `screenState` transitions from `Ready` back to `Loading` (e.g. after a pull-to-refresh on mobile).

### Events

| Event | Trigger | Kind |
|---|---|---|
| `ScreenOpened` | Platform mounts the screen | system |
| `DataReceived` | Initial transaction and account data arrives | system |
| `ApiErrorReceived` | Initial data fetch fails | system |
| `UserRetried` | User taps "Try again" in the Error state | user |
| `UserChangedView` | User selects an option from `View ▼` | user |
| `UserSearched` | User types in the Search field | user |
| `UserSortedColumn` | User clicks a sortable column header | user |
| `UserStartedAddTransaction` | User clicks `+ Add Transaction` | user |
| `UserCancelledAddTransaction` | User presses Escape on the blank add row | user |
| `UserCommittedNewTransaction` | User saves the new transaction row | user |
| `TransactionCreateSucceeded` | API confirms the new transaction | system |
| `TransactionCreateFailed` | API rejects the create | system |
| `UserClickedTransactionRow` | User clicks a row body (outside Checkbox and Cleared icon) | user |
| `UserCommittedEdit` | User saves an edited transaction | user |
| `UserCancelledEdit` | User presses Escape on an open edit row | user |
| `TransactionUpdateSucceeded` | API confirms the edit | system |
| `TransactionUpdateFailed` | API rejects the edit | system |
| `UserToggledCleared` | User clicks the cleared status icon on any row | user |
| `UserCheckedTransaction` | User checks a row checkbox | user |
| `UserUncheckedTransaction` | User unchecks a row checkbox | user |
| `UserCheckedAll` | User clicks the header checkbox (deselected → all selected) | user |
| `UserUncheckedAll` | User clicks the header checkbox (all selected → all deselected) | user |
| `UserOpenedDeleteModal` | User clicks Delete (single row or bulk) | user |
| `UserConfirmedDelete` | User clicks Delete in the confirmation modal | user |
| `UserCancelledDelete` | User clicks Cancel in the confirmation modal | user |
| `TransactionDeleteSucceeded` | API confirms the deletion | system |
| `UserBulkClearedTransactions` | User clicks "Mark as Cleared" in the Bulk Action Bar | user |
| `UserBulkUnclearedTransactions` | User clicks "Mark as Uncleared" in the Bulk Action Bar | user |
| `UserUndone` | User clicks Undo or presses `Ctrl/⌘ + Z` | user |
| `UserRedone` | User clicks Redo or presses `Ctrl/⌘ + Shift + Z` | user |
| `UserOpenedFileImport` | User clicks `File Import` | user |
| `FileImportCompleted` | Import finishes; new transactions are created | system |

### Transitions

| Current state | Event | Next state | Notes |
|---|---|---|---|
| `Loading` | `DataReceived` | `Ready` | Populate `transactions`, `accounts`, `clearedBalance`, `unclearedBalance` |
| `Loading` | `ApiErrorReceived` | `Error` | |
| `Error` | `UserRetried` | `Loading` | Re-dispatch data fetch |
| `Ready` | `UserChangedView` | `Ready` | Update `activeView`; re-filter displayed rows (no network call) |
| `Ready` | `UserSearched` | `Ready` | Update `searchQuery`; re-filter displayed rows (no network call) |
| `Ready` | `UserSortedColumn` | `Ready` | Update `sortColumn` and `sortDirection`; re-sort displayed rows |
| `Ready` | `UserStartedAddTransaction` | `Ready` | Set `isAddingTransaction = true`; set `editingTransactionId = null`; clear `selectedTransactionIds` |
| `Ready` | `UserCancelledAddTransaction` | `Ready` | Set `isAddingTransaction = false` |
| `Ready` | `UserCommittedNewTransaction` | `Ready` | Set `isAddingTransaction = false`; optimistically insert transaction; push to `undoStack`; clear `redoStack` |
| `Ready` | `TransactionCreateSucceeded` | `Ready` | Replace optimistic transaction with server-confirmed data; recompute balances |
| `Ready` | `TransactionCreateFailed` | `Ready` | Remove optimistic transaction; show toast error |
| `Ready` | `UserClickedTransactionRow` | `Ready` | Commit any open edit first; set `editingTransactionId` to the clicked row ID |
| `Ready` | `UserCancelledEdit` | `Ready` | Set `editingTransactionId = null`; revert changes |
| `Ready` | `UserCommittedEdit` | `Ready` | Set `editingTransactionId = null`; optimistically update transaction; push to `undoStack`; clear `redoStack` |
| `Ready` | `TransactionUpdateFailed` | `Ready` | Revert optimistic update; show toast error |
| `Ready` | `UserToggledCleared` | `Ready` | Toggle `cleared` on the transaction; recompute `clearedBalance` and `unclearedBalance`; push to `undoStack`; clear `redoStack` |
| `Ready` | `UserCheckedTransaction` | `Ready` | Add ID to `selectedTransactionIds` |
| `Ready` | `UserUncheckedTransaction` | `Ready` | Remove ID from `selectedTransactionIds` |
| `Ready` | `UserCheckedAll` | `Ready` | Set `selectedTransactionIds` to all visible row IDs |
| `Ready` | `UserUncheckedAll` | `Ready` | Clear `selectedTransactionIds` |
| `Ready` | `UserOpenedDeleteModal` | `Ready` | Set `deleteModal = { transactionIds }` |
| `Ready` | `UserCancelledDelete` | `Ready` | Set `deleteModal = null` |
| `Ready` | `UserConfirmedDelete` | `Ready` | Set `deleteModal = null`; optimistically remove transactions; push to `undoStack`; clear `redoStack`; clear `selectedTransactionIds` |
| `Ready` | `UserBulkClearedTransactions` | `Ready` | Set `cleared = "cleared"` on all selected; recompute balances; push to `undoStack`; clear `redoStack`; clear `selectedTransactionIds` |
| `Ready` | `UserBulkUnclearedTransactions` | `Ready` | Set `cleared = "uncleared"` on non-reconciled selected; recompute balances; push to `undoStack`; clear `redoStack`; clear `selectedTransactionIds` |
| `Ready` | `UserUndone` | `Ready` | Pop from `undoStack`; push to `redoStack`; reverse the action; recompute balances |
| `Ready` | `UserRedone` | `Ready` | Pop from `redoStack`; push to `undoStack`; re-apply the action; recompute balances |
| `Ready` | `UserOpenedFileImport` | `Ready` | Set `fileImportModalOpen = true` |
| `Ready` | `FileImportCompleted` | `Ready` | Set `fileImportModalOpen = false`; append new transactions; recompute balances |

> View filter, search, and sort changes are local derived state with no network call. The `transactions` array is always the full unfiltered server data; filtering and sorting occur at render time.

---

## 7. Display Rules

### 7.1 Layout

Full-width content area. No secondary panel. Single-column layout; vertical scroll.

#### Web Desktop

```
Content area (full width, side padding)
+-- Page title — "All Accounts" (H1)
|
+-- Balance Summary Bar (full width)
|   [⊙ 1.500.000đ]  [+]  [○ -160.000đ]  [=]  [1.340.000đ]
|    Cleared Balance       Uncleared Balance    Working Balance
|
+-- Toolbar (full width)
|   Left:  [+ Add Transaction]  [⬆ File Import]  |  [↩ Undo]  [↪ Redo]
|   Right: [View ▼]  [🔍 Search All Accounts________________]
|
+-- Bulk Action Bar (full width; only shown when selectedTransactionIds.length > 0)
|   3 transactions selected  [Mark as Cleared]  [Mark as Uncleared]  [Delete]  [Cancel]
|
+-- Transaction Table (full width, vertical scroll)
    +-- Header row (sticky)
    |   ☐  🚩  📎  ACCOUNT  DATE ▼  PAYEE  CATEGORY  MEMO  OUTFLOW  INFLOW  ⊙
    +-- Transaction row × N  (virtual scroll — render only rows in viewport)
    |   ☐  🚩  📎  VCB  27/06  KFC  Wants: 🍽 Dining out  kfc  50.000đ        ⊙
    +-- Empty state (replaces rows when list is empty after filtering)
```

#### Web Mobile / Narrow

Single column. Table scrolls horizontally inside `overflow-x: auto`. Flag and Attachment columns are hidden. Toolbar wraps to two rows.

```
Content area (full width)
+-- Page title
+-- Balance Summary Bar (values stacked vertically if viewport < 480px)
+-- Toolbar row 1: [+ Add Transaction]  [File Import]
+-- Toolbar row 2: [Undo]  [Redo]  [View ▼]  [Search_____]
+-- Bulk Action Bar (if active, below toolbar)
+-- Transaction Table (horizontal scroll; Checkbox + Account columns sticky left)
```

### 7.2 Balance Summary Bar

| Element | Content | Color condition |
|---|---|---|
| Cleared Balance icon | `⊙` (filled circle) | `color.icon.positive` |
| Cleared Balance value | Formatted VND amount | `color.text.positive` if ≥ 0; `color.text.negative` if < 0 |
| Cleared Balance label | "Cleared Balance" | `color.text.secondary` |
| `+` operator | Static `+` | `color.text.tertiary` |
| Uncleared Balance icon | `○` (empty circle) | `color.icon.tertiary` |
| Uncleared Balance value | Formatted VND amount | `color.text.secondary` if = 0; `color.text.negative` if < 0 |
| Uncleared Balance label | "Uncleared Balance" | `color.text.secondary` |
| `=` operator | Static `=` | `color.text.tertiary` |
| Working Balance value | Formatted VND amount | `color.text.positive` if ≥ 0; `color.text.negative` if < 0; **bold weight** |
| Working Balance label | "Working Balance" | `color.text.secondary` |

### 7.3 Toolbar

Left group and right group are placed at opposite ends of the toolbar row (`justify-content: space-between`). A vertical rule `|` separates `File Import` from `Undo` within the left group.

| Element | Appearance | Enabled condition |
|---|---|---|
| `+ Add Transaction` | Primary button, `+` icon | Always |
| `File Import` | Secondary button, file upload icon | Always |
| `Undo` | Ghost button, `↩` icon, label "Undo" | `undoStack.length > 0` |
| `Redo` | Ghost button, `↪` icon, label "Redo" | `redoStack.length > 0` |
| `View ▼` | Ghost button; active view label rendered inside | Always |
| Search input | Text input, `🔍` leading icon, `×` trailing clear button | Always |

### 7.4 Transaction Table — Columns

| # | Column | Header | Min width | Align | Sortable | Notes |
|---|---|---|---|---|---|---|
| C0 | Checkbox | — | 36px | Center | No | Header checkbox: select all / deselect all visible rows |
| C1 | Flag | — | 32px | Center | No | Colored flag; click to open color picker. Hidden on mobile. |
| C2 | Attachment | — | 32px | Center | No | Muted when empty; click to view or upload. Hidden on mobile. |
| C3 | Account | ACCOUNT | 120px | Left | Yes | Account name from the `accounts` map (e.g. `VCB`, `MoMo`) |
| C4 | Date | DATE | 110px | Left | Yes | `DD/MM/YYYY`; default sort column, descending |
| C5 | Payee | PAYEE | flex | Left | Yes | Payee name; empty cell when not set |
| C6 | Category | CATEGORY | flex | Left | Yes | `{Group}: {🎯 Envelope}`; special value: `Inflow: Ready to Assign` |
| C7 | Memo | MEMO | flex | Left | No | User note; truncated at ~40 chars with ellipsis |
| C8 | Outflow | OUTFLOW | 110px | Right | Yes | VND amount when `amount < 0`; empty otherwise |
| C9 | Inflow | INFLOW | 110px | Right | Yes | VND amount when `amount > 0`; empty otherwise |
| C10 | Cleared Status | `⊙` | 36px | Center | No | Click to toggle; see §7.6 for icon states |

### 7.5 Transaction Row — Types and States

| Type | Condition | Visual treatment |
|---|---|---|
| Normal — uncleared | `cleared = "uncleared"` | Default row background; Cleared icon `○` in `color.icon.tertiary` |
| Normal — cleared | `cleared = "cleared"` | Default row background; Cleared icon `⊙` in `color.icon.positive` |
| Normal — reconciled | `cleared = "reconciled"` | Default row background; Cleared icon `🔒`; all fields read-only except Memo |
| Starting Balance | `payee_name = "Starting Balance"` | Default row; no Delete button in edit mode; category always `Inflow: Ready to Assign` |
| Selected | `id ∈ selectedTransactionIds` | Row background: `color.bg.selectedRow`; checkbox checked |
| Editing | `id = editingTransactionId` | Row expands; all eligible fields become inputs; checkbox is disabled |
| Hover | Cursor over a non-editing, non-selected row | Subtle `color.bg.rowHover` background tint |

### 7.6 Cleared Status Icon States

| State | Icon | Color | Click behavior |
|---|---|---|---|
| Uncleared | `○` | `color.icon.tertiary` | Toggle to `cleared` |
| Cleared | `⊙` | `color.icon.positive` | Toggle to `uncleared` |
| Reconciled | `🔒` | `color.icon.secondary` | No-op; tooltip: "Reconciled transactions cannot be modified" |

### 7.7 Bulk Action Bar

A full-width strip rendered between the Toolbar and the Transaction Table. Visible only when `selectedTransactionIds.length > 0`. On mobile it is a fixed bar at the bottom of the viewport.

| Element | Content |
|---|---|
| Selection count | `{N} transaction{s} selected` |
| Mark as Cleared | Secondary button |
| Mark as Uncleared | Secondary button |
| Delete | Danger button; fires `UserOpenedDeleteModal` |
| Cancel | Ghost button; fires `UserUncheckedAll` |

---

## 8. Interaction Rules

### Add Transaction
- Clicking `+ Add Transaction` closes any open edit row (committing it if it has changes) and sets `isAddingTransaction = true`. The blank row appears at the top of the list, above all existing transactions.
- The Account field defaults to the account most recently used by the user. If no history exists, it defaults to the first account alphabetically.
- Entering a value in Outflow clears Inflow, and vice versa.
- The row is saved optimistically on commit — it appears in the list immediately and the Balance Summary Bar updates. The API call runs in the background. On failure, the transaction is removed and a toast shows: "Failed to save. Try again."

### Edit Transaction
- Clicking outside an open edit row (anywhere on the page that is not another transaction row) commits the edit. This is the primary save path.
- When clicking a second transaction row while one is already open: if the open row has unsaved changes, it is committed first; if it is unchanged, it is closed without an API call, then the new row opens.
- The Memo field does not submit on Enter (it allows natural linebreaks). All other fields commit on Enter.
- Reconciled transactions: all fields are read-only except Memo. The row can still be opened in edit mode, but only the Memo input is active.

### Flag
- Clicking the Flag column cell opens an inline color picker: no flag · red · orange · yellow · green · blue · purple. Selecting a color saves immediately; the row does not need to be in edit mode.

### Attachment
- Clicking the Attachment cell opens a file picker if no attachment exists. If an attachment exists, it opens a preview overlay with a delete option.
- Supported formats: JPEG, PNG, PDF. Max 10 MB per attachment. One attachment per transaction.

### Toggle Cleared Status
- Clicking the Cleared icon outside of edit mode (on a closed row) saves the toggle directly to the API. No row expansion occurs.
- The optimistic update is immediate. If the API call fails, the change is reverted and a toast is shown.

### Search
- The search filter debounces at 200 ms after the last keystroke.
- Search does not reset the sort order, active view, or selected rows. Selected rows that no longer match the search remain in `selectedTransactionIds` and reappear when the search is cleared.

### Sort
- Clicking a column header that is already the sort column reverses the direction (`asc ↔ desc`). Clicking a new column sets it as the sort column with `asc` as the initial direction, except Date which defaults to `desc` (see §12 Open Questions).
- Sort is applied after search and view filters.

### Bulk Selection
- The header checkbox selects all *visible* rows (post-filter). Rows outside the current filter are not added to `selectedTransactionIds`.
- After any bulk action completes, `selectedTransactionIds` is cleared and the Bulk Action Bar is dismissed.

### Delete
- The delete confirmation modal is non-destructive by default: closing the modal or pressing Escape cancels the delete with no changes.

### Undo / Redo
- Undo and Redo operate on a per-session, in-memory stack. The stacks are cleared on navigation away from the screen or app reload.
- File Import is not undoable and is not pushed to the undo stack.

### File Import
- The File Import dialog does not close on outside-click — the user must explicitly cancel or complete the import.
- After a successful import, the transaction list and Balance Summary Bar update to reflect all newly imported transactions. A toast confirms the count.

---

## 9. Number Formatting

All monetary values are VND integers (no decimals). Format follows the Vietnamese locale used throughout the app — see `docs/screens/BudgetOverview.md §9` for the canonical format table. Summary:

| Value type | Format | Example |
|---|---|---|
| VND amounts (positive) | Dot-separated thousands, `đ` suffix | `1.500.000đ` |
| VND amounts (negative) | Leading `-`, dot-separated thousands, `đ` suffix | `-50.000đ` |
| VND zero | `0đ` | `0đ` |
| Dates in table rows | `DD/MM/YYYY` | `27/06/2026` |
| Inline edit input | Dot-separated during typing (auto-formatted); stored as integer | `1.500.000` → stored as `1500000` |

---

## 10. Design Tokens

| Element | Token |
|---|---|
| Page background | `color.bg.page` |
| Table row — default | `color.bg.surface` |
| Table row — hover | `color.bg.rowHover` |
| Table row — selected | `color.bg.selectedRow` |
| Table row — editing | `color.bg.editingRow` |
| Table header background | `color.bg.subtleTint` |
| Table header border (bottom) | `color.border.strong` |
| Row border (between rows) | `color.border.subtle` |
| Cleared Balance value (positive) | `color.text.positive` |
| Cleared Balance value (negative) | `color.text.negative` |
| Uncleared Balance value (zero) | `color.text.secondary` |
| Uncleared Balance value (negative) | `color.text.negative` |
| Working Balance value | `color.text.positive` (≥ 0) / `color.text.negative` (< 0) |
| Cleared icon — uncleared | `color.icon.tertiary` |
| Cleared icon — cleared | `color.icon.positive` |
| Cleared icon — reconciled | `color.icon.secondary` |
| Outflow amount | `color.text.primary` |
| Inflow amount | `color.text.positive` |
| Primary button (Add Transaction) | `color.bg.primary` / `color.text.onPrimary` |
| Danger button (Delete) | `color.bg.danger` / `color.text.onDanger` |
| Bulk Action Bar background | `color.bg.subtleTint` |
| Bulk Action Bar border | `color.border.subtle` |
| Search input border (focus) | `color.border.active` |
| Disabled button | `color.text.disabled` / `color.bg.disabled` |

---

## 11. Platform Notes

### Android / iOS
- Tapping a transaction row does not open inline edit — it navigates to a full-screen `TransactionForm` screen (separate spec).
- Cleared Status toggle is accessible via a long-press context menu on each row.
- Bulk selection is entered via long-press on a row; subsequent taps toggle selection.
- File Import is accessible via the device native share sheet (e.g. share a CSV from the Files app into kang-money).
- The Bulk Action Bar renders as a fixed bottom bar above the system navigation bar.

### Web Desktop
- The Transaction Table header is sticky at the top of the content area.
- The Bulk Action Bar slides in between the Toolbar and the table (does not overlap content).
- Keyboard shortcuts are active: `Ctrl/⌘ + Z` (Undo), `Ctrl/⌘ + Shift + Z` (Redo), `Ctrl/⌘ + F` (focus Search), `Escape` (cancel edit / close modal).

### Web Mobile
- Flag and Attachment columns are hidden.
- The Transaction Table scrolls horizontally within its own `overflow-x: auto` container. The Checkbox and Account columns are sticky to the left edge during horizontal scroll.
- Toolbar wraps to two rows to prevent overflow.

---

## 12. Open Questions

1. **Initial sort direction for non-Date columns** — when the user clicks a column header other than Date for the first time, should the initial direction be ascending or descending? The current spec says ascending, but descending may be more intuitive for Outflow and Inflow (largest first). → Blocks finalizing §8 Sort rule.
2. **Transfer transactions** — a transfer between two internal accounts (e.g. MoMo → VCB) should appear as two linked rows. How should linked transfer rows be visually distinguished in the All Accounts view, and can one leg be edited independently? → Affects row type spec in §7.5 and the edit row form.
3. **Virtual scroll threshold** — at what row count should the table switch from a simple render-all to a virtual scroll? → Affects the `TransactionTable` component spec; performance-critical for accounts with multi-year transaction history.
4. **File import formats** — the Modal spec (§4) lists CSV, OFX, and QFX. Are there VN-specific bank formats (e.g. VCB Digibank Excel export, MoMo statement) that require a custom parser? → Affects the File Import modal's format selector and the import pipeline.
5. **Search by amount** — should the search scope include the transaction amount (e.g. searching `50000` returns rows with that value)? Exact match only, or range? → Affects §5 Search AC and §8 Search interaction rule.

---

## 13. Test IDs

### 1. Balance Summary Bar

| Element | `data-testid` | Description |
|---|---|---|
| Bar container | `balance-summary-bar` | Root container |
| Cleared Balance value | `cleared-balance` | Formatted VND amount |
| Uncleared Balance value | `uncleared-balance` | Formatted VND amount |
| Working Balance value | `working-balance` | Formatted VND amount |

### 2. Toolbar

| Element | `data-testid` | Description |
|---|---|---|
| Toolbar container | `toolbar` | Root toolbar row |
| Add Transaction button | `add-transaction-btn` | `+ Add Transaction` |
| File Import button | `file-import-btn` | `File Import` |
| Undo button | `undo-btn` | `↩ Undo` |
| Redo button | `redo-btn` | `↪ Redo` |
| View dropdown trigger | `view-dropdown` | `View ▼` button |
| View option (per option) | `view-option-{value}` | e.g. `view-option-all`, `view-option-uncleared` |
| Search input | `search-input` | Text input |
| Search clear button | `search-clear-btn` | `×` button inside search |

### 3. Bulk Action Bar

| Element | `data-testid` | Description |
|---|---|---|
| Bar container | `bulk-action-bar` | Root strip; not in DOM when no rows are selected |
| Selection count label | `bulk-count-label` | `{N} transactions selected` |
| Mark as Cleared button | `bulk-clear-btn` | |
| Mark as Uncleared button | `bulk-unclear-btn` | |
| Bulk Delete button | `bulk-delete-btn` | |
| Cancel bulk selection | `bulk-cancel-btn` | |

### 4. Transaction Table

| Element | `data-testid` | Description |
|---|---|---|
| Table root | `transaction-table` | Root table element |
| Table header | `transaction-table-header` | Sticky header row |
| Header checkbox | `select-all-checkbox` | Select / deselect all visible rows |
| Sort indicator (per column) | `sort-{column}` | e.g. `sort-date`, `sort-payee`, `sort-outflow` |
| New transaction row | `new-transaction-row` | Blank add row; in DOM only when `isAddingTransaction = true` |
| Transaction row | `transaction-row-{id}` | Per-transaction row |
| Row checkbox | `row-checkbox-{id}` | |
| Row flag cell | `row-flag-{id}` | |
| Row attachment cell | `row-attachment-{id}` | |
| Row cleared icon | `row-cleared-{id}` | Toggleable cleared status icon |
| Empty state (no transactions) | `transaction-table-empty` | Shown when account has no transactions |
| Empty state (no search results) | `search-empty` | Shown when search returns 0 results |
| Empty state (no view results) | `view-empty` | Shown when view filter returns 0 results |

### 5. Inline Edit Row

| Element | `data-testid` | Description |
|---|---|---|
| Date input | `edit-date-{id}` | |
| Account selector | `edit-account-{id}` | |
| Payee input | `edit-payee-{id}` | With autocomplete |
| Category selector | `edit-category-{id}` | With grouped dropdown |
| Memo input | `edit-memo-{id}` | |
| Outflow input | `edit-outflow-{id}` | |
| Inflow input | `edit-inflow-{id}` | |
| Save button | `edit-save-{id}` | |
| Delete button | `edit-delete-{id}` | Not rendered for Starting Balance row |
| Cancel button | `edit-cancel-{id}` | |

### 6. Modals

| Element | `data-testid` | Description |
|---|---|---|
| Delete modal | `delete-modal` | Delete confirmation dialog |
| Delete confirm button | `delete-confirm-btn` | |
| Delete cancel button | `delete-cancel-btn` | |
| File Import modal | `file-import-modal` | |
| File drop zone | `file-drop-zone` | |
| Import confirm button | `import-confirm-btn` | |
| Import cancel button | `import-cancel-btn` | |

### 7. Screen States

| Element | `data-testid` | Description |
|---|---|---|
| Screen root (ready) | `all-accounts-screen` | Root wrapper in Ready state |
| Screen root (loading) | `all-accounts-loading` | Root wrapper during Loading state |
| Screen root (error) | `all-accounts-error` | Root wrapper in Error state |
| Loading skeleton | `skeleton-loader` | Skeleton rows during Loading |
| Error state | `error-state` | Full-area error display |
| Retry button | `retry-btn` | "Try again" in Error state |

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-30 | **1.0** | Initial spec — All Accounts screen. Sections: Purpose, Dependencies, Component Usage (BalanceSummaryBar, Toolbar, TransactionTable, InlineEdit, Modal ×2), Acceptance Criteria, State Machine (reducer fields, events, transitions), Display Rules (layout ASCII diagrams for desktop and mobile, balance bar, toolbar, 11-column spec, row types, cleared icon states, bulk bar), Interaction Rules, Number Formatting (references BudgetOverview §9), Design Tokens, Platform Notes, Open Questions, Test IDs. | Finance App Team |
