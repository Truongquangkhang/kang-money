# Component: InlineEdit

**Status:** In Design
**Version:** 1.0
**Last Updated:** 2026-06-27
**Category:** input

> Click-to-edit field that renders a static value and swaps to an in-place input on tap; commits on Enter or blur, reverts on Escape. Supports plain text and VND-currency formats.

---

## 1. Purpose

`InlineEdit` lets the user edit a single value directly in place, without a separate form or modal. The field shows a formatted, read-only display by default; clicking (or tapping) it swaps the display for a focused input. The user types, then commits with `Enter` or by blurring the field, or cancels with `Escape` to revert to the original value.

It is used in two places on the Budget Overview screen (see `docs/screens/BudgetOverview.md` §4):

- **Assigned amount** — the editable VND amount in each Assigned column cell of the envelope table, and the `Assigned This Month` row of the Detail Panel balance breakdown. Uses `format = 'currency-vnd'`.
- **Envelope name** — the envelope name in the Detail Panel header, activated by clicking the name or the ✏️ edit icon. Uses `format = 'text'`.

The component is intentionally format-aware but otherwise generic: the same commit/cancel contract, state machine, and keyboard behavior apply to both usages. Format-specific concerns (VND parsing/formatting, `min`; or `maxLength` for text) are gated by the `format` prop.

This component owns **only** the inline display ⇄ edit interaction for one value. It does not own persistence, recalculation, or the single-active-cell policy — those belong to the parent (the screen / table). The parent wires `onCommit` to persist and recompute; the parent enforces "one cell editable at a time" by leaning on the commit-on-blur behavior documented in §6.

---

## 2. Anatomy

The component has exactly two visual modes that swap in place — the bounding box does not move.

### Display mode (text)

```
+------------------------------------------+
|  Groceries                            ✏️ |   ← static text, click anywhere to edit
+------------------------------------------+
```

### Edit mode (text)

```
+------------------------------------------+
| [ Groceries|                           ] |   ← focused <input>, caret active
+------------------------------------------+
   ^ border: color.border.active
   Enter / blur → commit · Escape → cancel
```

### Display mode (currency-vnd)

```
+--------------------+
|        1.500.000đ  |   ← formatted VND, dot thousands + đ suffix, right-aligned
+--------------------+
```

### Edit mode (currency-vnd)

```
+--------------------+
| [    1,500,000|  ] |   ← raw numeric input, comma thousands while typing, no đ
+--------------------+
   ^ border: color.border.active
   stored as integer 1500000 · Enter / blur → commit · Escape → revert
```

> **Display ⇄ Edit is a swap, not a layout shift.** Display formatting (`đ` suffix, dot separators) is replaced by the typing format (comma separators, no suffix) for the duration of edit mode, then re-rendered on commit.

---

## 3. Props

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| `value` | `number \| string` | Yes | — | The current committed value. Integer VND amount when `format = 'currency-vnd'`; string when `format = 'text'`. This is the source of truth the field reverts to on cancel. |
| `format` | `'text' \| 'currency-vnd'` | No | `'text'` | Display/parse mode. `'text'` renders the raw string; `'currency-vnd'` formats per the app VND rules (§9) and constrains input to non-negative integers. |
| `onCommit` | `(next: number \| string) => void` | Yes | — | Called when the edit commits (Enter or blur) with a new value. For currency, receives a raw integer (e.g. `1500000`). The parent persists and triggers any recalculation. Not called when the value is unchanged unless the parent opts in (idempotent commit is the parent's choice). |
| `onCancel` | `() => void` | No | — | Called when the edit is cancelled (Escape). The field reverts to `value`; the parent typically needs no action but may clear active-edit state. |
| `min` | `number` | No | `0` | **Currency only.** Minimum allowed value. Values below `min` are rejected by the input. With the default `0`, negative entry is blocked and `0` is a valid commit (removes the allocation). Ignored when `format = 'text'`. |
| `maxLength` | `number` | No | — | **Text only.** Maximum character count for the input. Entry beyond `maxLength` is blocked. Ignored when `format = 'currency-vnd'`. Detail Panel envelope name uses `50`. |
| `placeholder` | `string` | No | `''` | Placeholder shown in the input when the value is empty. |
| `disabled` | `boolean` | No | `false` | When `true`, the field renders display mode only and does not enter edit mode on click. |
| `commitOnBlur` | `boolean` | No | `true` | When `true`, blurring the input commits the current value (same as Enter). When `false`, blur cancels/reverts instead. The single-active-cell policy depends on the default `true` (see §6). |
| `data-testid` | `string` | No | — | Test id applied to the component root. The input and display sub-elements derive their ids from it (see §11). The screen passes `assigned-cell-{envelopeId}` / `envelope-name` here. |

> **Format gating.** `min` applies only to `currency-vnd`; `maxLength` applies only to `text`. Passing the inapplicable prop is a no-op, not an error.

---

## 4. Variants

| Variant | `format` | Display | Input behavior | Used by |
|---|---|---|---|---|
| **Currency (VND)** | `currency-vnd` | Dot-separated thousands + `đ` suffix, e.g. `1.500.000đ`; `0đ` when zero | Numeric only; comma-separated thousands rendered while typing; non-negative integer; respects `min` | Assigned column cell (`assigned-cell-{envelopeId}`), `Assigned This Month` breakdown row (`balance-assigned`) |
| **Text** | `text` | Raw string as-is | Free text; respects `maxLength`; no number coercion | Detail Panel envelope name (`envelope-name`) |

Both variants share one state machine (§5), one keyboard contract (§6), and one set of design tokens (§8). The only differences are display formatting, input parsing, and which constraint prop applies (`min` vs `maxLength`).

---

## 5. States

| State | Description |
|---|---|
| `display` | Read-only formatted value shown. Clicking/tapping (when not `disabled`) enters `editing`. |
| `editing` | Focused input shown with the editable representation of the value (raw integer-with-commas for currency, raw string for text). Active border `color.border.active`. Awaiting commit or cancel. |
| `committing` | Transient state after a commit is requested: the field shows the new value optimistically while `onCommit` runs (e.g. persistence + recalculation). On success the field settles back to `display`; if the parent rejects the change it may pass a corrected `value` and the field re-renders `display` from it. |

### State machine

```
        click / tap (not disabled)
display ───────────────────────────► editing
   ▲                                     │
   │                                     │ Enter  ─┐
   │                            blur ────┤         ├─► commit  ──► committing
   │      (commitOnBlur = true)          │         │
   │                                     │ Enter   ─┘
   │                                     │
   │            Escape  /  blur          │
   │        (commitOnBlur = false)       │
   └──────────────── cancel / revert ────┘
                                         │
committing ──── onCommit settles ───────► display
            (re-render from parent value)
```

> Transitions are local component state. The parent observes them only through `onCommit` / `onCancel`. There is no error state inside the component — validation failures (below `min`, negative) are prevented at the input level, so a commit is always a valid value.

---

## 6. Interaction Rules

- **Entering edit mode.** A click/tap anywhere on the display (or pressing `Enter`/`Space` when the field is keyboard-focused) enters `editing`, focuses the input, and selects its contents so the user can overtype. No effect when `disabled`.
- **Commit — Enter.** Pressing `Enter` commits the current input value, fires `onCommit(next)`, and returns to `display`.
- **Commit — blur.** When `commitOnBlur = true` (default), the input losing focus commits exactly as Enter does. This is the mechanism the **single-active-cell** policy relies on: when the user clicks a second Assigned cell, the first cell's input blurs and commits before the second cell opens. The parent does not need to manually flush the first edit — committing on blur guarantees no edit is silently dropped.
- **Cancel — Escape.** Pressing `Escape` discards the input, reverts to `value`, fires `onCancel()`, and returns to `display`. Escape always cancels regardless of `commitOnBlur`.
- **Blur when `commitOnBlur = false`.** Blur cancels and reverts (treated as Escape) instead of committing.
- **Currency input constraints.**
  - VND integer only — no decimal point, no fractional entry.
  - Comma-separated thousands are rendered live while typing for readability (typing `1500000` shows `1,500,000`); the value passed to `onCommit` is the raw integer `1500000`.
  - Negative values are rejected by the input (a leading `-` is not accepted); `min` (default `0`) is the floor.
  - `0` is a valid value and commits normally — for the Assigned usage this removes the envelope's allocation for the month.
- **Text input constraints.** Free text up to `maxLength` characters; entry beyond the limit is blocked. Leading/trailing whitespace handling (trim on commit) is the parent's decision via `onCommit`.
- **Unchanged value.** If the committed value equals `value`, the component still returns to `display`. Whether `onCommit` is invoked for an unchanged value is left to the parent contract (the Assigned/name usages treat commit as idempotent).
- **One field at a time.** The component does not coordinate with sibling instances. The parent enforces "only one cell editable at a time" by relying on commit-on-blur: opening a new edit blurs and commits any currently open one.

---

## 7. Accessibility

- **Focusable display.** In `display` mode the field is keyboard-focusable (`tabindex="0"` on the display element, or a native focusable control) so it can be entered without a pointer. `Enter` / `Space` activates edit mode.
- **Native input in edit mode.** `editing` mode renders a real `<input>` (numeric input mode for currency: `inputmode="numeric"`; text input for names), fully operable by keyboard: type, Enter to commit, Escape to cancel, Tab to blur (which commits when `commitOnBlur`).
- **Label association.** The input is associated with a label describing the field (e.g. "Assigned amount for {envelope}", "Envelope name") via `aria-label` or an `aria-labelledby` reference, so screen readers announce purpose on focus.
- **Announce on commit.** On a successful commit, the new formatted value is announced (e.g. via an `aria-live="polite"` region or by re-focusing the updated display) so non-visual users get confirmation that the edit took effect.
- **State exposure.** The display element exposes its editable nature (`role` / `aria` indicating it acts as a button-to-edit); the input exposes constraints (`aria-valuemin` reflecting `min` for currency; `maxlength` for text).
- **Disabled.** When `disabled`, the field is not focusable and not activatable, and is marked `aria-disabled="true"`.
- **Focus return.** After commit or cancel, focus returns to the display element (not lost to `document.body`) so keyboard navigation continues from the same place.

---

## 8. Design Tokens

Token names reference `docs/Theme.md` (canonical token source).

| Element | Token |
|---|---|
| Display text color | `color.text.primary` |
| Display value (currency, positive) | `color.text.primary` |
| Placeholder text | `color.text.secondary` |
| Edit-mode input border (active) | `color.border.active` |
| Edit-mode input background | `color.bg.surface` |
| Edit-mode input text | `color.text.primary` |
| Edit icon (✏️) — text variant | `color.icon.secondary` |
| Disabled display text | `color.text.secondary` |
| Field background (display) | transparent (inherits row / panel surface) |

---

## 9. Content / Formatting

VND formatting follows the app-wide rules (`docs/screens/BudgetOverview.md` §9). The display representation and the typing representation differ:

| Context | Format | Example |
|---|---|---|
| Currency — displayed (read-only) | Dot-separated thousands, `đ` suffix | `1.500.000đ` |
| Currency — zero displayed | `0đ` | `0đ` |
| Currency — while typing (input) | Comma-separated thousands, no suffix, no decimals | `1,500,000` |
| Currency — stored / committed | Raw integer (no separators) | `1500000` |
| Text — displayed and typing | Raw string, no transformation | `Groceries` |

- **Currency parse/format round-trip:** display `1.500.000đ` → enter edit → input shows `1,500,000` → user types → commit emits integer `1500000` → display re-renders `1.500.000đ`.
- **No decimals.** VND has no minor unit in this app; the input does not accept `.` or `,` as a decimal separator (commas are thousands grouping only, inserted automatically — not typed).
- **Empty value.** Text variant with empty value shows `placeholder`. Currency variant has no empty state (a missing allocation is `0`, displayed `0đ`).
- **Text length.** Names are capped at `maxLength` (50 for the envelope name); display does not truncate within the field — truncation, if any, is the parent's layout concern.

---

## 10. Platform Notes

### Web Desktop
- Click to edit; `Enter`/blur commit, `Escape` cancel. Pointer click outside the input blurs it (committing when `commitOnBlur`).
- The active-border highlight (`color.border.active`) makes the editable input visually distinct from the surrounding cell/panel.

### Web Mobile
- Tap to edit; the on-screen keyboard appears. Currency variant requests a numeric keypad (`inputmode="numeric"`).
- Tapping elsewhere blurs and commits (the same mechanism the table uses to enforce single-active editing).

### Android / iOS
- Tap to edit raises the platform numeric keyboard for currency and the text keyboard for names.
- The keyboard "done"/return action commits (equivalent to `Enter`). Dismissing the keyboard blurs the input, which commits when `commitOnBlur = true`.
- No hover affordance; the ✏️ icon (text variant) signals editability on touch platforms.

---

## 11. Test IDs

The component applies the `data-testid` passed by the parent to its root and derives child ids from it. The screen-level ids below are the values the Budget Overview passes (`docs/screens/BudgetOverview.md` §13).

| Element | `data-testid` | Description |
|---|---|---|
| Component root (display mode) | `{data-testid}` | The clickable field in display mode; receives the id passed by the parent |
| Input (edit mode) | `{data-testid}-input` | The focused `<input>` rendered while in `editing` |
| Edit trigger icon (text variant) | `{data-testid}-edit-btn` | The ✏️ icon that activates edit mode for text fields |
| Assigned cell (screen usage) | `assigned-cell-{envelopeId}` | The Assigned amount field in display mode (root id passed by the table) |
| Assigned input (screen usage) | `assigned-input-{envelopeId}` | The Assigned amount input while editing |
| Envelope name (screen usage) | `envelope-name` | The Detail Panel name field in display mode |
| Envelope name input (screen usage) | `envelope-name-input` | The Detail Panel name input while editing |

> When the parent passes `assigned-cell-{envelopeId}` as `data-testid`, the screen expects the editing input id to be `assigned-input-{envelopeId}`. The component honors an explicit input-id override so the screen's `assigned-input-{envelopeId}` / `envelope-name-input` conventions are met exactly rather than via the generic `-input` suffix.

---

## Change History

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-06-27 | **1.0** | Initial spec — InlineEdit component. Unified click-to-edit field covering `text` and `currency-vnd` formats; commit on Enter/blur, cancel on Escape; single-active-cell via commit-on-blur; VND dot-display / comma-typing formatting; accessibility and test-id contract aligned with Budget Overview screen (§4, §13). | Finance App Team |
