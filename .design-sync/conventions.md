# Settgast UI — design conventions

## Context

This design system is built for **financial dashboards** — market data, portfolio views, trading interfaces. All component choices and color semantics reflect that context.

## Color semantics

| Token | Use |
|---|---|
| `--color-accent` | Every interactive element: buttons, links, active tabs, selected state, focus rings |
| `--color-positive` | Financial gains, success states — green |
| `--color-negative` | Financial losses, error states — red |
| `--color-brand` | Identity only (logos, brand moments). **Never use near financial data** — it is red-adjacent and will be misread as a loss signal |
| `--color-text-secondary` | Labels, supporting info, placeholder text |
| `--color-text-muted` | Timestamps, metadata, tertiary content |
| `--color-surface` | Card interiors, panel backgrounds (one step above `--color-bg`) |

## Typography

- **Body / UI**: Inter (`--font-sans`) — weights 400 (regular), 500 (medium), 600 (semibold)
- **Code / mono / ticker symbols**: JetBrains Mono (`--font-mono`) — weights 400, 500
- Use mono for price figures, quantity values, and symbol names (BTC-USD, AAPL, etc.)

## Spacing rhythm

4px base unit. Standard steps: `--space-1` (4), `--space-2` (8), `--space-3` (12), `--space-4` (16), `--space-6` (24), `--space-8` (32).

## Component patterns

**Badge variants map to semantics**: use `positive`/`negative` for financial movement indicators, `accent` for live/active status, `warning` for pending/review states, `default` for neutral labels.

**StatCard**: the canonical pattern for a single KPI — label above, large value, trend change below. For grids of metrics, compose 3–4 StatCards in a row.

**Table**: designed for watchlist/portfolio data. Columns with financial values should right-align; use Badge for status/change columns.

**Sparkline**: a micro-chart that rides inline with other data (e.g. inside a Table row or a StatCard). Always pair it with a StatCard or Table context — it is never standalone content.

**Tabs**: use for switching between related views (Chart / Orders / History) within a panel. Don't use Tabs for top-level page navigation.

**Modal**: always triggered by a button (`triggerLabel` prop). The modal's content should be self-contained — don't nest another Modal.

**Pagination**: belongs below a Table. The `onChange` callback should update the table's data slice, not navigate the page.

**Select vs Input**: Select for constrained option lists (market, symbol, timeframe). Input for freeform text/numbers. Use Input `status` prop to show validation feedback inline (never a separate error element).

## Dark mode

All tokens have a `[data-theme="dark"]` variant. Apply `data-theme="dark"` to the root element to activate it. The accent color shifts from `#2563eb` to `#3b82f6` in dark mode — no manual override needed.

## Layout patterns

Financial dashboards use a **sidebar + main content** layout. Typical main panel: top row of StatCards, below that a Table or chart + Sparklines. Keep dense — financial users expect information-rich layouts.
