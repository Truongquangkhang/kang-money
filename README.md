# Kang Money

Design specs for a personal expense management app built around **envelope budgeting** (hũ). The app is VND-only and currently targets a single-user, single-theme (light) release.

This repository is a **spec-first design system**: every screen and component is documented as a behavior contract before implementation, all referencing a shared set of design tokens.

## Structure

```
docs/
├── Theme.md                 # Canonical design tokens (color, type, spacing, radius, shadow, z-index, breakpoints, motion)
├── screens/                 # Full-screen behavior contracts (+ HTML prototypes)
│   ├── BudgetOverview.md/.html
│   └── AllAccounts.md/.html
└── components/              # Reusable component specs, grouped by category
    ├── data/EnvelopeTable.md
    ├── feedback/AlertBar.md, EmptyState.md, ErrorState.md, LoadingState.md
    ├── input/InlineEdit.md
    └── overlay/ActivityPopup.md, Modal.md
```

## Screens

- **Budget Overview** — the core screen: allocate money across envelopes for the current month via a two-panel layout (envelope table + selection-driven detail panel).
- **All Accounts** — the unified transaction ledger across all accounts: review, add/edit, clear, search, and filter transactions.

## Conventions

- Specs reference design tokens by dot path (e.g. `color.text.positive`); implementations expose them as CSS custom properties (e.g. `--color-text-positive`). Raw hex/px/ms values should never appear in a spec.
- Out-of-scope features for the current release (shared budgets, targets/auto-assign, multi-currency, split transactions, recurring transactions, dark theme) are called out explicitly at the top of the relevant spec.
- See `docs/Theme.md` §12 for the full token reference index and §13 for open questions/known aliases.
