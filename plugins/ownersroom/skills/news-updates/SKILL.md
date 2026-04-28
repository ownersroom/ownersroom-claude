---
name: news-updates
description: Read, draft, edit, and publish news posts (room updates) on OwnersRoom — list posts, view full post bodies, create drafts, send preview emails, and publish updates to room members. Use when the user asks about posts, news, room updates, investor letters, or wants to draft, send, or read company communications.
---

# News & Post Updates

OwnersRoom rooms have a news feed for company updates — investor letters, quarterly summaries, announcements. This skill covers reading existing posts and drafting / publishing new ones.

## Fetching Data

| Tool | Parameters | Returns |
|------|-----------|---------|
| `list_posts` | `roomId`, optional `after`, `categories` | Paginated post summaries: `id`, `title`, `status`, `visibility`, dates, `author`, `categories` |
| `get_post` | `roomId`, `postId` | Single post with full `body`, `contents` (rich-text JSON), all metadata |

`list_posts` returns lightweight summaries; call `get_post` when the user wants the full body.

## Post Lifecycle

```
Draft → Published → (optionally Updated)
```

1. **Create** with `create_post`. New posts start as `Draft`. They are not visible to room members until published.
2. **Edit** with `update_post`. Drafts can be updated freely; published posts get marked `Updated` after the change.
3. **Preview** with `preview_post_email`. Sends a real preview email to the authenticated user (not to room members). Use to sanity-check formatting before publishing.
4. **Publish** with `publish_post`. The post becomes visible to room members. If `shouldSendEmail: true` was set, members are emailed.
5. **Delete** with `delete_post`. Destructive, removes the post regardless of status.

## PostInput Shape

`create_post` and `update_post` both accept a `params` object matching `PostInput`:

```json
{
  "title": "Q1 2026 update",            // required
  "body": "Plain-text body",            // optional
  "contents": { ... },                  // optional rich-text JSON document
  "categories": ["cat-id-1"],           // required (can be empty array)
  "visibility": "Internal",             // required: Public | Internal | Restricted
  "groups": ["group-id-1"],             // only meaningful when visibility = Restricted
  "shouldSendEmail": true,              // required
  "shouldAttachFiles": false            // required
}
```

### Visibility

- `Public` — anyone with the room link can read.
- `Internal` — only authenticated room members.
- `Restricted` — only members of specific `groups` (must be supplied).

### Email behavior

- `shouldSendEmail: true` plus `publish_post` → all eligible recipients receive an email.
- `shouldSendEmail: true` plus `preview_post_email` → only the authenticated user receives a preview (safe to use freely).
- `shouldSendEmail: false` → publishing makes the post visible in the news feed only; no email goes out.

### Attachments

`PostInput.attachments` is **not exposed** through this plugin (the `Upload!` scalar is incompatible with MCP transport). To attach files, use the OwnersRoom web UI; the LLM cannot upload binary files.

## Safety Guidance

These operations have external, hard-to-undo effects. Confirm with the user before calling:

- **`publish_post`** — the post becomes visible to its target audience and (if `shouldSendEmail: true`) emails go out. There is no undo for emails. Always read the draft back to the user with `get_post` and confirm before publishing.
- **`preview_post_email`** — sends a real email to the authenticated user. Lower stakes than `publish_post` (only the user receives it), but it's still a real email.
- **`delete_post`** — destructive. Removes the post; cannot be recovered.

## Common Workflows

### Drafting an investor update

1. Ask the user for the message they want to send (or generate from context).
2. Pick `categories` and `visibility`. If unsure, default to `Internal` and an empty `categories` array.
3. Decide whether to email recipients. Ask the user if not specified.
4. Call `create_post` with `shouldSendEmail` set per the user's choice.
5. Read the draft back from the response and confirm the title / body with the user.
6. Optionally `preview_post_email` so the user sees what recipients will see.
7. Call `publish_post` only after the user explicitly approves.

### Reviewing recent posts

1. `list_posts(roomId)` for the room.
2. Show titles, dates, status, and authors as a table.
3. Offer to drill into a specific post via `get_post`.

### Editing a published post

1. `get_post` to fetch current state.
2. Show the user what will change.
3. `update_post` with the new `params`. Note: most `PostInput` fields are required, so the params object should contain the full post — derive missing fields from the current `get_post` response.

## Field Mapping Cheat Sheet

| Input field        | Required? | Notes |
|--------------------|-----------|-------|
| `title`            | yes       | Post title |
| `body`             | no        | Plain-text fallback for clients that can't render rich text |
| `contents`         | no        | Rich-text JSON document (e.g., a Tiptap / ProseMirror doc) |
| `categories`       | yes       | Array of category IDs; empty array `[]` is valid |
| `visibility`       | yes       | `Public` / `Internal` / `Restricted` |
| `groups`           | only if Restricted | Array of group IDs |
| `shouldSendEmail`  | yes       | If `true`, publishing emails recipients |
| `shouldAttachFiles`| yes       | Must be `false` from this plugin (attachments not supported) |
