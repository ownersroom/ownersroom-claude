---
name: portfolio-intelligence
description: Analyze investment portfolio data from OwnersRoom — portfolio summary, per-company cases, share holdings, transaction history, and option vesting. Use when the user asks about their portfolio, investments, returns, MOIC, holdings, or transaction history.
---

# Portfolio Intelligence

## Fetching Data

Portfolio tools are user-scoped (no `roomId` needed, except where noted).

| Tool | Parameters | Returns |
|------|-----------|---------|
| `get_portfolio_summary` | (none) | Top-level: `invested[]`, `realized[]`, `unrealized[]`, `moic` |
| `list_portfolio_cases` | (none) | Per-company: `name`, `identifier`, `invested[]`, `realized[]`, `unrealized[]`, `moic`, `assets[]` |
| `list_portfolio_holdings` | (none) | Share-level: `roomName`, `shareClass`, `holding`, `totalShares`, `ownership`, `invested[]`, `unrealized`, `moic` |
| `list_portfolio_events` | `roomId`, optional `after` | Transaction history: `registrationDate`, event type, share/option details |
| `get_portfolio_vesting` | `caseIdentifier`, optional `poolId` | Option vesting for a case: `grants[]`, `vestingHistory` with data points |

### Recommended Workflow

1. Start with `get_portfolio_summary` for the big picture
2. Drill into `list_portfolio_cases` for per-company breakdown
3. Use `list_portfolio_holdings` for share-level detail
4. Use `list_portfolio_events` for transaction history (roomId from cases or `list_rooms`)
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
| **Pie chart** | `list_portfolio_cases` → invested amounts | Portfolio allocation across companies |
| **Bar chart** | `list_portfolio_cases` → moic per case | Performance comparison across companies |
| **Line chart** | `get_portfolio_vesting` → dataPoints | Vesting progress over time |
| **Table** | `list_portfolio_holdings` | Holdings with ownership percentages |
| **Timeline** | `list_portfolio_events` | Transaction history chronologically |

## Analysis Patterns

- **Portfolio composition**: which companies represent the largest allocation (by invested amount)
- **Performance ranking**: sort cases by MOIC to identify best and worst performers
- **Realized vs unrealized**: how much has been cashed out vs. paper value
- **Concentration risk**: what percentage of portfolio is in the top 1-2 companies
- **Transaction audit**: use events to trace the history of purchases, grants, and exercises
