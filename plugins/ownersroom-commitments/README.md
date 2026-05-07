# OwnersRoom Commitments for Claude Code

> **Experimental.** Internal prototype exploring AI-assisted workflows for equity data. Not a supported OwnersRoom product. Not intended for end users of OwnersRoom.

A focused install for **PE / VC fund managers (GPs)** — manage LP commitments, run capital calls and distributions, configure hurdles, send subscription docs and side letters out for e-signature, and communicate with LPs. ~27 tools.

## Install

```
/plugin marketplace add ownersroom/ownersroom-claude
/plugin install ownersroom-commitments@ownersroom
```

## What you get

### MCP tools (~27)

- **Identity & rooms** — current user, list rooms, per-room capabilities.
- **Cap-table reads (minimal)** — share classes, shareholders, capital events for portfolio companies you have access to.
- **Fund operations** — get fund, list LP commitment holders.
- **Commitment events** — create commitment closes, transfers; create capital calls, distributions, equalizations; update equalizations; delete capital events.
- **Interest rate schedules** — list / create / update / delete schedules used for fund hurdle rates.
- **Portfolio reads** — fund's portfolio summary, cases, holdings, and transaction history (the fund's own portfolio of investments).
- **Document e-signature** — initiate / cancel / prolong signing requests against existing documents (subscription agreements, side letters, etc.); send reminders; update a signatory entry. (No file uploads — references documents by path.)
- **News (full)** — read, draft, edit, preview, publish, delete LP letters and announcements.

Writes that touch a room respect your per-room permissions automatically — Claude is told exactly which permission is missing if a write isn't allowed. Every write tool exposes a typed JSON-Schema for its parameters.

### MCP Resources

The server also exposes **read-only Resources** for passive context: enums (currencies, post-visibilities, day-count conventions, capitalization frequencies, capital-event kinds), identity (`me://`, `me://capabilities` for one-shot user + per-room capability matrix), rooms (`rooms://`, `room://{id}`, `room://{id}/capabilities`), per-room reads (share classes, shareholders, interest-rate schedules, fund, commitment holders, signing requests), and templated reads (`room://{id}/posts/{postId}`).

### MCP Prompts

Saga templates that wrap multi-step workflows. Clients advertise them via `prompts/list`; invoke by name with optional arguments.

- **`capital_call_cycle`** — single capital-call cycle: discover via `room://{id}/fund` and `room://{id}/fund/commitment-holders` → `create_capital_call` (real financial event) → optional `create_capital_equalization` (heuristic: only when there's been a close since the last call) → LP notification post (`create_post` → optional `preview_post_email` → `publish_post`). Stops to confirm before issuing the call and before publishing.
- **`quarterly_lp_letter`** — quarterly LP letter: gather fund + commitment-holder + recent capital-event context → caller-provided portfolio updates (the saga doesn't auto-discover — `list_portfolio_cases` is portfolio-persona only; the user is directed to a `ownersroom-portfolio` session if they want Claude to gather it) → structured `create_post` → `preview_post_email` → `publish_post`. Stops to confirm before publishing.

### Skills

- **news-updates** — drafting, previewing, and publishing LP communications.

(More fund-specific skills are TBD — feedback welcome.)

## What this plugin does *not* include

- Cap-table admin (shares, options, employee equity) — use **`ownersroom-captable`**.
- Investor-side portfolio book management — use **`ownersroom-portfolio`**.

LP investors viewing *their* fund holdings should install **`ownersroom-portfolio`**, not this plugin — `portfolio` lists holdings cross-room, which is what an LP actually wants.

## When to install this vs. the full plugin

- Install **`ownersroom-commitments`** if your work is GP-side fund operations only.
- Install **`ownersroom`** (kitchen sink) if you also need cap-table admin or your own portfolio book.
- **Don't install both** — pick one. The kitchen sink already includes everything in this plugin.
