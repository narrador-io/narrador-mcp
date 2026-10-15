# narrador MCP server

narrador turns documents and X (Twitter) posts into natural narrated audio with word-level timestamps. Upload PDFs, Word, EPUBs or text, pick from multilingual voices (English, Spanish, Portuguese and regional accents), start audio generation jobs, check their status and download the audio with synced timings. Also generates audio for X posts. Secure OAuth sign-in with your narrador.io account, no API key needed.

Remote [Model Context Protocol](https://modelcontextprotocol.io) server for **narrador**. It lets AI agents (Claude, Cursor, VS Code, Windsurf, ChatGPT and any MCP client) turn documents and X posts into natural-sounding audio.

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

This repository only holds the public listing (`server.json`) and documentation. The server is hosted by narrador, so there is nothing to install.

## Tools

| Tool | Product | What it does |
|---|---|---|
| `get_account` | both | Shows the authenticated account and what it can use |
| `list_voices` | both | Lists available voices and languages |
| `create_files_job` | narrador files | Starts converting a document into audio |
| `get_files_job` | narrador files | Checks the status of a conversion job |
| `get_files_result` | narrador files | Gets the audio and word timings of a finished job |
| `generate_x_audio` | narrador X | Turns an X post into audio |
| `get_x_generation` | narrador X | Checks the status and result of an X generation |

## Authentication

**OAuth (recommended).** Add the URL `https://api.narrador.io/mcp` to your client. The client discovers the authorization server (`https://narrador.io/api/auth`), you sign in to narrador and approve access. Scopes: `narrador:files`, `narrador:x`.

**API key.** Create a key in the narrador web app and send it as `Authorization: Bearer <key>` or `x-api-key: <key>`. Keys are scoped per product:

| Product | Prefix |
|---|---|
| narrador files | `nrd_files_` |
| narrador X | `nrd_x_` |

## Connect

### Claude (claude.ai and Claude Desktop): custom connector
Settings → Connectors → **Add custom connector** → URL `https://api.narrador.io/mcp` → Connect, then sign in with OAuth.

### Claude Code
```bash
claude mcp add --transport http narrador https://api.narrador.io/mcp
# or with an API key:
claude mcp add --transport http narrador https://api.narrador.io/mcp --header "Authorization: Bearer nrd_files_..."
```

### Cursor (`~/.cursor/mcp.json` or `.cursor/mcp.json`)
```json
{
  "mcpServers": {
    "narrador": { "url": "https://api.narrador.io/mcp" }
  }
}
```
With an API key, add `"headers": { "Authorization": "Bearer nrd_files_..." }`.

### VS Code (`.vscode/mcp.json`)
```json
{
  "servers": {
    "narrador": { "type": "http", "url": "https://api.narrador.io/mcp" }
  }
}
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
Point it to `https://api.narrador.io/mcp` (Streamable HTTP). If the client has no OAuth support, send an API key in the `Authorization` header.

## Example prompts
- "Convert this PDF into audio with narrador files and give me the link."
- "Turn this X post into audio with narrador X: https://x.com/…/status/…"

## License
MIT. See [LICENSE](LICENSE).
