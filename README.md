> **English** | [**中文**](README.zh.md)

# dsh-vercel-mcp

Vercel MCP connection for DeepSeek Harness (DSH): the official
Vercel API MCP server ([mcp.vercel.com](https://mcp.vercel.com))
with the full OAuth 2.0 client dance — dynamic client registration, PKCE
S256, loopback callback on the GUI's own web server, refresh-token grant —
plus a web settings panel.

Once authorized, the Vercel platform tools (deployments, projects, domains,
environment variables, DNS records, deploy code, and more) become available
in agent sessions as `mcp__vercel__*`, auto-refreshing its token on 401.

## Why this plugin

DSH's built-in `@deepseek-ai/dsh-mcp-client` supports static headers only —
it cannot complete the OAuth flow that `mcp.vercel.com` requires
(`WWW-Authenticate: Bearer realm="OAuth"`). This plugin implements the
OAuth client itself on the official `@modelcontextprotocol/sdk` primitives,
so the connection is genuinely authenticated, not token-pasted.

## Install

```sh
# local development (link)
dsh plugin --profile web add link:/path/to/dsh-vercel-mcp
```

The bundle patch mounts the plugin row automatically; a Host restart is
needed for the layer to compose. Tokens live in
`~/.dsh/dsh-vercel-mcp.json` (mode 0600).

## Authorize

Two ways, both starting the same OAuth flow against Vercel:

1. **Settings panel** (Web GUI → 设置 → Vercel MCP): click 开始授权,
   the browser opens Vercel's consent page, and the loopback callback
   completes automatically. 测试连接 lists the discovered tools.
2. **Chat**: ask the agent to run `vercel_mcp_oauth_start`, open the
   returned `authorizeUrl`, or paste the callback `code` back into
   `vercel_mcp_oauth_finish`.

## Tools

Agent-facing helpers (mounted on `ctx.tools`):

| tool | what it does |
|---|---|
| `vercel_mcp_status` | authorization + connection state, tool count (no secrets) |
| `vercel_mcp_oauth_start` | begin the OAuth flow, return the browser URL |
| `vercel_mcp_oauth_finish` | manual code-paste fallback |
| `vercel_mcp_test` | connect and list the MCP server's tools |
| `vercel_mcp_clear` | wipe credentials and disconnect |

Plus the MCP server's own tools as `mcp__vercel__*`.

## API routes

All loopback-only (`127.0.0.1`), except the OAuth callback which accepts
the cross-site landing from `mcp.vercel.com` while still verifying the
OAuth `state`:

- `GET  /api/dsh-vercel-mcp/status`
- `POST /api/dsh-vercel-mcp/oauth/start`
- `GET  /api/dsh-vercel-mcp/oauth/callback` (browser landing)
- `POST /api/dsh-vercel-mcp/oauth/finish`
- `POST /api/dsh-vercel-mcp/oauth/refresh`
- `POST /api/dsh-vercel-mcp/test`
- `POST /api/dsh-vercel-mcp/clear`

## Development

```bash
pnpm install
npm run typecheck
npm run test                  # offline store/provider smoke
DSH_VERCEL_MCP_LIVE=1 npm run test   # live OAuth phase-1 against mcp.vercel.com
npm run build                 # tsc declarations + tsdown (lib/ + lib/client.js)
```

## License

MIT
