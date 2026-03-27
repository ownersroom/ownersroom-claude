# OwnersRoom for Claude Code

> **Experimental.** This is an internal prototype exploring AI-assisted workflows for equity data. It is not a supported OwnersRoom product and may change or break without notice. Not intended for end users of OwnersRoom.

Access OwnersRoom cap table, options, and portfolio data from Claude Code.

## Install

### Claude Code (CLI)

In Claude Code, run:

```
# Add the OwnersRoom marketplace
/plugin marketplace add ownersroom/ownersroom-claude

# Install the plugin
/plugin install ownersroom@ownersroom
```

You can also browse and install via the interactive `/plugin` menu under the **Discover** tab.

### Cowork (Desktop App)

1. Open the Claude Desktop app and switch to the **Cowork** tab.
2. Click **Customize** in the left sidebar.
3. Click **Browse plugins** to open the plugin browser.
4. Find the OwnersRoom plugin and click **Install**.

If the plugin isn't listed in the browser, you can upload it as a custom plugin file instead.

---

Claude Code will prompt you to log in with your OwnersRoom account on first use.

## What You Get

### MCP Tools

The plugin connects to the OwnersRoom API and gives Claude access to your data:

- **Companies** — list rooms you have access to
- **Cap table** — share classes, shareholders, ownership percentages
- **Options** — pools, grants, holders, vesting schedules
- **Portfolio** — summary, per-company performance, holdings, transaction history

See [CONNECTORS.md](plugins/ownersroom/CONNECTORS.md) for the full tool reference.

### Skills

Skills activate automatically when Claude detects a relevant question — just ask naturally.

| Skill | Triggers on |
|-------|------------|
| **captable-analysis** | Ownership, shareholders, share classes, dilution, cap table |
| **options-expertise** | Options, vesting, grants, strike price, exercise cost |
| **portfolio-intelligence** | Portfolio, investments, returns, MOIC, holdings |

### Commands

| Command | Description |
|---------|-------------|
| `/ownersroom:ownership-report` | Shareholder ownership report for a company |
| `/ownersroom:portfolio-report` | Portfolio overview with performance metrics |

## Examples

```
> What companies do I have access to?
> Show me the ownership breakdown for Acme Corp
> What's my portfolio performance?
> /ownersroom:ownership-report
> /ownersroom:portfolio-report
> Show me the vesting schedule for my options in Acme Corp
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) v1.0.33 or later
- An OwnersRoom account

## License

Apache 2.0 — see [LICENSE](LICENSE).
