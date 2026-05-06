# OwnersRoom Portfolio for Claude Code

> **Experimental.** Internal prototype exploring AI-assisted workflows for equity data. Not a supported OwnersRoom product. Not intended for end users of OwnersRoom.

A focused install for **investors** — track your portfolio, view holdings and vesting, read news from companies you've invested in. ~13 tools. Smaller token footprint than the full `ownersroom-all` plugin (~56 tools).

## Install

```
/plugin marketplace add ownersroom/ownersroom-claude
/plugin install ownersroom-portfolio@ownersroom
```

## What you get

### MCP tools (~13)

- **Identity & rooms** — current user, list rooms you can access, per-room capabilities.
- **Cap-table reads** — share classes, shareholders, capital events for companies you've invested in (subject to room permissions).
- **Vesting** — your own option vesting timelines.
- **Portfolio (read)** — summary, per-company breakdown, holdings, transaction history, vesting per case.
- **Portfolio (manage your book)** — create / update / delete portfolio cases and assets, update estimated values.
- **News (read)** — read posts from companies in your portfolio.

### MCP Resources

The server also exposes **read-only Resources** for passive context: identity (`me://`, `me://capabilities` for one-shot user + per-room capability matrix), rooms (`rooms://`, `room://{id}`, `room://{id}/capabilities`), per-room reads (`room://{id}/share-classes`, `room://{id}/shareholders`, `room://{id}/posts/{postId}`, `room://{id}/people/{actorId}/vesting`), and portfolio (`portfolio://summary`, `portfolio://cases`, `portfolio://holdings`, `portfolio://cases/{caseIdentifier}/vesting`).

### Skills

- **portfolio-intelligence** — analyze investment performance, MOIC, holdings, transaction history.

### Slash commands

- **/portfolio-report** — generate a portfolio overview with summary and per-company performance.

## When to install this vs. the full plugin

- Install **`ownersroom-portfolio`** if you only need investor-facing tools.
- Install **`ownersroom`** (kitchen sink) if you also need cap-table admin or fund-management tools.
- **Don't install both** — pick one. The kitchen sink already includes everything in this plugin.
