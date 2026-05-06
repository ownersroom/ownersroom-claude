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

## Modifying data

Option pools, grants, cancellations, and exercises are all writable. **Every options write requires `capTable.manageOptions`** in the target room (the schema's `manageOptions` flag covers view, create, edit, cancel, and exercise). Call `get_room_capabilities(roomId)` before a write to confirm.

| Tool | Use for |
|------|---------|
| `create_option_pool` | Create a new option pool (`OptionType`). After creation, `count` is calculated from the ledger and cannot be overridden. |
| `update_option_pool` | Rename, resize, or change the share class / default vesting schedule on a pool |
| `delete_option_pool` | Remove a pool. **Destructive.** Fails if the pool still has active grants — cancel grants first. |
| `create_option_grants` | Issue one or more grants under a single capital event. All grants in the call share the registration date and event description. |
| `update_option_grant_event` | Edit grants within an existing event. The `grants` array is the **full list** for the event: existing grants keep their `grantId`, new grants omit it, grants you remove disappear. Shared fields (`vestingStartDate`, `strikePrice`, etc.) apply to all grants in the event. |
| `cancel_option_grant` | **Destructive.** Without `cancelAmount`, cancels all remaining options. With `cancelAmount`, partial-cancels. Set `validateUnvestedOnly: true` to refuse if the cancel would touch vested options. |
| `exercise_option_grant` | Exercise vested options. Creates a capital event and a ledger entry. `exerciseAmount` must not exceed vested unexercised options. |

### Safe write pattern

Options affect compensation. Before any write:

1. **Read** current state — `list_option_grants` for a person, `list_option_pools` for a pool, `get_vesting_history` for a vesting timeline.
2. **Propose** the change with concrete numbers (counts, dates, strike prices). Distinguish vested vs unvested clearly.
3. **Confirm** with the user. Never assume a write is approved.
4. **Write**, then **read back** to confirm the resulting state matches expectations.

Destructive operations (`cancel_option_grant`, `delete_option_pool`) are non-recoverable. For `cancel_option_grant`, prefer `validateUnvestedOnly: true` unless the user has explicitly asked to cancel vested options.

### `exercise_option_grant` guidance

Exercising is a real money event for the holder (they pay `strikePrice × exerciseAmount`). Always show the total exercise cost before calling. If you can compute current market value (from a recent share price), also show whether the grant is in the money.

### Permission failures

If a write returns `action_not_allowed`, the user does not have `capTable.manageOptions` in this room. Surface the exact requirement and offer to find a different room via `list_rooms` if relevant — don't retry.
