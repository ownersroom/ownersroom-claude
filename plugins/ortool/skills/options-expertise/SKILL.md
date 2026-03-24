---
name: options-expertise
description: Interpret options data and vesting schedules from OwnersRoom — option pools, grants, vesting progress, strike prices, and exercise analysis. Use when the user asks about options, vesting, grants, strike price, exercise cost, or option pools.
---

# Options Expertise

## Fetching Data

All options tools require a `roomId`. Call `list_rooms` first if the room ID is unknown.

| Tool | Parameters | Returns |
|------|-----------|---------|
| `list_option_pools` | `roomId` | Option pools: `id`, `name`, `poolSize`, `count`, `expectedShares`, `isArchived` |
| `list_option_grants` | `roomId`, optional `poolId`, `actorId` | Individual grants: `actor`, `optionPool`, `grantedCount`, `strikePrice`, `vestingStartDate`, `vestingEndDate`, `vestingProgress` |
| `list_option_holders` | `roomId` | Aggregated per-person: `totalGrantedCount`, `totalVestedCount`, `totalUnvestedCount`, `totalExercisedCount`, `totalCancelledCount`, `activeOptionsCount`, `vestingPercentage` |
| `get_vesting_history` | `roomId`, `actorId`, optional `poolId` | Vesting timeline: `totalActiveOptions`, `dataPoints[].timestamp`, `dataPoints[].vestedCount`, `dataPoints[].vestedPercentage` |

## Key Concepts

### Vesting Math
- **Vested** (`vestedCount`) = options that can be exercised now
- **Unvested** = `totalGrantedCount - totalVestedCount - totalCancelledCount - totalExercisedCount`
- **Active** (`activeOptionsCount`) = granted - cancelled - exercised (i.e., still in play)

### Option Pool Utilization
- **Pool size** (`poolSize`) = total authorized options
- **Grants issued** (`count`) = options allocated to people
- **Available** = `poolSize - count`
- **Utilization** = `count / poolSize` as a percentage

### Strike Price & Value
- **Strike price** = exercise cost per option (what the holder pays)
- **In the money** = current share value > strike price
- **Out of the money** = current share value < strike price
- **Net gain per option** = market value - strike price (when in the money)
- **Total exercise cost** = strike price × vested count

## Vesting Timeline Graphing

The `get_vesting_history` tool returns chronological data points for graphing:

- **X-axis**: timestamp (dates)
- **Y-axis**: vestedCount or vestedPercentage
- **Cliff events** appear as sharp vertical jumps (e.g., 0% → 25% on cliff date)
- **Linear vesting** appears as a steady upward slope between cliffs

When generating charts, include:
- A horizontal line at 100% for reference
- Labels for cliff dates
- Current date marker showing progress

## Analysis Patterns

- **Total option exposure**: sum `activeOptionsCount` across all holders
- **Vesting cliff dates**: identify grants with upcoming cliff events (vestingStartDate + cliff duration)
- **Exercise cost analysis**: total cost = Σ(strikePrice × vestedCount) across all grants for an actor
- **Pool runway**: available options vs. planned future grants
- **Holder comparison**: table of holders ranked by total granted or vesting percentage
