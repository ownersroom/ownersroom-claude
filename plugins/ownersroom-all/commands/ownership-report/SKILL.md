---
name: ownership-report
description: Generate a comprehensive ownership report for a company, showing shareholders, share classes, and ownership percentages.
---

# Ownership Report

Generate a formatted ownership report for a company.

## Steps

1. Call `list_rooms` to list accessible companies. Present the list and ask the user which company to analyze (unless context makes it obvious).

2. Once the room ID is known, fetch data in parallel:
   - `list_share_classes(roomId)`
   - `list_shareholders(roomId)`

3. Cross-reference the data:
   - For each shareholder, match `assets.shares[].assetId` to share class `id`
   - Compute ownership per class: `holding / totalShares`
   - Compute total ownership: sum of holdings / sum of all totalShares

4. Present the report:

   **Header**: Company name, total share capital, number of share classes

   **Share Classes Table**:
   | Class | Shares | Nominal Value | Share Capital | Shareholders |
   |-------|--------|--------------|---------------|-------------|

   **Ownership Table** (sorted by total ownership descending):
   | Shareholder | Class A | Class B | ... | Total Shares | Ownership % |
   |------------|---------|---------|-----|-------------|-------------|
   | ... | ... | ... | ... | ... | ... |
   | **Total** | ... | ... | ... | ... | **100%** |

   **Key Insights**:
   - Top 3 shareholders and their combined ownership
   - Whether ownership is concentrated or distributed
   - Any notable patterns (e.g., single majority holder, equal split)

5. Offer follow-up analysis: dilution modeling, options impact, or drill into a specific shareholder.
