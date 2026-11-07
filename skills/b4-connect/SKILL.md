---
name: b4-connect
description: Step-by-step procedure for connecting an AI app to the MCP server built into b4, the DPI-bypass and routing service on the user's router, and for fixing that connection. Load it before answering whenever the user asks how to connect Claude Code, Claude Desktop, ChatGPT, Codex, Cursor, VS Code, LM Studio, Gemini CLI, Jan or another AI app to b4, when this chat has no b4 tools, when the b4 MCP server fails to start or connect, answers 401, 403, 404 or 405, or its certificate is refused, or when b4 tools are missing after a switch was turned on - even when the user only pastes a config file or an error and the fix looks obvious. Russian requests too - "подключить ИИ к b4", "настроить MCP", "нет инструментов b4", "MCP не подключается", "ошибка 401", "токен MCP". Works without b4's tools.
compatibility: Works without b4's MCP tools; when they are connected it confirms the connection with b4_status. The user needs access to b4's web interface (Settings > Integrations > MCP server) and to the AI app's MCP settings or config file.
---

# Connecting an AI app to b4

b4 is a DPI-bypass and traffic-routing service on the user's router. It serves MCP at `/api/mcp` on its web interface's address and port; the model runs in the user's AI app, and for MCP b4 contacts no AI provider and needs no API key. Reply in the user's language. When you reply in Russian, use the labels of b4's Russian interface: Settings > Integrations > MCP server = Настройки > Интеграции > MCP-сервер; Enable MCP server = Включить MCP-сервер; Access token = Токен доступа; Generate = Сгенерировать; Regenerate = Пересоздать; Save Changes = Сохранить; Client configuration = Настройка клиента; Copy = Копировать; the placeholder `<token>` = `<токен>`; Allow configuration changes = Разрешить изменение настроек; Allow active probes = Разрешить активные проверки; Settings > System > Web Server = Настройки > Система > Веб-сервер; Bind Address = Адрес привязки; Port = Порт; TLS Certificate = TLS Сертификат; TLS Key = TLS Ключ; Restart Service = Перезапустить сервис; Logs = Логи; Dashboard = Главная; Web server TLS is not usable, serving plain HTTP instead = TLS веб-сервера не работает, вместо него используется обычный HTTP; System Info under Settings > System > Service = Информация о системе в Настройки > Система > Сервис; Copy JSON = Копировать JSON; Import/Export = Импорт/Экспорт; Discovery = Дискавери. Sets = Сеты (a top-level page, not under Settings); Logs = Логи (a top-level page with Start trace = Начать трассировку); Log Level = Уровень логирования (Settings > System > Service); Update available = Доступно обновление.

## 0. Are b4's tools already here?

Apps may prefix tool names (`mcp__b4__b4_status` in Claude Code, `mcp_b4_b4_status` in Gemini CLI and VS Code) or load tools only on demand. Look for a tool whose name ends in `b4_status`, use the app's tool search, or call `b4_status` with `{}`; decide only after that. A refusal naming an unexpected property means arguments were sent: retry with `{}`.

- **It answers**: go to section 4.
- **Otherwise** this chat has no working b4 connection: never set up, failed to start, or switched off or deselected in the app (Cursor's **Customize** toggle, VS Code's tool picker), so check the app's server status first. Do not guess at b4's state; work through sections 1 to 3. Setup page: https://docs.b4core.app/docs/settings/mcp/ (Russian: https://docs.b4core.app/ru/docs/settings/mcp/).

**The token is a password.** Never ask for it, repeat it, or print a config file or log that holds it. Write `<token>` in every command and entry. If it was pasted anyway: **Regenerate**, **Save Changes**, update the app. Never suggest committing a file that holds it.

## 1. b4's side, in b4's web interface (never in the AI app)

1. **Settings > Integrations**, card **MCP server**: turn on the switch right of the card's title (**Enable MCP server**). The token field, the two switches and **Client configuration** appear only then.
2. **Generate** next to **Access token**, then **Save Changes**: the token is stored only on save.
3. **Copy** under **Client configuration** gives the whole entry with the full token; the copy icon in the **Access token** field, the bare token. Never retype the card's snippet: it shortens the token. Copied before **Generate**, the entry holds the placeholder `<token>` (Russian interface: `<токен>`).
4. The entry is `"b4": { "url": "...", "headers": { "Authorization": "Bearer ..." } }`, no wrapper. Its URL is the web interface's address in the browser plus `/api/mcp`, and must be reachable from the app's computer (`localhost` only on the same machine).

- **Token**: while saved, it is the only credential `/api/mcp` accepts and opens nothing else in b4; it survives restarts; **Regenerate** replaces it for every app. Without one, the endpoint follows the web login (whose token expires after a day unused and on restart), or, with no login, is open to anyone who reaches the port. Always generate one.
- **Allow configuration changes** and **Allow active probes**, on the same card, are off by default and independent; while one is off, its tools are not published. They apply on **Save Changes**, but a connected app keeps its old tool list until it reconnects.
- **HTTP or HTTPS**: the endpoint uses the web server's address, port and certificate (**Settings > System > Web Server**). With both TLS fields empty it is plain HTTP and the token crosses the network readable: trusted network only. HTTPS: https://docs.b4core.app/docs/settings/security/#https; the URL's scheme must match. **Port**, **Bind Address**, **TLS Certificate** and **TLS Key** apply only after **Save Changes** and **Restart Service**.

## 2. The app's side

Put the entry into the app's own format, named `b4`. Open [references/apps.md](references/apps.md) for each common app's exact entry, command, status check and reconnect step, and for the traps behind most failed setups (`"type": "http"`, `bearer_token_env_var`, committed project files, variables a desktop app cannot see). Give only what it or the app's own documentation states. An unlisted app needs MCP over streamable HTTP with a custom `Authorization` header: send the user to its MCP documentation.

**Cloud clients** (claude.ai, ChatGPT on the web, connectors added in Claude Desktop's connector settings) connect from the vendor's cloud and cannot reach a home router without a public address or a vendor tunnel that only some organization accounts have: suggest a local app (for Claude Desktop, a local Code tab session). Never suggest exposing b4 to the internet as a fix. ChatGPT on the web cannot send b4's token at all (only OAuth or no authentication); for the one claude.ai case where exposure could work, and its warning, see the apps file.

## 3. When it fails: evidence first

Ask for the app's exact error or server status (token removed), the app and its version, and the entry's URL. Then the **browser test**: open that URL in a browser on the app's computer. A timeout or "connection refused" means the address, port or network is wrong; a certificate warning, that the app may refuse the certificate too; an HTTP status is b4 answering, and since the browser sends no token, it shows which check is live.

**b4's Logs**, in this order: wait 30 seconds after the browser test, open the **Logs** page (it shows only lines written while the web interface is open), reconnect the app, then take the first `mcp: refused a request from <address>: <reason>` line, with the app's computer as `<address>`. b4 lists one refusal per 30 seconds from any caller and only counts the rest. The token never appears in the log.

Status and body, from the browser or the app:
- **401 `invalid or missing MCP token`**: a saved MCP token is checked. For the app, the Logs reason decides: `no MCP token was presented` (no header, or its variable is empty); `the MCP token presented is wrong` (regenerated, retyped from the snippet, an old login token, extra characters, or an unset variable sent as written; apps file); `an Authorization header was sent, but its scheme is not Bearer` (`Bearer ` missing).
- **401 `unauthorized` or `invalid token`**: the web login answered: no saved MCP token, or the MCP server off while a login is set. Finish section 1.
- **404 `mcp server is disabled`**: **Enable MCP server** is off or not saved.
- **405** in the browser: no MCP token is saved and there is no web login, so the endpoint is open to anyone: do section 1 step 2. From the app, an entry on SSE (`"type": "sse"` in Gemini CLI, `--transport sse`): switch it to `http`. Claude Code, which can fall back to SSE by itself, may hide its first error behind the 405: read its own logs for that first error (apps file).
- **A sign-in prompt**: b4 has no OAuth. Claude Code's `! Needs authentication` means no `Authorization` header was sent: add it. Gemini CLI's `/mcp auth` message follows any 401: the Logs reason decides.
- **403**: `origin not allowed` or `cross-site requests are refused ...` comes from a browser page, not an AI app; `Forbidden: invalid Host header` from a proxy or port forward on the router. Fixes: apps file.
- **Connection refused or timeout**: b4 or its web server is down, the address or port is wrong, or **Bind Address** is `127.0.0.1` (router only). If the web interface does not open either, follow https://docs.b4core.app/docs/troubleshooting/reporting/#web-interface-does-not-open; b4 has no `--sysinfo` flag, the installer does.
- **Certificate refused**: typically self-signed and not trusted, or without the URL's address or host name in its subjectAltName (the `CN=b4` example on b4's HTTPS page has none, so trusting it is not enough). Send the user to the app's TLS documentation (Claude Code and mcp-remote read `NODE_EXTRA_CA_CERTS`). If the **Dashboard** shows **Web server TLS is not usable, serving plain HTTP instead**, use `http://`.
- **The server fails to start**: read the app's status (apps file); for Codex or the ChatGPT desktop app check `bearer_token_env_var` first.
- **Tools missing after a switch**: after **Save Changes**, reconnect or reset the tool cache (apps file); where no step is given, restart the app.
- **Two apps disagree**: compare `version`, `sets_total` and `uptime_s` from `b4_status` in each; they may reach different b4 instances.

## 4. Once it connects

Call `b4_status` and report `version`, `engine`, `you_can_change_settings` and `you_can_test_and_discover`. A false flag means that switch is off in b4's web interface; once the user turns it on and saves, the app must reconnect. Nothing on the MCP server card, no credential and no web server setting can be changed over MCP.

A working connection says nothing about whether a site loads. This skill changes and tests nothing; for a change or a site test afterwards, use the b4-troubleshoot skill if installed, otherwise:
- change a setting only on the user's request and with **Allow configuration changes** on (`you_can_change_settings`), then compare the reported `current` with what was meant (`changed: false`: b4 did not keep it);
- `b4_revert_last_change` undoes the last MCP change since b4 started, one per call; never web-interface edits, and it is refused when something else changed the configuration since;
- `b4_test_domain_now`, the only tool that says whether a site loads, sends real traffic from the router: only with the user's consent and **Allow active probes** on (`you_can_test_and_discover`).

## 5. Never

- claim the app is connected before a b4 tool returned data in this chat, or promise that a site will work;
- say a switch lives in the AI app, the IDE or the client;
- invent a config key, flag or menu path;
- call b4 broken before the browser test and the Logs line;
- claim you changed anything: the user makes every change in b4 and in the app;
- ask for, repeat or print the token, or suggest exposing b4 or disabling certificate checks as a fix.

## 6. Reply

Name the cause and its evidence (the app's error, the browser answer, the Logs line), the exact change with `<token>` for the token, where to make it, and how to check it (reconnect, then `b4_status`). Keep it short.
