# narrador MCP server

[![smithery badge](https://smithery.ai/badge/nicolaseliasarias/narrador)](https://smithery.ai/servers/nicolaseliasarias/narrador)

narrador turns documents and X (Twitter) posts into natural narrated audio with word-level timestamps. Upload PDFs, Word, EPUBs or text, pick a voice (or let narrador files choose one from the document language), start conversion jobs, check status and download audio with synced timings. narrador X turns posts into audio the same way. Sign in with OAuth at narrador.io, or use a product API key.

Remote [Model Context Protocol](https://modelcontextprotocol.io) server for **narrador**. It lets AI agents (Claude, Cursor, VS Code, Windsurf, ChatGPT and any MCP client) use narrador files and narrador X.

- **narrador files**: PDF, Word, EPUB and other documents → audio with word-level timings.
- **narrador X**: X/Twitter posts → audio.

| | |
|---|---|
| Endpoint | `https://api.narrador.io/mcp` (Streamable HTTP) |
| Auth | OAuth 2.1 (recommended) or API key |
| REST API docs | https://api.narrador.io/docs |
| OpenAPI | https://api.narrador.io/openapi.json |
| Website | https://narrador.io |

Repository: https://github.com/narrador-io/narrador-mcp

This repository only holds the public listing (`server.json`), registry metadata (`glama.json`) and documentation. The server is hosted by narrador; there is nothing to install locally.

## Credits

Usage is metered in **narrador credits** (see `get_account`). Jobs and generations consume credits when they run. There are no subscriptions—buy or earn credits in the narrador web app and check balance before calling tools that spend credits.

## Tools (7)

| Tool | Product | What it does |
|---|---|---|
| `get_account` | both | Identity, product scope, and narrador credit balances (read-only) |
| `list_voices` | both | Voices for narrador files or narrador X |
| `create_files_job` | narrador files | Start a document → audio job |
| `get_files_job` | narrador files | Poll job status and progress |
| `get_files_result` | narrador files | Download audio and word timings for a completed job |
| `generate_x_audio` | narrador X | Generate audio for an X post |
| `get_x_generation` | narrador X | Fetch a generation you created earlier |

### `get_account`

No parameters. Returns `user`, `app` (API key product or `null` for OAuth), `products` (`files` and/or `x`), and `credits` (`totalAvailable` and per-bucket `balances`). Does not consume credits.

### `list_voices`

| Parameter | Required | Description |
|---|---|---|
| `product` | No | `files` or `x`—which catalog to list. Defaults to the API key’s product, or `files` when an OAuth token has both products. |

Each voice includes `id`, `name`, `tier` (`standard` or `cheaper`), `defaultAccent`, `isPublic`, `description`, and `sampleUrl` (`https://api.narrador.io/v1/voices/{id}/sample`, a neutral sample redirect that does not require an API key). A legacy `provider` field may mirror `tier` and is deprecated—use `tier`.

### `create_files_job`

Provide **either** `text` **or** `fileId`, not both.

| Parameter | Required | Description |
|---|---|---|
| `text` | One of text/fileId | Plain text (1–500,000 characters). `fileName` optional (defaults to `document.txt`). |
| `fileId` | One of text/fileId | UUID of an uploaded file. `fileName` required. |
| `fileName` | If `fileId` | Job file name (1–200 characters). |
| `voiceId` | No | Voice UUID from `list_voices`. Must be set together with `accent`, or omit both. |
| `accent` | No | Accent code (e.g. `en-us`, `es-es`, `pt-br`). Must be set together with `voiceId`, or omit both. |

When `voiceId` and `accent` are omitted, narrador detects language from the document (`en-us`, `es-es`, `pt-br`). Very short or undetectable text falls back to `en-us`. It then picks a **standard** tier voice for that language, or a **cheaper** tier voice if none—deterministic catalog order. If the detected language has no supported voice, the job ends with `failed_file_processing`; pass `voiceId` and `accent` explicitly.

Returns `jobId`. Poll with `get_files_job`, then call `get_files_result` when status is `completed`. Consumes credits.

### `get_files_job`

| Parameter | Required | Description |
|---|---|---|
| `jobId` | Yes | UUID from `create_files_job`. |

Returns status (e.g. `processing`, `completed`, `failed`), `progressPercentage`, and `audioId` when ready.

### `get_files_result`

| Parameter | Required | Description |
|---|---|---|
| `jobId` | Yes | Completed job UUID. |

Returns `audioUrl`, `audioId`, `duration`, and `wordTimings` (`word`, `startTime`, `endTime`).

### `generate_x_audio`

| Parameter | Required | Description |
|---|---|---|
| `voiceId` | Yes | Voice UUID from `list_voices` (`product`: `x`). |
| `accent` | No | Accent code; defaults to `en-us`. |
| `postId` | One of postId/postUrl | Numeric X post id. |
| `postUrl` | One of postId/postUrl | Full X post URL. |

Returns `generationId`, `audioUrl`, `duration`, and `wordTimings` when ready (often immediately). Poll `get_x_generation` if needed. Consumes credits.

### `get_x_generation`

| Parameter | Required | Description |
|---|---|---|
| `generationId` | Yes | UUID from `generate_x_audio`. |

Returns the same audio metadata as a successful `generate_x_audio` call. **Only generations linked to your account are returned**; other ids respond with generation not found.

## Authentication

**OAuth (recommended).** Add the URL `https://api.narrador.io/mcp` to your client. The client discovers the authorization server (`https://narrador.io/api/auth`); sign in on narrador.io and approve access. Scopes: `narrador:files`, `narrador:x`.

**API key.** Create a key in the narrador web app and send it as `Authorization: Bearer <key>` or `x-api-key: <key>`. Keys are scoped per product:

| Product | Prefix |
|---|---|
| narrador files | `nrd_files_` |
| narrador X | `nrd_x_` |

## Connect

### Claude Desktop

Settings → Connectors → **Add custom connector** → URL `https://api.narrador.io/mcp` → Connect, then complete OAuth on narrador.io.

### Cursor

Project (`.cursor/mcp.json`) or user (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "narrador": { "url": "https://api.narrador.io/mcp" }
  }
}
```

With an API key:

```json
{
  "mcpServers": {
    "narrador": {
      "url": "https://api.narrador.io/mcp",
      "headers": { "Authorization": "Bearer nrd_files_..." }
    }
  }
}
```

### VS Code

`.vscode/mcp.json`:

```json
{
  "servers": {
    "narrador": { "type": "http", "url": "https://api.narrador.io/mcp" }
  }
}
```

### Claude (claude.ai)

Same as Claude Desktop: **Add custom connector** with `https://api.narrador.io/mcp`.

### Claude Code

```bash
claude mcp add --transport http narrador https://api.narrador.io/mcp
```

### Windsurf (`~/.codeium/windsurf/mcp_config.json`)

```json
{
  "mcpServers": {
    "narrador": { "serverUrl": "https://api.narrador.io/mcp" }
  }
}
```

### Any other client

Point it at `https://api.narrador.io/mcp` (Streamable HTTP). If the client has no OAuth support, send an API key in the `Authorization` header.

## Example prompts

- "Check my narrador credits, then convert this PDF to audio with narrador files."
- "List standard-tier voices for narrador files in Spanish, then create a job for this text."
- "Turn this X post into audio with narrador X: https://x.com/…/status/…"

## License

MIT. See [LICENSE](LICENSE).
