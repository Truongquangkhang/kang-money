# Component: EnvelopeTable

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-27
**Category:** data

> The main budget table for the Budget Overview screen — renders grouped envelope (hũ) rows across four columns (Name / Assigned / Activity / Available), with inline-editable assignment, click-through activity, and selection-driven row highlight.

---

## 1. Purpose

`EnvelopeTable` is the central data display of the Budget Overview screen (`docs/screens/BudgetOverview.md`). It presents the user's envelopes (hũ) for a single budget month, organized into collapsible category groups, and exposes the three money columns the user works with: how much was **Assigned** this month, the **Activity** (spending) recorded against the envelope, and the running **Available** balance.

The table is a controlled, presentational component: the parent screen owns all data and selection state and passes it in via props. The table renders `available` exactly as received from the API — it does **not** compute multi-month carry-over locally (see §6). Its only mutating affordance is the inline Assigned edit, which it delegates to the `InlineEdit` component (`docs/components/input/InlineEdit.md`) and surfaces back to the parent through `onEditAssigned`.

All monetary values are VND integers (no decimals); the app is VND-only.

---

## 2. Anatomy

```
┌──────────────────────────────────────────────────────────────────────────┐
│ ▾  Bills                              1.500.000đ    -800.000đ     700.000đ │  ← group header row
├──────────────────────────────────────────────────────────────────────────┤
│      Rent                               800.000đ    -800.000đ          0đ  │  ← envelope row (Fully Spent)
│      Electricity                        300.000đ          0đ      300.000đ │  ← envelope row
│      Internet  [Overspent 100.000đ]     400.000đ    -500.000đ    -100.000đ │  ← envelope row (Overspent)
├──────────────────────────────────────────────────────────────────────────┤
│ ▸  Fun Money                            500.000đ          0đ      500.000đ │  ← group header row (collapsed)
└──────────────────────────────────────────────────────────────────────────┘
   Col 1: Name        Col 2: Assigned     Col 3: Activity    Col 4: Available
   (+ group label /                       (red when           (colored by
    status badge)                          negative)           threshold)
```

Columns, left to right:

- **Col 1 — Name:** group name (bold) on header rows; envelope name (indented) on envelope rows, optionally followed by a status badge.
- **Col 2 — Assigned:** `assignedThisMonth` (VND); inline click-to-edit on envelope rows; summed total on group header rows.
- **Col 3 — Activity:** `activityThisMonth` (VND, negative = spending); click opens the Activity Popup; summed total on group header rows.
- **Col 4 — Available:** `available` (VND), colored by threshold; summed total on group header rows.

A collapsed group (`▸`) hides all of its child envelope rows but keeps the aggregated totals visible in its header row.

---

## 3. Props

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| `envelopes` | `EnvelopeGroup[]` | Yes | — | Filtered, grouped envelope data for the current month. Each group carries its `CategoryGroup` metadata and its child envelopes; already filtered by the active filter tab upstream. |
| `selectedEnvelopeId` | `string \| null` | Yes | `null` | The currently selected envelope ID. Drives the selected-row highlight and the screen's Detail Panel. `null` means nothing is selected. |
| `onSelectEnvelope` | `(envelopeId: string) => void` | Yes | — | Called when the user clicks an envelope row (outside the Assigned and Activity cells). Sets `selectedEnvelopeId` and opens the Detail Panel. |
| `onSelectActivity` | `(envelopeId: string) => void` | Yes | — | Called when the user clicks an Activity cell whose value is non-zero. Opens the Activity Popup for that envelope-month. Not called when the Activity value is `0`. |
| `onEditAssigned` | `(envelopeId: string, newValue: number) => void` | Yes | — | Called when the user commits an inline edit of an Assigned cell. Receives the envelope ID and the new VND integer value. |
| `month` | `string` (`YYYY-MM`) | Yes | — | The current budget month. Used for context (e.g. passed through to the Activity Popup) and to scope the rendered values. |

> **Empty data is the parent's responsibility to render.** The table itself renders only the empty-state hooks described in §5 (`envelope-table-empty`, `filter-empty`); the screen draws the actual `EmptyState` component (`docs/components/feedback/EmptyState.md`) into those slots.

---

## 4. Column Structure

| Column | Header | Content | Editable |
|---|---|---|---|
| 1 | (none) | Envelope name + group label | No (edit via Detail Panel) |
| 2 | Assigned | `assignedThisMonth` formatted as VND | Yes — inline click-to-edit |
| 3 | Activity | `activityThisMonth` formatted as VND (negative = spending) | No — click opens Activity Popup |
| 4 | Available | `available` formatted as VND, colored by state | No |

---

## 5. Row States

### Group header row

One group header row per `CategoryGroup`. Collapsible.

| Element | Content |
|---|---|
| Expand/collapse chevron | `▾` when expanded, `▸` when collapsed |
| Group name | Group name, bold, primary text |
| Assigned total | Sum of `assignedThisMonth` for all envelopes in the group |
| Activity total | Sum of `activityThisMonth` for all envelopes in the group |
| Available total | Sum of `available` for all envelopes in the group |

Clicking the row toggles its expanded state (see §6). When collapsed, all child envelope rows are hidden but the aggregated totals stay visible in the header. Collapse state is local UI state that persists **per session** — it is not persisted to the server.

### Envelope row

Rendered indented beneath its group header. Hidden when its group is collapsed.

| Element | Content |
|---|---|
| Name | Envelope name, indented beneath the group header |
| Status badge | See "Status badges" below; absent when neither condition applies |
| Assigned cell | `assignedThisMonth` in VND; click to inline-edit |
| Activity cell | `activityThisMonth` in VND; shown in `color.text.negative` when negative; click opens the Activity Popup if value ≠ 0 |
| Available cell | `available` in VND; colored by threshold (see below) |

### Status badges

| Badge | Condition | Style |
|---|---|---|
| `Fully Spent` | `activityThisMonth ≠ 0 && available = 0` | Muted pill |
| `Overspent {amount}` | `available < 0` | Red pill; `{amount}` is the absolute value of `available`, VND-formatted (e.g. `Overspent 100.000đ`) |

> If `available < 0` the `Overspent` badge applies (the `Fully Spent` condition requires `available = 0`, so the two are mutually exclusive). At most one badge is shown per row.

### Available-cell color thresholds

| Condition | Color token |
|---|---|
| `available > 0` | `color.text.positive` (green) |
| `available = 0` | `color.text.secondary` (muted) |
| `available < 0` | `color.text.negative` (red) |

---

## 6. Interaction Rules

### Assigned cell — inline edit
- Clicking an Assigned cell activates an inline numeric edit for **that cell only**, via the `InlineEdit` component (`docs/components/input/InlineEdit.md`, format `currency-vnd`, `min` `0`).
- Only **one** cell can be in edit mode at a time. Clicking another Assigned cell commits the current edit first, then opens the new one.
- `Enter` or blur (clicking outside the cell) **commits** the value and fires `onEditAssigned(envelopeId, newValue)`.
- `Escape` **cancels** the edit and reverts the cell to its previous value.
- The input accepts VND integers only (no decimals, no negatives). `0` is a valid value.

### Activity cell — click
- Clicking an Activity cell whose `activityThisMonth = 0` does **nothing** (no callback, no state change).
- Clicking an Activity cell whose `activityThisMonth ≠ 0` fires `onSelectActivity(envelopeId)`, opening the Activity Popup for the envelope-month.

### Envelope row — selection
- Clicking an envelope name, or **any part of the row outside the Assigned and Activity cells**, fires `onSelectEnvelope(envelopeId)` and opens the Detail Panel.
- The selected row is highlighted with `color.bg.selectedRow`.
- Clicking the **same** selected row again does **nothing** — there is no deselect from within the table.

### Group header — expand / collapse
- Clicking a group header row toggles its expanded/collapsed state (local, per-session — see §5).
- Collapsing a group hides its child envelope rows; the group's aggregated totals remain in the header.

### Available — carry-over is server-side
- Each envelope's `available` is a running **multi-month carry-over balance** computed **server-side** from the envelope's history through the current month.
- The table renders `available` exactly as received. It does **not** compute carry-over locally and does not recompute `available` after an inline Assigned edit — that recomputation is the parent screen's responsibility (it re-fetches or recalculates and passes fresh `envelopes` down).

---

## 7. Accessibility

- The table uses native table semantics: a `rowgroup` per category group, `row` per group header and per envelope, and `cell` per column. Header rows expose the column labels (Assigned / Activity / Available) to assistive technology.
- **Keyboard navigation:** all rows and editable cells are reachable in DOM order via `Tab` / arrow keys; focus order follows visual order (group header, then its envelope rows).
- **Editable cells:** Assigned cells are focusable and operable from the keyboard — `Enter` (or `Space`) activates inline edit, `Enter`/blur commits, `Escape` cancels. The active input receives focus on open and returns focus to the cell on commit/cancel.
- **Expand / collapse:** the group chevron is a real button, toggleable via `Enter`/`Space`, with `aria-expanded` reflecting the group's state. Collapsing/expanding is announced.
- **Selected row:** the selected envelope row exposes `aria-selected="true"` and the selection change is announced (e.g. via a polite live region), so the linked Detail Panel context is conveyed to screen-reader users.
- Activity cells that are non-zero are operable buttons (announced as opening the Activity Popup); zero Activity cells are inert and not focusable as actions.
- Color is never the sole signal: negative Activity and the Available thresholds are accompanied by the leading `-` sign and, for overspent/fully-spent envelopes, the text status badge.

---

## 8. Design Tokens

Reference: `docs/Theme.md`.

| Element | Token |
|---|---|
| Envelope row — default background | `color.bg.surface` |
| Envelope row — selected background | `color.bg.selectedRow` |
| Group header row background | `color.bg.subtleTint` |
| Available cell — positive | `color.text.positive` |
| Available cell — zero | `color.text.secondary` |
| Available cell — negative | `color.text.negative` |
| Activity cell — negative value | `color.text.negative` |
| Assigned cell — edit-active border | `color.border.active` |
| `Fully Spent` badge | `color.text.secondary` (muted pill) |
| `Overspent {amount}` badge | `color.text.negative` (red pill) |

---

## 9. Content / Formatting

All monetary values are VND integers (no decimals).

| Value type | Format | Example |
|---|---|---|
| VND amount (positive) | Dot-separated thousands, `đ` suffix | `1.500.000đ` |
| VND amount (negative) | Leading `-`, dot-separated thousands, `đ` suffix | `-50.000đ` |
| VND amount (zero) | `0đ` | `0đ` |
| Large amount (≥ 1 billion) | `{N}B` for exact billions, else full format | `2B` / `1.200.000.000đ` |
| Overspent badge amount | Absolute value of `available`, VND-formatted | `Overspent 100.000đ` |

- Negative **Activity** values are both signed (`-`) and rendered in `color.text.negative`.
- The **Available** column applies the §5 color thresholds; the value itself always uses the standard VND format above.
- Inline Assigned edit input formatting (comma separators during typing, stored as integer) is owned by `InlineEdit` (`docs/components/input/InlineEdit.md`).

---

## 10. Platform Notes

### Android / iOS
- Month navigation is performed via a **swipe gesture** (left / right) on the table area, replacing the desktop month-navigator arrows. The swipe is handled at the screen level; the table area is the gesture surface.
- Inline Assigned edit invokes the numeric keyboard; the active cell scrolls into view above the keyboard.

### Web Desktop / Web Mobile
- Month navigation is via the header arrows / month picker (owned by the screen); the table area is not a gesture surface.
- The table scrolls vertically within the left column; the Detail Panel selection state is driven entirely through `selectedEnvelopeId` / `onSelectEnvelope`.

---

## 11. Test IDs

| Element | `data-testid` | Description |
|---|---|---|
| Table root | `envelope-table` | Root table element |
| Group header row | `group-row-{groupId}` | Category group header row |
| Group expand/collapse | `group-toggle-{groupId}` | Chevron button |
| Envelope row | `envelope-row-{envelopeId}` | Per-envelope table row |
| Assigned cell | `assigned-cell-{envelopeId}` | Clickable assigned amount cell |
| Assigned input | `assigned-input-{envelopeId}` | Inline input shown when the assigned cell is in edit mode |
| Activity cell | `activity-cell-{envelopeId}` | Clickable activity amount cell |
| Available cell | `available-cell-{envelopeId}` | Available balance cell |
| Empty state (no envelopes) | `envelope-table-empty` | Hook rendered when there are no envelopes |
| Empty state (filter) | `filter-empty` | Hook rendered when the active filter matches no envelopes |

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | 1.0 | Initial spec | Finance App Team |
