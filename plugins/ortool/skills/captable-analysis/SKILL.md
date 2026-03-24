---
name: captable-analysis
description: Analyze cap table data from OwnersRoom — ownership percentages, share class distribution, shareholder concentration, and dilution modeling. Use when the user asks about ownership, shareholders, share classes, or cap tables.
---

# Cap Table Analysis

## Fetching Data

Use the ortool MCP tools to retrieve data. All responses are JSON.

1. **Find the room** — call `list_rooms` to list companies the user has access to. Each room has an `id` (the room ID) and a `company.name`.
2. **Fetch share classes and shareholders in parallel**:
   - `list_share_classes(roomId)` — returns share classes with `id`, `name`, `totalShares`, `nominalValue`, `currency`, `shareCapital`, `totalShareholders`
   - `list_shareholders(roomId)` — returns shareholders with `actor.displayName`, `assets.shares[].assetId` (matches share class ID), and `assets.shares[].holding`

## Calculating Ownership

Each shareholder's holdings are **per share class**. To compute ownership:

- **Per-class ownership** = `holding / totalShares` for that share class
- **Total shares across all classes** = sum of `totalShares` from all share classes
- **Total holding** = sum of all holdings across classes for a shareholder
- **Overall ownership %** = total holding / total shares across all classes

Note: different share classes (A, B, ordinary, preference) may have different voting rights or liquidation preferences. The data does not encode these differences — flag them as context-dependent when relevant.

## Share Capital

- `shareCapital` = `nominalValue × totalShares` per class
- Total share capital = sum across all classes (ensure same currency)

## Analysis Patterns

- **Ownership concentration**: identify top 3-5 shareholders and their combined ownership percentage
- **Share class distribution**: how total shares and capital split across classes
- **Dilution modeling**: if the user provides a number of new shares, compute how existing ownership percentages change: `new_ownership = old_shares / (total_old_shares + new_shares)`

## Presentation

- Always show both **absolute share counts** and **percentages**
- Use tables for multi-shareholder comparisons
- Format monetary values with currency symbols (e.g., NOK 1,000,000)
- Sort shareholders by ownership descending
- Include a total/sum row at the bottom
