# App entries and refusal details

Open this file before giving any app-side step (an entry, a command, a status check, a reconnect step), when the app reports 403 or 405 or `the MCP token presented is wrong`, and before saying anything about exposing b4 for a cloud client. Write `<url>` for the copied entry's URL and `<token>` for the token; the user fills in both. Name the server `b4`; several b4 entries can sit side by side under different names. Never commit a file that holds the token.

## Per app

| App | Where the entry goes | Status and reconnect |
| --- | --- | --- |
| Claude Code CLI, in a terminal | `claude mcp add --transport http b4 <url> --header "Authorization: Bearer <token>"`; local scope by default, `--scope user` for every project, both kept in `~/.claude.json`. `.mcp.json` is the project's file, meant to be committed: there add `"type": "http"` and write `"Authorization": "Bearer ${B4_MCP_TOKEN}"`, which Claude Code fills from that environment variable | `claude mcp list` and `claude mcp get b4` show the status, and the HTTP status on failure; `/mcp`, the server, **Reconnect** |
| Codex CLI and the ChatGPT desktop app (they share `~/.codex/config.toml`) | `[mcp_servers.b4]` with `url = "<url>"` and either `http_headers = { "Authorization" = "Bearer <token>" }` or `bearer_token_env_var = "B4_MCP_TOKEN"`. The desktop app's Settings > MCP servers > Add server form documents only a name and a URL, so the token goes into `config.toml` | desktop: **Restart** after saving; `/mcp` in the composer or in the Codex TUI lists connected servers |
| Cursor | `~/.cursor/mcp.json` (all projects) or the project's `.cursor/mcp.json`, under `mcpServers`; the copied entry fits as is. In the project file write `"Bearer ${env:NAME}"` in `headers` instead of the token | **Customize**, toggle the server off and on (a server toggled off does not load or appear in chat); Output panel, **MCP Logs** |
| VS Code | `.vscode/mcp.json` or the user `mcp.json`, under `servers`, plus `"type": "http"`; an `inputs` entry of type `promptString` with `"password": true` and `"Bearer ${input:b4-token}"` makes VS Code ask for the token (docs example). The Copilot harness does not receive servers that need `${input:...}` | **MCP: List Servers**, the server, Restart or Show Output; **MCP: Reset Cached Tools** for a stale tool list |
| LM Studio 0.3.17 or later | Program tab, Install > Edit mcp.json, under `mcpServers`, no `type`; the copied entry fits as is | its docs give no reconnect step: restart the app |
| Gemini CLI | `gemini mcp add --transport http --header "Authorization: Bearer <token>" -s user b4 <url>` (the default scope is the project). It writes `url` with `"type": "http"`; use the same two keys in `settings.json`. The older `httpUrl` key still works; `"type": "sse"` selects SSE | `/mcp list`; restart the CLI to reconnect |
| Jan | its `mcp_config.json` entry takes `"type": "http"`, `url` and a `headers` map, so add `"type": "http"` to the copied entry: without it Jan drops the entry. A manual `Authorization` header turns its OAuth off | restart the app; where to enter it: Jan's MCP documentation |
| Claude Desktop | see the Claude Desktop item below | quit and restart Claude Desktop |
| Antigravity CLI (`agy`) | `agy mcp add -H "Authorization: Bearer <token>" b4 <url>`, or by hand in `~/.gemini/config/mcp_config.json` under `mcpServers` with `serverUrl` (not `url`) and a `headers` map; every key directly under `mcpServers` must be a server, so a setting such as `timeoutSeconds` placed there is skipped as an invalid server. Known issue: Antigravity speaks MCP 2026-07-28 and sends `notifications/roots/list_changed`, which b4 answers with 400, and Antigravity then drops the connection (https://github.com/DanielLavrushin/b4/issues/533); until that is fixed its b4 tools do not appear | `agy mcp list` lists the entries; restart `agy`; its log `~/.gemini/antigravity-cli/cli.log` names the failure |

## Traps

- **`bearer_token_env_var` holds the NAME of an environment variable**, and that variable holds the bare token; Codex adds `Bearer`. Putting the token itself there is a known mistake: the server then fails to start, or b4 logs `no MCP token was presented`. With `env_http_headers`, the variable's value must include `Bearer `.
- **A desktop app may not see a variable exported only in a shell startup file** (`~/.bashrc`, `~/.zshrc`, `~/.config/fish/config.fish`); apps differ, so check the app's documentation. A static `http_headers` entry avoids that; the file then holds the token as plain text, so it stays private and out of any repository.
- **LM Studio**: its docs warn that servers built for Claude, ChatGPT or Gemini can use excessive amounts of tokens and trigger frequent context overflows in a local model. With the two switches off, b4 publishes fewer tools.
- **Claude Desktop**: a custom connector added in its connector settings connects from Anthropic's cloud and cannot reach a router on a home network. Use a local session in the Code tab: it loads servers from `~/.claude.json` and `.mcp.json` (where the Claude Code CLI keeps them) and from `claude_desktop_config.json`. The terminal commands in the Claude Code CLI row are for the standalone CLI: the Code tab handles terminal-dialog commands differently, and Desktop's documentation does not say that it installs the CLI. `claude_desktop_config.json` (macOS `~/Library/Application Support/Claude/`, Windows `%APPDATA%\Claude\`; if it is missing, the mcp-remote README points to Settings > Developer) takes the stdio bridge mcp-remote (Node 18 or later), and an entry there serves both the chat and the Code tab. Then quit and restart Claude Desktop; its MCP logs are `~/Library/Logs/Claude/mcp*.log` on macOS:
  `"b4": { "command": "npx", "args": ["mcp-remote", "<url>", "--header", "Authorization:${AUTH_HEADER}"], "env": { "AUTH_HEADER": "Bearer <token>" } }` inside `mcpServers`. Add `"--allow-http"` to `args` when b4 serves plain HTTP on the LAN. The form without spaces around `:` avoids an argument-escaping bug the mcp-remote README describes.

## `the MCP token presented is wrong`

The app sent a value that is not the saved token:
- regenerated since, retyped from the shortened snippet on the card, or an old web login token;
- extra characters in the value: quotes, the `<` `>` of the placeholder, the `…` of the shortened snippet (b4 trims spaces around the scheme and the token, so whitespace alone is not the cause);
- an unset variable sent as written: when `${B4_MCP_TOKEN}` names a variable that is not set, Claude Code sends the text `${B4_MCP_TOKEN}` unexpanded and names the variable in a warning in `claude mcp list` and `/mcp`.

## 405 from the app

b4's endpoint answers only POST. An entry on the SSE transport opens with a GET and gets 405: `"type": "sse"` in Gemini CLI's `settings.json`, or `--transport sse` in `claude mcp add` or `gemini mcp add`. Use `http` instead.

An app can also switch to SSE by itself: Claude Code 2.1.265 or later tries HTTP first and switches to SSE when the server does not accept it, and can then report the retry's 405 instead of the first error. That first error is not a b4 refusal: a request b4 refuses (401, 403, 404) is refused again on the SSE retry, which then never reaches the 405, and the 405 itself leaves no line in b4's Logs. Gemini CLI also retries over SSE when an entry with `url` and no `type` failed for any reason other than a 401, but when that retry fails it reports the first error, not the 405. Pin the transport where the app allows it (`"type": "http"` in Gemini CLI). In Claude Code, `--transport http` is already the setting that falls back, and which attempt's status `claude mcp get b4` then shows is not documented, so read the app's own logs for the first error (untested).

## 403

- `origin not allowed` or `cross-site requests are refused ...`: the request came from a browser page. AI apps send no origin. A web page on a host name needs that name in the allowed origins of the config file; see the setup page.
- `Forbidden: invalid Host header`: b4's own checks passed, then the MCP library refused a request that reached b4 over the router's loopback interface (a reverse proxy or port forward on the router that connects to `127.0.0.1` or `localhost`) with a Host header naming another address. b4 logs no `mcp:` line for it. Point the app at b4's own LAN address and port; that needs a **Bind Address** other than `127.0.0.1`, then **Save Changes** and **Restart Service**. The allowed origins do not affect this check.

## Cloud clients and exposure

- **ChatGPT on the web** offers only OAuth or no authentication for a custom server. b4 accepts only a bearer token and refuses to expose its web interface without a login, so exposing b4 cannot make ChatGPT on the web work.
- **claude.ai** can send fixed request headers, `Authorization` among them, only as a beta for a limited set of organizations. Only there could exposure work, and only if the user insists, give https://docs.b4core.app/docs/settings/security/#expose-to-internet with this warning: it puts the web interface, its API and the MCP server on the internet together, and it needs a login and HTTPS first.
- **Vendor MCP tunnels** (Claude's is an Enterprise-only research preview; ChatGPT's Secure MCP Tunnel needs a Platform-organization role) reach a server without a public address, but the client must still send b4's token: ChatGPT on the web cannot, claude.ai only with the request-headers beta. Send such a user to the vendor's documentation.
