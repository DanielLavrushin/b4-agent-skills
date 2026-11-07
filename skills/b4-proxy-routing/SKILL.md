---
name: b4-proxy-routing
description: Step-by-step procedure for b4 sets that route traffic - sending sites or devices through a VPN interface or a SOCKS5 upstream, deciding which set takes a connection, or bypassing everything except some sites - using b4's MCP tools. Load it before answering whenever the user asks how to route, proxy, exclude or prioritise traffic with b4 sets, or reports routed traffic going direct, proxied connections without a site name, a routing or proxy mode reported unavailable, sets applied in the wrong order, or DNS breaking after a routing set was turned on - even when they paste their config and the answer looks obvious. Russian requests too - "пустить через VPN", "через socks5", "идёт мимо прокси", "всё кроме", "порядок сетов". Needs b4's MCP tools.
compatibility: Needs a connection to b4's MCP server (the b4_* tools). Changes need "Allow configuration changes" in b4's web interface.
---

# Routing sites and devices with b4 sets

b4 is a DPI-bypass and traffic-routing service on the user's router. A set's `routing` block can send what the set matches out of a VPN interface, to a SOCKS5 upstream, over Telegram's WebSocket edge, or drop it. Reply in the user's language. When you reply in Russian, use the labels of b4's Russian interface: Settings > Integrations > MCP server = Настройки > Интеграции > MCP-сервер; Allow configuration changes = Разрешить изменение настроек; Allow active probes = Разрешить активные проверки; System Info under Settings > System > Service = Информация о системе в Настройки > Система > Сервис; Copy JSON = Копировать JSON; Import/Export = Импорт/Экспорт; Discovery = Дискавери. Sets = Сеты (a top-level page, not under Settings); Logs = Логи (a top-level page with Start trace = Начать трассировку); Log Level = Уровень логирования (Settings > System > Service); Update available = Доступно обновление.

## 0. Check the tools first

Some apps load MCP tools only on demand, or put the server name in front of the tool name (for example `mcp__b4__b4_status`). Look for any tool whose name ends in `b4_status`, use the app's tool search if it has one, or simply call `b4_status`. Only if none of that works, do not guess at b4's configuration. Tell the user that this chat has no working connection to b4's MCP server: either it is not set up in this app, or it is set up but failed to start (in ChatGPT and Codex, `/mcp` lists the connected servers). Point them to the b4-connect skill if it is installed, and to the setup page: https://docs.b4core.app/docs/settings/mcp/ (Russian: https://docs.b4core.app/ru/docs/settings/mcp/).

## 1. Read before you explain

1. `b4_status`: version, capture engine, whether changes are allowed.
2. `b4_list_sets`: every set in list order, which are enabled, what each targets.
3. `b4_get_set` for each set that could take the traffic, not only the one the user named.
4. `b4_check_domain` for the sites involved: which sets target each name and how (exact, a parent listed, a regexp, only a subdomain listed).
5. `b4_diagnostics`: the kernel capabilities with their `available` flag and `detail`, and `firewall.routing` (whether routing rules are installed, for how many sets).
6. `b4_logs_tail` with `file=errors`: routing failures are logged there. Quote the lines exactly.

Treat the user's own System Info, set JSON and logs as describing their router; the tools describe the b4 you are connected to.

## 2. Look up every setting in play

Before you say what a setting does, call `b4_get_topic` with its dotted key, for example `topic="routing.mode"`. Do this for each routing and target setting the set uses or the user asks about, including:

- `routing.mode`, `routing.egress_interface`, the `routing.upstream.*` settings, `routing.source_interfaces`, `routing.router_traffic`, `routing.ip_ttl_seconds`;
- `targets.sni_domains`, `targets.ip`, `targets.ip_priority`, `targets.source_devices`.

A `query=` search matches one exact substring, so "router traffic" finds nothing while `topic="routing.router_traffic"` does. If you are unsure of a key, call `b4_get_topic` with no arguments to list them. Never infer a default, a unit or the effect of a value from a field name, and never fill a gap from memory: if no topic covers it, say so.

## 3. Which set takes a connection

Do not assume list order decides. The rules for name matches against address matches, for `targets.ip_priority` and for device-scoped sets are in the topics `targets.sni_domains` and `targets.ip_priority`; read them, then apply them to the user's sets. Then check what actually happened:

- ask the user to open the site once more, then call `b4_recent_connections` with `contains` set to the domain, then to the destination or client address;
- the `sni_set` or `ip_set` column names the set that handled the flow, and the optional action shows what it did;
- the list holds only the last 500 events, so an empty result means "nothing in that window".

## 4. Checks that come before strategy or set changes

- **Capability**: a routing mode that needs a kernel capability does nothing when `b4_diagnostics` reports it unavailable. Report the capability and its `detail` as they stand.
- **Rules installed**: `firewall.routing` in `b4_diagnostics` compares `sets` (the routing sets the configuration wants) with `installed`; when they differ, `error`, `set_errors` and `missing_tool` say why, and the error log usually has the full line.
- **Catch-all targets**: if a routing set lists `0.0.0.0/0` or `::/0`, read `routing.source_interfaces` and `routing.router_traffic` before anything else: they decide whether the router's own traffic, its DNS included, is sent the same way. Check this first when DNS or the whole network broke right after a routing set was turned on.
- **The installed version**: compare `b4_status.version` with the latest release. The released versions are at https://github.com/DanielLavrushin/b4/releases (the one marked Latest), and the bottom of b4's side menu reads **Update available** when a newer stable release exists. `changelog.md` on main lists versions that are not released yet, with dates, so never take its top entry for the latest release; read it only for what a version fixed. If you cannot open the releases page, ask the user what the side menu shows. For fixes, read https://raw.githubusercontent.com/DanielLavrushin/b4/main/changelog.md. Look for FIXED entries newer than the installed version that match the symptom. If you cannot read web pages, say which version is installed and suggest updating to the latest release and testing again.
- **DNS**: a set that targets names also routes the addresses b4 learns for them, from the DNS answers it sees among other sources (see the topic `routing.ip_ttl_seconds`). If those addresses never show up in the set's traffic, ask the user how devices and the router resolve names (the router's resolver, a local DNS or DoH proxy, DNS set in the devices) before changing the set. Read `dns.enabled` and `dns.target_dns` with `b4_get_topic` if the set uses its own DNS.

## 5. Changing sets

Only when the user asks, and only if `b4_status` reports `you_can_change_settings`. The switch is **Allow configuration changes** in b4's web interface, under Settings > Integrations > MCP server; the AI app must reconnect after it is turned on. Without it, give the exact change as text and say where the user makes it in the web interface.

- Use `b4_list_writable_paths` instead of guessing a path, `b4_set_config_value` for a setting, `b4_edit_set_targets` for domain, address and device lists, and `b4_manage_set` to create, move, enable or delete a set. Read each tool's own note on what it changes.
- After each write, read the value the tool reports back (`current`) and compare it with what you meant; `changed: false` means b4 did not accept it.
- Confirm the effect with `b4_recent_connections` after the user reproduces the traffic. A saved value is not a working route.
- `b4_revert_last_change` undoes the last change made over MCP; calling it again goes one more change back. It does not undo edits made in the web interface.
- Upstream credentials, the web server, the capture engine, the firewall backend and packet marks are never changed over MCP. Send the user to the web interface for those.

## 6. When to stop and report

- a capability is unavailable or routing rules are not installed: report that first;
- a log line names a concrete routing failure;
- the installed version is older than a changelog fix that matches the symptom;
- the traffic never reaches b4, or reaches it from an unexpected source: ask what else on the router handles that traffic;
- you need the set's JSON (**Copy JSON** on the set's Import/Export tab), the System Info JSON (**System Info** under Settings > System > Service, then **Copy JSON**) or log lines from the user; see https://docs.b4core.app/docs/troubleshooting/reporting/.

## 7. Never claim

- that you can change a setting when the write tools (`b4_set_config_value`, `b4_edit_set_targets`, `b4_manage_set`) are not in your tool list, whatever `b4_status` reports;
- that traffic goes through the VPN or proxy unless `b4_recent_connections` shows the connection handled by that set;
- which set wins without having read the matching topics and the user's set list;
- a setting's default, unit or effect without `b4_get_topic`;
- that the firmware lacks a feature, or that something is a b4 bug, without the diagnostics `detail`, a log line or a changelog entry;
- that you changed or tested anything you did not.

## 8. Reply

Name the set that takes the traffic and why, with the evidence (tool and field), the change to make and where, and what to send if it still does not work. Keep it short.
