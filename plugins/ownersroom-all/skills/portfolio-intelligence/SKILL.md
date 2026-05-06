---
name: portfolio-intelligence
description: Analyze investment portfolio data from OwnersRoom — portfolio summary, per-company cases, share holdings, transaction history, and option vesting. Use when the user asks about their portfolio, investments, returns, MOIC, holdings, or transaction history.
---

# Portfolio Intelligence

## Fetching Data

Portfolio data is user-scoped (no `roomId` needed, except where noted).

| Source | Parameters | Returns |
|--------|-----------|---------|
| `portfolio://summary` Resource | (none) | Top-level: `invested[]`, `realized[]`, `unrealized[]`, `moic` |
| `portfolio://cases` Resource | (none) | Per-company: `name`, `identifier`, `invested[]`, `realized[]`, `unrealized[]`, `moic`, `assets[]` |
| `portfolio://holdings` Resource | (none) | Share-level: `roomName`, `shareClass`, `holding`, `totalShares`, `ownership`, `invested[]`, `unrealized`, `moic` |
| `list_portfolio_events` tool | `roomId`, optional `after` | Transaction history (paginated): `registrationDate`, event type, share/option details |
| `get_portfolio_vesting` tool | `caseIdentifier`, optional `poolId` | Option vesting for a case: `grants[]`, `vestingHistory` with data points. The `portfolio://cases/{caseIdentifier}/vesting` Resource is the no-filter alternative. |

### Recommended Workflow

1. Start with `portfolio://summary` for the big picture
2. Drill into `portfolio://cases` for per-company breakdown
3. Use `portfolio://holdings` for share-level detail
4. Use `list_portfolio_events` for transaction history (roomId from cases or `rooms://`)
5. Use `get_portfolio_vesting` for option vesting (caseIdentifier from cases)

## Key Concepts

### MOIC (Multiple on Invested Capital)
- **Formula**: total value / invested capital
- **MOIC > 1.0** = positive return (e.g., 2.5x = 150% gain)
- **MOIC < 1.0** = loss
- Available at both portfolio level and per case

### Cases vs Holdings
- **Cases** = company-level view. Aggregates shares and options for each company. Includes `invested`, `realized`, `unrealized`, `moic`.
- **Holdings** = share-level detail. One row per share class per company. Includes `ownership` percentage, `holding` count, `totalShares`.

### Monetary Values
- All amounts include a `currency` field (e.g., `NOK`, `EUR`, `USD`)
- The portfolio may span multiple currencies
- **Always group and sum within the same currency**
- Flag cross-currency comparisons as approximate

### Data Quality Flags
- `outdatedEstimate: true` — the valuation may be stale
- `incompletePricing: true` — not all assets have pricing data
- When either is true, note values as approximate in any analysis

## Graphing & Visualization

Portfolio data is designed for visualization:

| Chart Type | Data Source | What It Shows |
|-----------|-------------|--------------|
| **Pie chart** | `portfolio://cases` → invested amounts | Portfolio allocation across companies |
| **Bar chart** | `portfolio://cases` → moic per case | Performance comparison across companies |
| **Line chart** | `get_portfolio_vesting` → dataPoints | Vesting progress over time |
| **Table** | `portfolio://holdings` | Holdings with ownership percentages |
| **Timeline** | `list_portfolio_events` | Transaction history chronologically |

## Analysis Patterns

- **Portfolio composition**: which companies represent the largest allocation (by invested amount)
- **Performance ranking**: sort cases by MOIC to identify best and worst performers
- **Realized vs unrealized**: how much has been cashed out vs. paper value
- **Concentration risk**: what percentage of portfolio is in the top 1-2 companies
- **Transaction audit**: use events to trace the history of purchases, grants, and exercises

## Modifying data

Portfolio writes operate on the user's own portfolio. Unlike room-scoped writes, **they are not gated by per-room capability flags** — the user always controls their own portfolio. No capability check needed.

| Tool | Use for |
|------|---------|
| `create_portfolio_cases` | Create one or more portfolio cases. `cases` is an array of `CaseInput`: `{ name, identifier, country, assets[], owner }`. |
| `update_portfolio_case` | Update a case's name / identifier / country / assets / owner |
| `delete_portfolio_case` | Remove a case (destructive — also removes its assets) |
| `create_portfolio_asset` | Add an asset to a case. `asset` is an `AssetInput`: `{ name, invested, realized, estimatedPrice, quantity }`. |
| `update_portfolio_asset` | Update an asset within a case |
| `delete_portfolio_asset` | Remove an asset from a case (destructive) |
| `update_estimated_asset_value` | Update the user's own estimated value for an asset. Identify the asset either by `assetId` or by `holding: { actorId, shareClassId }`. `price` is `{ amount, currency }`; optional `note` records the context. |

### Safe write pattern

1. **Read** current state with the `portfolio://cases` / `portfolio://holdings` Resources so you can show what's there before a change.
2. **Propose** the change with concrete numbers. For estimated values, show the previous estimate (and whether it was flagged `outdatedEstimate`).
3. **Confirm** with the user.
4. **Write**, then **read back** to confirm.

### `update_estimated_asset_value` notes

Estimated values are user-specific signals (your view of what an investment is worth) — not the company's official price. Two ways to identify which value to update:

- **`assetId`** — for assets already in your portfolio (most common).
- **`holding: { actorId, shareClassId }`** — when you want to update the value of a holding that may not yet have an explicit asset row.

Always set `note` if the user provided context for the estimate (e.g., "based on Q1 trading round").

## User profile

`update_user` lets the authenticated user update their profile (name, email, country, address, about/description). Required fields: `firstName`, `lastName`, `email`, `country` (ISO 3166-1 alpha-2). Confirm with the user before changing email — it affects login and notifications.
