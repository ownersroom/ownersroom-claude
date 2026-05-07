# OwnersRoom for Claude Code

> **Experimental.** This is an internal prototype exploring AI-assisted workflows for equity data. It is not a supported OwnersRoom product and may change or break without notice. Not intended for end users of OwnersRoom.

Access OwnersRoom cap table, options, portfolio, and fund management data from Claude Code.

## Plugins

The marketplace ships four plugins. **Pick one** based on what you do — they all overlap in tool coverage, and installing two at once doubles the LLM's tool surface unnecessarily.

| Plugin | Tools | Best for |
|---|---:|---|
| **`ownersroom-all`** | 83 | Default. Full surface — pick this if you want everything. |
| **`ownersroom-portfolio`** | ~13 | Investor / shareholder — track your portfolio, holdings, vesting, news. |
| **`ownersroom-captable`** | ~65 | Founder / CFO / cap-table admin — shares, options, employee equity, contacts, deals, e-signature, news. |
| **`ownersroom-commitments`** | ~27 | PE / VC GP — LP commitments, capital ops, e-signature, LP letters. |

### Decision tree

- **Reading your own holdings or news?** → `ownersroom-portfolio`.
- **Running a company's cap table?** → `ownersroom-captable`.
- **Running a fund?** → `ownersroom-commitments`.
- **Crossing roles, or unsure?** → `ownersroom-all`.

### Boundary clarifications

- `ownersroom-captable` covers **company-internal** capital structure (shares, options, employee equity). LP commitments live in `ownersroom-commitments`.
- `ownersroom-commitments` is **GP-side** fund operations. LP investors viewing *their* fund holdings should install `ownersroom-portfolio` (it lists holdings cross-room).

## Install

### Claude Code (CLI)

In Claude Code, run:

```
# Add the OwnersRoom marketplace
/plugin marketplace add ownersroom/ownersroom-claude

# Install the plugin you want (one of)
/plugin install ownersroom-all@ownersroom
/plugin install ownersroom-portfolio@ownersroom
/plugin install ownersroom-captable@ownersroom
/plugin install ownersroom-commitments@ownersroom
```

You can also browse and install via the interactive `/plugin` menu under the **Discover** tab.

### Cowork (Desktop App)

1. Open the Claude Desktop app and switch to the **Cowork** tab.
2. Click **Customize** in the left sidebar.
3. Click **Browse plugins** to open the plugin browser.
4. Find the OwnersRoom plugin you want and click **Install**.

If the plugin isn't listed in the browser, you can upload it as a custom plugin file instead.

---

Claude Code will prompt you to log in with your OwnersRoom account on first use. Per-plugin tokens are independent — installing a different plugin requires re-authenticating.

## Renamed in 2.0.0

The previously single `ownersroom@ownersroom` plugin was renamed to `ownersroom-all@ownersroom` to fit the persona-prefix family. If you previously installed `ownersroom@ownersroom`:

```
/plugin uninstall ownersroom@ownersroom
/plugin install ownersroom-all@ownersroom
```

You will need to re-complete the OAuth flow — the underlying MCP server name also changed (`ortool` → `ortool-all`) and tokens don't carry over.

## What you get

All plugins share the same building blocks:

### MCP tools

The plugin connects to the OwnersRoom API and gives Claude access to your data — both reads and writes. Each plugin includes a focused subset of the full surface; see the plugin's own README for the tools it ships.

Every write tool exposes a typed JSON-Schema for its parameters — clients with structured-output support can generate type-safe call sites instead of guessing field shapes from prose.

Writes that touch a room respect your per-room permissions automatically — Claude is told exactly which permission is missing if a write isn't allowed, so you can request access (or pick a different room) instead of seeing a cryptic failure.

See [CONNECTORS.md](plugins/ownersroom-all/CONNECTORS.md) for the full tool reference, including the structured-error envelope and the capability-planning model.

### MCP Resources

Beyond the tools, the server exposes **28 read-only Resources** — passive context Claude can browse without burning a tool call. Examples: `me://` (your profile), `me://capabilities` (your profile + every accessible room with its capability matrix, in one read), `enums://capital-event-kinds` (valid event types), `room://{id}/share-classes` (a room's share classes), `room://{id}/people` (people in a room), `room://{id}/deals/{dealId}` (a single deal with offers and participants), `room://{id}/signing-requests` (in-flight signing requests), `portfolio://summary` (your portfolio summary), `room://{id}/posts/{postId}` (a single news post). Each persona profile exposes a tailored subset; the kitchen-sink (`ownersroom-all`) shows all 28.

Resources work alongside tools — they're for read-only context, not writes. See [CONNECTORS.md](plugins/ownersroom-all/CONNECTORS.md) for the full Resource catalog.

### MCP Prompts

The server also exposes **saga Prompts** — reusable templates that wrap multi-step workflows the LLM would otherwise have to assemble from prose hints. Clients advertise them via `prompts/list` and let you invoke them by name with optional arguments.

| Prompt | What it does | In plugins |
|--------|-------------|-----------|
| `financing_round` | Walks through a primary financing round end-to-end: deal creation → terms → participants → open (real emails) → close → finalize allocations & payments → bridge to cap table (irreversible) → announcement post. Stops to confirm before each side-effecting write. | `-all`, `-captable` |

### Skills

Skills activate automatically when Claude detects a relevant question — just ask naturally.

| Skill | Triggers on | In plugins |
|-------|------------|-----------|
| **captable-analysis** | Ownership, shareholders, share classes, dilution, cap table | `-all`, `-captable` |
| **options-expertise** | Options, vesting, grants, strike price, exercise cost | `-all`, `-captable` |
| **portfolio-intelligence** | Portfolio, investments, returns, MOIC, holdings | `-all`, `-portfolio` |
| **news-updates** | News posts, room updates, investor letters, drafting / publishing announcements | `-all`, `-captable`, `-commitments` |

### Slash commands

| Command | Description | In plugins |
|---------|-------------|-----------|
| `/ownership-report` | Shareholder ownership report for a company | `-all`, `-captable` |
| `/portfolio-report` | Portfolio overview with performance metrics | `-all`, `-portfolio` |

## Examples

```
> What companies do I have access to?
> Show me the ownership breakdown for Acme Corp
> What's my portfolio performance?
> /portfolio-report
> /ownership-report
> Show me the vesting schedule for my options in Acme Corp
> List the most recent news posts in Acme Corp
> Draft a Q1 investor update for Acme Corp and let me preview it before publishing
> Add a 200-share issuance to Bob in Acme Corp at 100 NOK per share, dated today
> Update my estimated value for the Acme Corp shares to 250 NOK
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) v1.0.33 or later
- An OwnersRoom account

## License

Apache 2.0 — see [LICENSE](LICENSE).
