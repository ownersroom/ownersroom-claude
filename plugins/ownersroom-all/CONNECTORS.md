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
| `list_rooms` | optional `withCapabilities` (default `true`) | List accessible companies. When `withCapabilities` is true, each entry includes per-module status and supportedActions for planning. |
| `get_room_capabilities` | `roomId` | Per-module status (`Available` / `AccessDenied` / `UpgradeRequired` / `Empty`) + supportedActions for a single room |
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

### Write — User profile

| Tool | Parameters | Description |
|------|-----------|-------------|
| `update_user` | `profile` (UserInput) | Update the authenticated user's profile (name, email, country, address, about/description) |

### Write — Cap table

All cap-table writes require the matching capability flag on the room (see "Capability-aware planning" below).

| Tool | Parameters | Required action |
|------|-----------|-----------------|
| `update_share_class` | `roomId`, `id`, `params` (ShareClassInput) | `capTable.editSecurities` |
| `delete_share_class` | `roomId`, `id` | `capTable.editSecurities` (destructive) |
| `create_share_issuance` | `roomId`, `params` (ShareIssuanceInput) | `capTable.createShareIssuance` |
| `update_share_issuance` | `roomId`, `id`, `params` | `capTable.editCapTable` |
| `create_share_transaction` | `roomId`, `params` (ShareTransactionInput) | `capTable.editCapTable` |
| `update_share_transaction` | `roomId`, `id`, `params` | `capTable.editCapTable` |
| `delete_capital_event` | `roomId`, `capitalEventId` | `capTable.editCapTable` (destructive) |

### Write — Options

| Tool | Parameters | Required action |
|------|-----------|-----------------|
| `create_option_pool` | `roomId`, `params` (OptionTypeInput) | `capTable.manageOptions` |
| `update_option_pool` | `roomId`, `id`, `params` | `capTable.manageOptions` |
| `delete_option_pool` | `roomId`, `id` | `capTable.manageOptions` (destructive) |
| `create_option_grants` | `roomId`, `grants[]` (CreateOptionGrantInput) | `capTable.manageOptions` |
| `update_option_grant_event` | `roomId`, `capitalEventId`, `grants[]` | `capTable.manageOptions` |
| `cancel_option_grant` | `roomId`, `id`, optional `cancelAmount`, `validateUnvestedOnly`, `asOfDate` | `capTable.manageOptions` (destructive) |
| `exercise_option_grant` | `roomId`, `id`, `exerciseAmount`, optional `asOfDate` | `capTable.manageOptions` |

### Write — Portfolio (user-scoped, no room capability check)

Portfolio writes operate on the authenticated user's portfolio (not a room). They are not gated by room-level capabilities.

| Tool | Parameters | Description |
|------|-----------|-------------|
| `create_portfolio_cases` | `cases[]` (CaseInput) | Create one or more portfolio cases |
| `update_portfolio_case` | `caseId`, `case` (CaseInput) | Update a portfolio case |
| `delete_portfolio_case` | `caseId` | Delete a portfolio case (destructive) |
| `create_portfolio_asset` | `caseId`, `asset` (AssetInput), optional `owner` | Add an asset to a case |
| `update_portfolio_asset` | `caseId`, `assetId`, `asset`, optional `owner` | Update an asset |
| `delete_portfolio_asset` | `caseId`, `assetId` | Remove an asset from a case (destructive) |
| `update_estimated_asset_value` | `assetId` OR `holding` (`{actorId, shareClassId}`), `price`, optional `note` | Update the user's estimated value of an asset |

### Write — News posts

| Tool | Parameters | Required action |
|------|-----------|-----------------|
| `create_post` | `roomId`, `params` (PostInput) | `updates.createPost` |
| `update_post` | `roomId`, `postId`, `params` | `updates.createPost` |
| `publish_post` | `roomId`, `postId` | `updates.createPost` (visible to room members; may send email) |
| `delete_post` | `roomId`, `postId` | `updates.createPost` (destructive) |
| `preview_post_email` | `roomId`, optional `postId` or `params` | `updates.createPost` (sends preview email to the authenticated user only) |

File attachments are not supported in `params` (the GraphQL `Upload!` scalar can't go through MCP transport). Attach files via the OwnersRoom web UI.

### Typed write-tool params

Every write tool above declares an explicit JSON-Schema for its `params` field — no more opaque `params: object`. Clients with structured-output support can generate type-safe call sites; LLMs see the field shape directly without parsing description prose. Reusable value-object schemas (`MonetaryInput`, `ActorIdInput`, ledger-entry shapes per aggregate, `ShareClassInput`, `OptionTypeInput`, `PostInput`, etc.) appear in multiple tools and are consistent across them.

## Available Resources

The MCP server also exposes **23 read-only Resources** — passive context the LLM can browse without burning a tool call. Resource-aware clients (Claude Code, Claude Desktop, Gemini CLI) auto-discover these via `resources/list` and `resources/templates/list`. Resources don't replace tools; tools still take parameters and perform writes.

Each persona profile exposes a tailored subset; the kitchen-sink (`ownersroom-all`) shows all 23.

### Static enums

| URI | Profiles |
|-----|----------|
| `enums://currencies` | every profile |
| `enums://post-visibilities` | every profile |
| `enums://day-count-conventions` | captable, commitments, all |
| `enums://capitalization-frequencies` | captable, commitments, all |
| `enums://capital-event-kinds` | captable, commitments, all |

### Identity

| URI | Profiles |
|-----|----------|
| `me://` | every profile |
| `me://capabilities` | every profile |

`me://capabilities` returns the user plus every accessible room with its per-module capability matrix in one read — useful as a session-start scope check when the user's intent spans rooms or capability is uncertain. Per-room capability fetches that fail surface as `fetchError` on the entry rather than failing the whole response.

### Rooms (collection + per-room metadata)

| URI | Profiles |
|-----|----------|
| `rooms://` | every profile |
| `room://{id}` | every profile |
| `room://{id}/capabilities` | every profile |

### Per-room reference data

| URI | Profiles |
|-----|----------|
| `room://{id}/share-classes` | every profile |
| `room://{id}/shareholders` (first page) | captable + portfolio + all |
| `room://{id}/option-pools` | captable + all |
| `room://{id}/option-holders` (first page) | captable + all |
| `room://{id}/interest-rate-schedules` | captable + commitments + all |
| `room://{id}/fund` | commitments + all |
| `room://{id}/fund/commitment-holders` (first page) | commitments + all |

### Per-room templated reads

| URI | Profiles |
|-----|----------|
| `room://{id}/posts/{postId}` | every profile |
| `room://{id}/people/{actorId}/vesting` | portfolio + captable + all |

### Portfolio (user-scoped, cross-room)

| URI | Profiles |
|-----|----------|
| `portfolio://summary` | portfolio + all |
| `portfolio://cases` | portfolio + all |
| `portfolio://holdings` | portfolio + all |
| `portfolio://cases/{caseIdentifier}/vesting` | portfolio + all |

**Pagination on Resources**: `shareholders`, `option-holders`, and `commitment-holders` Resources return the first page only (default 100). Use the corresponding `list_*` tool when more pages are needed.

**Optional template parameters dropped**: the vesting Resources don't accept the `poolId` filter that the underlying tools accept. Use `get_vesting_history` or `get_portfolio_vesting` when filtering by pool.

## Capability-aware planning

Room-scoped tools check the user's per-room access *before* issuing the request. This lets you plan without bouncing off failures.

- `list_rooms` returns a `capabilities` matrix per room (set `withCapabilities: false` to skip the per-room fan-out for many-room users).
- `get_room_capabilities(roomId)` returns the same matrix for a single room — useful when you already know which room and just want to check what is allowed before a write.

A capability profile looks like:

```json
{
  "roomId": "...",
  "company": { "name": "Acme", "...": "..." },
  "modules": {
    "capTable":  { "status": "Available",    "actions": { "editCapTable": false, "manageOptions": false, "createShareIssuance": false, "viewSecurities": true,  "editSecurities": false, "exportCapTable": false } },
    "updates":   { "status": "Available",    "actions": { "listPosts": true, "createPost": false, "editCategories": false, "viewGroups": false, "listDrafts": false } },
    "contacts":  { "status": "AccessDenied" },
    "documents": { "status": "Available",    "actions": { "viewSigningRequests": true, "viewDownloadAnalytics": false } },
    "dealsModule": { "status": "Hidden" }
  }
}
```

Rules:

- **Module `status` is not `Available`** → tools in that module will fail. Don't call them.
- **`actions.<flag>` is `false`** → write tools that require that flag will fail. Reads typically still work as long as the module is Available.
- **Actions are omitted** when the module is not Available — they would all be false and just add noise.

## Structured errors

When a tool can't produce data (permission denied, room not found, validation error), it returns a structured envelope in place of the data. The envelope is normal tool output (not a transport error) — read it as data and react.

```json
{
  "error": "action_not_allowed",
  "hint": "The capTable.editCapTable permission is required but is not granted in this room (module status: Available). Use list_rooms or get_room_capabilities to find a room where this action is allowed.",
  "tool": "update_share_issuance",
  "roomId": "...",
  "module": "capTable",
  "action": "editCapTable",
  "moduleStatus": "Available"
}
```

Codes you can switch on:

| Code | Meaning |
|------|---------|
| `room_not_found` | The `roomId` does not exist or the user has no access to it. |
| `module_not_available` | The room exists, but the module's content status is not `Available` (`AccessDenied` / `UpgradeRequired` / `Empty` / `Hidden`). |
| `action_not_allowed` | The module is available, but the specific `supportedActions.<flag>` required for this tool is `false`. The `module`, `action`, and `moduleStatus` fields tell you exactly what's missing. |
| `not_found` | An entity inside an accessible scope (a post, a portfolio case, an option grant) doesn't exist. |
| `validation_error` | Input failed schema validation (`BAD_USER_INPUT`). |
| `unauthenticated` | The OAuth token is missing or rejected. The user needs to re-authenticate. |
| `internal` | Anything else. |

When you get one of these, prefer recovery moves (try a different room, ask the user to confirm a value, ask for a different action) over blind retries.
