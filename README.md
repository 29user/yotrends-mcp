# YoTrends MCP server

Connect Claude, Cursor or any MCP client to live YouTube and TikTok trend data,
and turn a trend into a publish-ready text pack without leaving the chat.

This is a **remote MCP server**. There is nothing to install and no package to
run — you point your client at an HTTPS endpoint and authenticate with a token
from your YoTrends account.

```
https://yotrends.ai/mcp/
```

Transport: Streamable HTTP. Auth: `Authorization: Bearer ytm_…`.

## What you can do with it

Nine tools, covering the loop from "what is happening" to "here is the copy":

| Tool | What it answers |
|---|---|
| `whoami` | Which plan is connected, how many AI credits are left, what the limits are |
| `search_trends` | What is trending right now, by region, platform, format and window |
| `list_top_creators` | Which creators are appearing in trends most often |
| `list_trend_clusters` | Which topics are *starting* to rise, before they are obvious |
| `list_my_topics` | The topics this account already tracks |
| `get_topic_feed` | Videos matching one of those topics, matched live |
| `create_topic` | Start tracking a new topic from a description and keywords |
| `get_topic_digest` | An AI read of why a topic is moving, with the key videos |
| `generate_text_pack` | Titles, hooks, a script outline, description and tags for one angle |

Full parameters and return shapes: [docs/tools.md](docs/tools.md).

Two tools spend AI credits — `get_topic_digest` costs 2, `generate_text_pack`
costs 3. If the balance is empty the call returns a message saying so and
charges nothing.

## Requirements

- A YoTrends account on a paid plan. MCP access is gated on the plan, and the
  free tier does not include it.
- A token: **YoTrends → Settings → Connect to AI**. Tokens start with `ytm_`.
  Treat one like a password; it carries your plan's access.

## Which clients this works with today

| Client | Works | How |
|---|---|---|
| Claude Code | yes | `--header` on `claude mcp add` |
| Cursor | yes | `headers` in `mcp.json` |
| VS Code | yes | `headers`, with the token as a prompted input |
| Claude Desktop | yes | through `mcp-remote`, which passes the header |
| Claude web and the Desktop connector UI | **not yet** | needs OAuth |

The last row is a real limitation, not an oversight. Claude's custom-connector
dialog takes a server URL and, optionally, an OAuth client ID and secret — there
is no field for an arbitrary header, so a static bearer token cannot be entered.
The MCP specification expects HTTP servers to authorise over OAuth 2.1: the
server advertises its authorisation server through protected-resource metadata,
the client discovers it from the `WWW-Authenticate` header on a 401, and the
flow runs with PKCE.

This server currently answers 401 with a plain `WWW-Authenticate: Bearer` and
serves no `/.well-known/oauth-protected-resource`, so that discovery cannot
start. Until OAuth lands, use one of the clients above.

## Setup

### Claude Code

```bash
claude mcp add --transport http yotrends https://yotrends.ai/mcp/ \
  --header "Authorization: Bearer ytm_YOUR_TOKEN"
```

### Claude Desktop

Claude Desktop connects to remote servers through `mcp-remote`. Add this to
`claude_desktop_config.json` (see [examples/claude-desktop.json](examples/claude-desktop.json)):

```json
{
  "mcpServers": {
    "yotrends": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote", "https://yotrends.ai/mcp/",
        "--header", "Authorization: Bearer ytm_YOUR_TOKEN"
      ]
    }
  }
}
```

### Cursor

`.cursor/mcp.json` in your project, or the global one — see
[examples/cursor.json](examples/cursor.json).

### VS Code

[examples/vscode.json](examples/vscode.json) uses an input prompt so the token
is not stored in the file.

## Checking it works

Ask the assistant to call `whoami` first. It returns your plan, remaining AI
credits and limits, and it costs nothing. If it answers, everything downstream
will work.

A `401` with `{"error":"unauthorized"}` means the token is missing, wrong, or
belongs to a plan without MCP access. Regenerate it in Settings.

## What this repository contains

Configuration and documentation for the hosted server, plus `server.json` for
the MCP registry. The server itself runs as part of YoTrends and is not
distributed here — there is no local build, and the endpoint above is the whole
integration surface.

## Links

- Product: https://yotrends.ai/en/
- What the integration is for: https://yotrends.ai/en/create-content-from-trends/
- Support: hello@yotrends.ai

## License

MIT — see [LICENSE](LICENSE). The licence covers this repository's
configuration and documentation.
