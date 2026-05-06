# OwnersRoom (everything) for Claude Code

> **Experimental.** Internal prototype exploring AI-assisted workflows for equity data. Not a supported OwnersRoom product. Not intended for end users of OwnersRoom.

The full OwnersRoom surface in one install: cap table, options, employee equity, portfolio, fund management, and news. **56 MCP tools + 23 read-only Resources, all skills, all slash commands.**

## Install

```
/plugin marketplace add ownersroom/ownersroom-claude
/plugin install ownersroom-all@ownersroom
```

## Renamed in 2.0.0

This plugin was previously `ownersroom@ownersroom`. It was renamed to `ownersroom-all` for consistency with the focused alternatives (`ownersroom-portfolio`, `ownersroom-captable`, `ownersroom-commitments`).

If you previously installed `ownersroom@ownersroom`, uninstall it and re-install as `ownersroom-all@ownersroom`. You will need to complete the OAuth flow again — the MCP server name also changed (`ortool` → `ortool-all`) and tokens don't carry over.

## What you get

56 MCP tools across:

- **Identity & rooms** — current user, list rooms, per-room capabilities, update profile.
- **Cap table** — share classes, shareholders, share issuances, transactions, capital events.
- **Options** — option pools, grants, holders, vesting; create / update / delete pools and grants; cancel and exercise.
- **Employee loans** — list loans and repayments, simulate interest, create / update issuances and repayments.
- **Interest rate schedules** — list / create / update / delete schedules used for loan accrual and fund hurdles.
- **Capital commitments (fund)** — get fund, list LP commitment holders, create commitment closes, transfers, calls, distributions, equalizations.
- **Portfolio** — summary, per-company breakdown, holdings, transaction history, vesting per case; create / update / delete portfolio cases and assets; update estimated values.
- **News** — read, draft, edit, preview, publish, delete posts.

Writes that touch a room respect your per-room permissions automatically. Every write tool exposes a typed JSON-Schema for its parameters.

### MCP Resources

The server also exposes **23 read-only Resources** for passive context — enums (currencies, capital-event kinds, …), identity (`me://`, `me://capabilities` for one-shot user + per-room capability matrix), rooms (`rooms://`, `room://{id}`, `room://{id}/capabilities`), per-room reference data (share classes, shareholders, option pools, …), templated reads (`room://{id}/posts/{postId}`, `room://{id}/people/{actorId}/vesting`), and portfolio (`portfolio://summary`, `portfolio://holdings`, …).

See [CONNECTORS.md](CONNECTORS.md) for the full tool and Resource reference, including the structured-error envelope and the capability-planning model.

### Skills

- **captable-analysis** — ownership, share classes, dilution.
- **options-expertise** — option pools, vesting, strike prices, exercise.
- **portfolio-intelligence** — investment performance, MOIC, holdings, transactions.
- **news-updates** — drafting, previewing, publishing posts.

### Slash commands

- **/ownership-report** — formatted ownership report.
- **/portfolio-report** — portfolio overview with summary and per-company performance.

## Smaller install options

If you only need part of the surface, install one of the focused plugins instead:

| Plugin | Tools | For |
|---|---:|---|
| **ownersroom-portfolio** | ~13 | Investor / shareholder — own holdings, vesting, news |
| **ownersroom-captable** | ~38 | Founder / CFO / cap-table admin — shares, options, employee equity, news |
| **ownersroom-commitments** | ~22 | PE / VC GP — LP commitments, capital ops, LP letters |
| **ownersroom-all** *(this plugin)* | 56 | Everything |

**Don't install both this plugin and a focused one** — pick one. The focused plugins are subsets of this one.
