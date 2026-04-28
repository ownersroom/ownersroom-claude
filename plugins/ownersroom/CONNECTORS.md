# Connectors

This plugin connects to the OwnersRoom MCP server over HTTP. Authentication is handled via OAuth — Claude Code performs the login flow automatically on first use.

## MCP Server

The MCP server is bundled with the plugin. When installed, the `ortool` MCP server is automatically available — no manual `claude mcp add` needed.

The server supports Dynamic Client Registration, so OAuth configuration is discovered automatically.

## Available Tools

### Read

| Tool | Parameters | Description |
|------|-----------|-------------|
| `get_current_user` | — | Current authenticated user profile |
| `list_rooms` | — | List accessible companies (returns room IDs) |
| `list_share_classes` | `roomId` | Share classes with nominal value, total shares, share capital |
| `list_shareholders` | `roomId` | Shareholders with per-class holdings |
| `list_option_pools` | `roomId` | Option pools with size and grant counts |
| `list_option_grants` | `roomId`, optional `poolId`, `actorId` | Individual option grants |
| `list_option_holders` | `roomId` | Aggregated option holder data |
| `get_vesting_history` | `roomId`, `actorId`, optional `poolId` | Vesting timeline with data points |
| `get_portfolio_summary` | — | Portfolio invested, realized, unrealized, MOIC |
| `list_portfolio_cases` | — | Per-company portfolio breakdown |
| `list_portfolio_holdings` | — | Share holdings across companies |
| `list_portfolio_events` | `roomId`, optional `after` | Paginated transaction history |
| `get_portfolio_vesting` | `caseIdentifier`, optional `poolId` | Option vesting for a portfolio case |
| `list_posts` | `roomId`, optional `after`, `categories` | News post summaries (paginated) |
| `get_post` | `roomId`, `postId` | Single news post with full body |

### Write — News posts

| Tool | Parameters | Description |
|------|-----------|-------------|
| `create_post` | `roomId`, `params` (PostInput) | Create a draft news post |
| `update_post` | `roomId`, `postId`, `params` | Update an existing post |
| `publish_post` | `roomId`, `postId` | Publish a draft (visible to room members; may send email) |
| `delete_post` | `roomId`, `postId` | Delete a post (destructive) |
| `preview_post_email` | `roomId`, optional `postId` or `params` | Send a preview email to the authenticated user |

File attachments are not supported in `params` (the GraphQL `Upload!` scalar can't go through MCP transport). Attach files via the OwnersRoom web UI.
