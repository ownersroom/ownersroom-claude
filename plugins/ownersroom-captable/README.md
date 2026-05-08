# OwnersRoom Cap-Table for Claude Code

> **Experimental.** Internal prototype exploring AI-assisted workflows for equity data. Not a supported OwnersRoom product. Not intended for end users of OwnersRoom.

A focused install for **company admins** — manage your company's capital structure: share classes, share issuances, option pools, grants, vesting, employee loans, contacts (people, companies), deals (primary share offerings), document e-signature, and company communications. ~65 tools.

## Install

```
/plugin marketplace add ownersroom/ownersroom-claude
/plugin install ownersroom-captable@ownersroom
```

## What you get

### MCP tools (~65)

- **Identity & rooms** — current user, list rooms, per-room capabilities.
- **Shares** — list classes & shareholders; create / update / delete share classes, share issuances, transactions; delete capital events.
- **Options** — list pools, grants, holders; vesting history; create / update / delete pools and grants; cancel and exercise grants.
- **Employee loans** — list loans, repayments, simulate interest; create / update issuance and repayment events.
- **Interest rate schedules** — list / create / update / delete schedules used for loan accrual.
- **Contacts** — list people / companies; create / update Person and Company actors; search and look up organizations against external registries (e.g., Brønnøysund).
- **Deals (primary share offerings)** — create / open / close / cancel deals; manage participants, primary share offers, allocations, and payments; bridge a closed deal to the cap table as a share-issuance event.
- **Document e-signature** — initiate / cancel / prolong signing requests against existing documents, send reminder emails to outstanding signatories, update a signatory entry. (No file uploads — references documents by path.)
- **News** — read, draft, edit, preview, publish, delete company posts.

Writes that touch a room respect your per-room permissions automatically — Claude is told exactly which permission is missing if a write isn't allowed. Every write tool exposes a typed JSON-Schema for its parameters.

### MCP Resources

The server also exposes **read-only Resources** for passive context: enums (currencies, post-visibilities, day-count conventions, capitalization frequencies, capital-event kinds), identity (`me://`, `me://capabilities` for one-shot user + per-room capability matrix), rooms (`rooms://`, `room://{id}`, `room://{id}/capabilities`), per-room reads (share classes, shareholders, option pools, option holders, interest-rate schedules, people, companies, deals, signing requests), and templated reads (`room://{id}/posts/{postId}`, `room://{id}/deals/{dealId}`, `room://{id}/people/{actorId}/vesting`). Four paginated Resources (`option-holders`, `people`, `companies`, `signing-requests`) support forward-only cursor pagination via `{?after}` — re-read with `?after=<endCursor>` from the previous response's `pageInfo` to advance.

### MCP Prompts

Saga templates that wrap multi-step workflows. Clients advertise them via `prompts/list`; invoke by name with optional arguments.

- **`financing_round`** — primary financing round end-to-end: `create_deal` → set offer terms → add participants → `open_deal` (real emails) → `close_deal` → finalize allocations & payments → `add_deal_to_cap_table` (irreversible) → announcement post. Stops to confirm before each side-effecting write.

### Skills

- **captable-analysis** — ownership percentages, share class distribution, dilution modeling.
- **options-expertise** — option pools, grants, vesting, strike prices, exercise analysis.
- **news-updates** — drafting, previewing, and publishing company news posts.

### Slash commands

- **/ownership-report** — formatted ownership report showing shareholders, share classes, and percentages.

## What this plugin does *not* include

- LP commitment / fund operations (use **`ownersroom-commitments`**).
- Investor-facing portfolio tools (use **`ownersroom-portfolio`**).

## When to install this vs. the full plugin

- Install **`ownersroom-captable`** if your work is company-internal cap-table admin only.
- Install **`ownersroom`** (kitchen sink) if you also need fund operations or your own portfolio book.
- **Don't install both** — pick one. The kitchen sink already includes everything in this plugin.
