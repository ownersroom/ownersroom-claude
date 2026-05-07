# Connectors

This plugin connects to the OwnersRoom MCP server over HTTP. Authentication is handled via OAuth — Claude Code performs the login flow automatically on first use.

## MCP Server

The MCP server is bundled with the plugin. When installed, the `ortool` MCP server is automatically available — no manual `claude mcp add` needed.

The server supports Dynamic Client Registration, so OAuth configuration is discovered automatically.

### Reading Resources via the MCP client

Claude Code's `ReadMcpResourceTool` expects the canonical, colon-namespaced server name — for this plugin, `plugin:ownersroom-all:ortool-all` (persona plugins: `plugin:ownersroom-portfolio:ortool-portfolio`, `plugin:ownersroom-captable:ortool-captable`, `plugin:ownersroom-commitments:ortool-commitments`).

The underscore-form prefix that appears in MCP tool names (e.g., `mcp__plugin_ownersroom-all_ortool-all__list_shareholders`) is **not** accepted by the resource API. Use `ListMcpResourcesTool` to discover the correct server name; each entry's `server` field is the form to pass back into `ReadMcpResourceTool`. This is a known Claude Code wart ([anthropics/claude-code#29360](https://github.com/anthropics/claude-code/issues/29360)).

## Available Tools

### Read

Read-only tools cover anything that requires filtering or pagination. Reads with no arguments (or just a `roomId`) are exposed as Resources instead — see "Available Resources" below.

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_shareholders` | `roomId`, optional `after` | Shareholders with per-class holdings (paginated) |
| `list_share_events` | `roomId`, optional `actorIds`, `after` | Share events (issuance, transaction, split, cancellation, import) with full per-event ledger |
| `list_option_grants` | `roomId`, optional `poolId`, `actorId` | Individual option grants with optional filtering |
| `list_option_holders` | `roomId`, optional `after` | Aggregated option holder data (paginated) |
| `list_option_events` | `roomId`, optional `actorIds`, `after` | Option events (grant created, cancelled, exercised) with full per-event ledger |
| `get_vesting_history` | `roomId`, `actorId`, optional `poolId` | Vesting timeline with data points |
| `list_employee_loans` | `roomId`, optional `borrowerId`, `lenderId`, `currency`, `asOfDate`, `after` | Paginated employee loans with live interest snapshot |
| `list_loan_repayments` | `roomId`, `loanId`, optional `after` | Paginated repayments for a loan |
| `list_employee_loan_events` | `roomId`, optional `actorIds`, `after` | Employee-loan events (issuance, repayment) — room-wide flat view |
| `simulate_loan_interest` | `roomId`, `principal`, `startDate`, `endDate`, optional schedule, repayments | Compute interest accrual on a hypothetical loan |
| `list_commitment_holders` | `roomId`, optional `after` | Per-actor commitment breakdown (paginated) |
| `list_commitment_events` | `roomId`, optional `actorIds`, `after` | Commitment + cash-flow events (close, transfer, capital call, distribution, equalization) with full per-event ledger |
| `list_people` | `roomId`, optional `after` | People in a room's contacts module (paginated) |
| `list_companies` | `roomId`, optional `after` | Companies in a room's contacts module (paginated) |
| `search_organization` | `searchText`, `country` | Free-text search against an external registry (e.g., Brønnøysund). Returns array, non-paginated. |
| `lookup_organization` | `orgNumber`, `country` | Exact-orgNumber lookup against an external registry. Returns single result with `alreadyExists` flag. |
| `list_portfolio_events` | `roomId`, optional `after` | Paginated transaction history |
| `get_portfolio_vesting` | `caseIdentifier`, optional `poolId` | Option vesting for a portfolio case |
| `list_posts` | `roomId`, optional `after`, `categories` | News post summaries (paginated) |

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

### Write — Contacts (people, companies)

Contacts writes are gated only by module-level `contacts.Status == Available` (the `ContactsActions` struct exposes only `canMergeContacts`, no per-write actions; backend enforces finer-grained role-based access and surfaces denials as structured errors).

| Tool | Parameters | Description |
|------|-----------|-------------|
| `create_person` | `roomId`, `params` (PersonActorInput) | Create a Person actor. Required: firstName, lastName, country. PII fields (`governmentIdentifier`, `bankAccount`) are opt-in — only included when the user volunteers them. |
| `update_person` | `roomId`, `params` (PersonActorInput, params.id required) | **Update semantics:** full-replace. |
| `create_company` | `roomId`, `params` (OrganizationActorInput) | Create a Company actor. Required: orgNumber, name, country. |
| `update_company` | `roomId`, `params` (OrganizationActorInput, params.id required) | **Update semantics:** full-replace. |

### Write — Deals (primary share offerings)

Deal writes are gated by module-level `dealsModule.Status == Available`. The per-deal `DealActions` struct (`openDeal`, `closeDeal`, `cancelDeal`, `editParticipants`, `importToCapTable`, etc.) is per-deal not module-level — surfaced on Deal reads so the LLM can preflight, but actual gating happens backend-side.

| Tool | Parameters | Description |
|------|-----------|-------------|
| `create_deal` | `roomId` | Create a new (empty) Deal in CREATED status. |
| `open_deal` | `roomId`, `dealId`, optional `doNotSendOffers` | **Lifecycle:** transitions to OPENED. **Side effect:** sends offer emails to participants unless `doNotSendOffers=true`. |
| `close_deal` | `roomId`, `dealId` | **Lifecycle:** transitions OPENED to CLOSED. |
| `cancel_deal` | `roomId`, `dealId` | **Lifecycle:** withdraws an in-flight deal (destructive). |
| `update_deal_info` | `roomId`, `dealId`, `params` (DealInfoInput) | **Update semantics:** full-replace. |
| `create_deal_participant` | `roomId`, `dealId`, `params` (DealParticipantInput), optional `doNotSendOffer` | Add a participant. params.signingPersonId required. |
| `update_deal_participant` | `roomId`, `dealId`, `dealParticipantId`, `params` | **Update semantics:** full-replace. |
| `delete_deal_participant` | `roomId`, `dealId`, `dealParticipantId` | Remove a participant (destructive). |
| `create_primary_share_offer` | `roomId`, `dealId`, `params` (PrimaryShareOfferInput) | Create an offer. RangeInput sub-types for sharesOnOffer / pricePerShare / capitalToRaise / preMoneyValuation. |
| `update_primary_share_offer` | `roomId`, `dealId`, `primaryShareOfferId`, `params` | **Update semantics:** full-replace. |
| `delete_offer` | `roomId`, `dealId`, `offerId` | Remove an offer (destructive). |
| `update_primary_share_subscription_allocations` | `roomId`, `offerId`, `allocations[]` (IntegerAllocationInput) | Set per-subscription share allocations. |
| `update_primary_share_subscription_payments` | `roomId`, `offerId`, `payments[]` (SubscriptionPaymentInput) | Record per-subscription payments received. |
| `add_deal_to_cap_table` | `roomId`, `dealId`, `eventName`, `registrationDate`, optional `useGuidingSharePrice` | **Side effect:** creates a share-issuance capital event from the deal's allocations. **Not reversible** — verify the deal is CLOSED with finalized allocations. |

### Write — Document e-signature

Operates on documents already uploaded to the room (file uploads are out of scope — the GraphQL `Upload!` scalar can't go through MCP transport). Writes don't use `requireAction`; per-request `supportedActions.prolongDeadline` is enforced backend-side. Signatories carry a full `PersonActorInput` identity object — the backend dedupes by email and government identifier, so the same person can be supplied across requests without first creating them via `create_person`.

| Tool | Parameters | Description |
|------|-----------|-------------|
| `create_signing_request` | `roomId`, `path`, optional `title` / `invitationBody` / `deadline`, `signatories[]` (SignatoryInput) | Initiate a signing request against an existing document at `path`. **Side effect:** real signatory emails. |
| `cancel_signing_request` | `roomId`, `signingRequestId` | **Lifecycle:** cancel an in-flight request (destructive). |
| `prolong_signing_request` | `roomId`, `signingRequestId`, `days` | Extend the deadline. Only valid when `supportedActions.prolongDeadline` is true. |
| `remind_signatories` | `roomId`, exactly one of `signingRequestId` or `signingRequestIds[]` | **Side effect:** sends reminder emails to outstanding signatories. |
| `update_signatory` | `roomId`, `signingRequestId`, `params` (SignatoryInput) | **Update semantics:** full-replace a signatory entry. |

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

The MCP server also exposes **28 read-only Resources** — passive context the LLM can browse without burning a tool call. Resource-aware clients (Claude Code, Claude Desktop, Gemini CLI) auto-discover these via `resources/list` and `resources/templates/list`. Resources don't replace tools; tools still take parameters and perform writes.

Each persona profile exposes a tailored subset; the kitchen-sink (`ownersroom-all`) shows all 28.

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
| `room://{id}/people` (first page) | captable + all |
| `room://{id}/companies` (first page) | captable + all |
| `room://{id}/deals` | captable + all |
| `room://{id}/deals/{dealId}` | captable + all |
| `room://{id}/signing-requests` (first 50) | captable + commitments + all |
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

**Pagination on Resources**: `shareholders`, `option-holders`, `commitment-holders`, `people`, and `companies` Resources return the first page only (default 100). Use the corresponding `list_*` tool when more pages are needed.

**Optional template parameters dropped**: the vesting Resources don't accept the `poolId` filter that the underlying tools accept. Use `get_vesting_history` or `get_portfolio_vesting` when filtering by pool.

## Available Prompts

MCP Prompts are reusable saga templates — orchestrated sequences of tool calls and Resource reads that the LLM would otherwise have to assemble from prose hints. Clients (Claude Code, Claude Desktop) advertise them via `prompts/list` and let the user invoke them by name with optional arguments. Saga prose lives in markdown templates in the server source so reviewers can read and diff the saga shape directly.

| Prompt | Arguments | Persona |
|--------|-----------|---------|
| `financing_round` | `roomId` (required); optional `dealName`, `targetRoundSize` | captable + all |
| `capital_call_cycle` | `roomId` (required); optional `callPercentage`, `dueDate` | commitments + all |
| `quarterly_lp_letter` | `roomId` (required); optional `quarter` | commitments + all |

`financing_round` walks through a primary financing round end-to-end: discover IDs via `room://{id}/share-classes` and `room://{id}/people` → `create_deal` → `create_primary_share_offer` → `create_deal_participant` (×N) → **stop & confirm** → `open_deal` (sends real offer emails) → `close_deal` → `update_primary_share_subscription_allocations` + `_payments` → **stop & confirm** → `add_deal_to_cap_table` (irreversible share issuance) → `create_post` + `publish_post`. Verification of state via `room://{id}/deals/{dealId}` after each transition; recovery guidance for partial failures.

`capital_call_cycle` runs a single capital-call cycle: discover via `room://{id}/fund` and `room://{id}/fund/commitment-holders` → **stop & confirm** → `create_capital_call` (real financial event) → optional `create_capital_equalization` based on a close-history heuristic → LP notification with `create_post` → optional `preview_post_email` → **stop & confirm** → `publish_post`.

`quarterly_lp_letter` drafts a quarterly LP letter: read `room://{id}/fund`, `room://{id}/fund/commitment-holders`, recent `list_commitment_events` for the quarter window → caller-provided portfolio updates (the saga deliberately doesn't auto-discover, since `list_portfolio_cases` is portfolio-persona only) → `create_post` with a structured body (Q summary / capital activity / portfolio updates / outlook) → `preview_post_email` → **stop & confirm** → `publish_post`.

**Prompts vs slash commands**: MCP Prompts are server-side saga templates advertised to any MCP client; plugin slash commands (`/ownership-report`, `/portfolio-report`) are client-side report rendering that only Claude Code surfaces. The two coexist deliberately — sagas with multi-tool write flows live in Prompts; presentation-focused reports live in slash commands.

## Capability-aware planning

Room-scoped tools check the user's per-room access *before* issuing the request. This lets you plan without bouncing off failures.

- `me://capabilities` returns identity plus every accessible room with its capability matrix in one read — the bootstrap call for cross-room planning.
- `room://{id}/capabilities` returns the matrix for a single room — useful when you already know which room and just want to check what is allowed before a write.
- `rooms://` returns the bare list of accessible rooms without capability fan-out, when capability data isn't needed.

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
  "hint": "The capTable.editCapTable permission is required but is not granted in this room (module status: Available). Read me://capabilities or room://{id}/capabilities to find a room where this action is allowed.",
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
