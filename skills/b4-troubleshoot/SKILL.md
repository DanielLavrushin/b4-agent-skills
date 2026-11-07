---
name: b4-troubleshoot
description: Step-by-step procedure for diagnosing a problem a user reports with b4, the DPI-bypass and routing service on their router, using b4's MCP tools. Load it before answering whenever the user describes b4 not working or behaving oddly - a site or app that does not open, Discovery finding nothing, Telegram or YouTube trouble, a capability or kernel module shown as unavailable, unexpected values on the Connections page, problems that began after an update, a clash with firmware features or another proxy or VPN tool - even when they paste logs or System Info and the answer looks obvious. Russian requests too - "не открывается сайт", "b4 не работает", "после обновления", "Discovery ничего не находит". Needs b4's MCP tools.
compatibility: Needs a connection to b4's MCP server (the b4_* tools). Probes need "Allow active probes" and changes need "Allow configuration changes" in b4's web interface.
---

# Troubleshooting b4

b4 is a DPI-bypass and traffic-routing service on the user's router. Its MCP tools read its state; with the user's permission switches on, they can also test a site and change settings. Reply in the user's language. When you reply in Russian, use the labels of b4's Russian interface: Settings > Integrations > MCP server = Настройки > Интеграции > MCP-сервер; Allow configuration changes = Разрешить изменение настроек; Allow active probes = Разрешить активные проверки; System Info under Settings > System > Service = Информация о системе в Настройки > Система > Сервис; Copy JSON = Копировать JSON; Import/Export = Импорт/Экспорт; Discovery = Дискавери.

## 0. Check the tools first

Some apps load MCP tools only on demand, or put the server name in front of the tool name (for example `mcp__b4__b4_status`). Look for any tool whose name ends in `b4_status`, use the app's tool search if it has one, or simply call `b4_status`. Only if none of that works, do not guess at b4's state. Tell the user that this chat has no working connection to b4's MCP server: either it is not set up in this app, or it is set up but failed to start (in ChatGPT and Codex, `/mcp` lists the connected servers). Point them to the b4-connect skill if it is installed, and to the setup page: https://docs.b4core.app/docs/settings/mcp/ (Russian: https://docs.b4core.app/ru/docs/settings/mcp/). If the tools exist but every call fails to connect, b4 or its web server may not be running. b4 itself has no `--sysinfo` flag; the installer prints the system, the installed version and the kernel modules when the user runs this over SSH on the router: `curl -fsSL https://raw.githubusercontent.com/DanielLavrushin/b4/main/install.sh | sh -s -- --sysinfo` (see https://docs.b4core.app/docs/troubleshooting/reporting/#web-interface-does-not-open).

## 1. What each tool can and cannot prove

| Tool | Proves | Does not prove |
| --- | --- | --- |
| `b4_status` | b4 runs; version, capture engine, set counts, which permission switches are on | that any site works |
| `b4_diagnostics` | kernel capabilities with their detail, firewall rule groups, firewall flags, interfaces | that any site works |
| `b4_logs_tail` | errors b4 recorded (`file=errors`), the last update's transcript (`file=update`) | per-domain traffic |
| `b4_check_domain` | which set's configuration targets a name | that traffic reached b4, or that the site loads |
| `b4_recent_connections` | that a connection reached b4 recently and which set took it | that the site loads |
| `b4_test_domain_now` | whether the site loads from the router, through b4 and with b4 bypassed | anything when the user has not allowed probes |

Only `b4_test_domain_now` answers "does it load". Everything else is evidence about configuration or traffic.

## 2. Work in this order, reads first

1. **Collect what the user already gave you**: b4 version, platform, capture engine, firewall backend, exact log lines, what changed recently. Treat their System Info and logs as describing their router.
2. **Service health**: `b4_status`, then `b4_diagnostics`. Read every capability whose `available` is false together with its `detail`, check that firewall rule groups are present, and read the firewall flags in full, not only `nfqueue_works`. Then `b4_logs_tail` with `file=errors`, and `file=update` when the trouble began with an update.
3. **Read log lines literally.** Quote the exact line. The host name, address, path or command inside an error is what b4 actually tried to use; compare it character by character with what the user meant.
4. **The site**: `b4_check_domain` for the name, then `b4_get_set` for the set that covers it. Before saying anything about a setting, call `b4_get_topic` with its dotted key (for example `topic="tcp.conn_bytes_limit"`). A `query=` search matches one exact substring, so if it finds nothing, retry with the dotted key or call `b4_get_topic` with no arguments to list the keys. Never infer a unit, a default or the meaning of a number from a field name.
5. **Traffic**: ask the user to open the site once more, then call `b4_recent_connections` with `contains` set to the domain, then to the client's address. The list holds only the last 500 events, so an empty result means "nothing in that window", not "never reached b4".
6. **The installed version**: compare `b4_status.version` with the release notes at https://github.com/DanielLavrushin/b4/blob/main/changelog.md (raw text: https://raw.githubusercontent.com/DanielLavrushin/b4/main/changelog.md; Russian: changelog_ru.md). Look for FIXED entries newer than the installed version that match the symptom. If you cannot read web pages, say which version is installed and ask the user to update to the latest release and test again before deeper changes.
7. **Other software on the router**: b4 shares the firewall and the packet path with the firmware and other tools. When the evidence does not fit b4's own configuration (connection sources that are not LAN devices, packets that never reach b4, a firewall flag `diagnostics` reports as unsafe), ask the user what else handles traffic on the router: VPN clients, other transparent proxies, hardware or software acceleration in the firmware. Do not blame b4 or the other tool before the user answers.

## 3. Probes and Discovery

Only with the user's consent, and only if `b4_status` reports `you_can_test_and_discover` (the switch is **Allow active probes** in b4's web interface, under Settings > Integrations > MCP server; the AI app must reconnect after it is turned on):

- `b4_test_domain_now` to confirm a diagnosis and to check a fix. Run `b4_check_domain` first: when no enabled set targets the site, the two results compare nothing b4 does to it.
- `b4_find_bypass_strategy` runs Discovery for several minutes. Start it, tell the user, and check its status once when they ask; do not poll in a loop. After applying a result, test the site again with `b4_test_domain_now` several times.

Never use a private or local address as a test target.

## 4. Changing settings

Only when the user asks, and only if `b4_status` reports `you_can_change_settings` (the switch is **Allow configuration changes** in the same card).

- Read `b4_get_topic` for the setting first, and use `b4_list_writable_paths` instead of guessing a path.
- After `b4_set_config_value`, `b4_edit_set_targets` or `b4_manage_set`, read the value the tool reports back (`current`). If it differs from what you meant, say so; `changed: false` means b4 did not accept it.
- A setting that applied is not a working site: test again.
- `b4_revert_last_change` undoes the last change made over MCP; calling it again goes one more change back. It does not undo edits made in the web interface.
- Credentials, the web server, the MCP settings, the capture engine, the firewall backend and packet marks are never changed over MCP. Tell the user where to change them in the web interface.

If the switch is off, give the exact change as text and say where the user makes it.

## 5. When to stop and report

Stop and give the user a diagnosis when one of these holds:

- a capability is unavailable or firewall rule groups are missing: that comes before any strategy work;
- a log line names a concrete failure;
- the installed version is older than a changelog fix that matches the symptom;
- the evidence points at something outside b4, and you need the user's answer about it;
- you need data only the user has. List exactly what: the log lines, the set's JSON (**Copy JSON** on the set's Import/Export tab), the System Info JSON (**System Info** under Settings > System > Service, then **Copy JSON**); see https://docs.b4core.app/docs/troubleshooting/reporting/.

## 6. Never claim

- that a site loads or works, unless `b4_test_domain_now` through b4 succeeded;
- that b4 "is working" because `b4_status` or `b4_diagnostics` look fine;
- that a change fixed anything before it was tested;
- a setting's unit, default or meaning without `b4_get_topic`;
- that something is a b4 bug without a matching log line or changelog entry; say "possibly" and suggest a report instead;
- that you changed, tested or applied anything you did not.

## 7. Reply

Give the diagnosis with the evidence behind it (tool and field), the change to make and where to make it, and what to send if it is still not resolved. Keep it short.
