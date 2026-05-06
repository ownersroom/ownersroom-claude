---
name: portfolio-report
description: Generate a portfolio overview showing investment summary, per-company performance, and key metrics like MOIC.
---

# Portfolio Report

Generate a comprehensive portfolio report for the current user.

## Steps

1. Fetch headline metrics:
   - `get_portfolio_summary`

2. Fetch per-company breakdown:
   - `list_portfolio_cases`

3. Present the report:

   **Portfolio Summary**:
   | Metric | Value |
   |--------|-------|
   | Total Invested | (sum per currency) |
   | Total Realized | (sum per currency) |
   | Total Unrealized | (sum per currency) |
   | Overall MOIC | (from summary) |

   **Per-Company Performance** (sorted by MOIC descending):
   | Company | Invested | Unrealized | Realized | MOIC |
   |---------|----------|-----------|----------|------|
   | ... | ... | ... | ... | ... |

   **Key Insights**:
   - Best performer (highest MOIC) and worst performer
   - Portfolio concentration: what % of invested capital is in the top company
   - Realized vs unrealized split: how much is paper value vs. cashed out
   - Any data quality flags (outdated estimates, incomplete pricing)

4. Offer drill-down options:
   - "Would you like to see share holdings detail?" → `list_portfolio_holdings`
   - "Would you like to see transaction history for a company?" → `list_portfolio_events(roomId)`
   - "Would you like to see option vesting for a company?" → `get_portfolio_vesting(caseIdentifier)`
