---
name: b4-report-issue
description: Step-by-step procedure for writing a short, checked bug report about b4, the DPI-bypass and routing service on the user's router, for b4's GitHub issues, with the facts collected through b4's MCP tools and the user's private data kept out. Load it before answering whenever the user wants to report a bug, open or file an issue, write to the developer, asks what to attach, or prepares System Info, a log trace or a set's JSON for someone else - even when they already have a draft or an AI-written analysis and only want it tidied, and even when b4 does not start at all. Russian requests too - "написать баг-репорт", "создать issue", "сообщить об ошибке разработчику", "что приложить к issue", "оформи отчёт об ошибке". Reads b4's MCP tools; without them it still builds the report from what the user pastes.
compatibility: Reads need a connection to b4's MCP server (the b4_* tools); without one the report is built from what the user pastes. An optional probe needs "Allow active probes" and an optional log-level change needs "Allow configuration changes" in b4's web interface. The user posts the issue; the assistant never does.
---

# Reporting a b4 problem

b4 is a DPI-bypass and traffic-routing service on the user's router. This skill helps the user write a bug report for https://github.com/DanielLavrushin/b4/issues: facts read with b4's MCP tools, exact log lines, no private data, short enough to read. The user posts it; you never do. Reply in the user's language. When you reply in Russian, use the labels of b4's Russian interface: Settings > Integrations > MCP server = Настройки > Интеграции > MCP-сервер; Allow configuration changes = Разрешить изменение настроек; Allow active probes = Разрешить активные проверки; System Info under Settings > System > Service = Информация о системе в Настройки > Система > Сервис; System Diagnostics = Диагностика системы; Copy JSON = Копировать JSON; Import/Export = Импорт/Экспорт; Discovery = Дискавери; Logs = Логи; Start trace = Начать трассировку; Stop & save = Остановить и сохранить; Log Level = Уровень логирования; Debug = Отладка; Info = Инфо; Save Changes = Сохранить; Download safe copy under Settings > System > Backup = Скачать безопасную копию в Настройки > Система > Бэкап; Watchdog = Мониторинг. Sets = Сеты (a top-level page, not under Settings); Logs = Логи (a top-level page with Start trace = Начать трассировку); Log Level = Уровень логирования (Settings > System > Service); Update available = Доступно обновление.

## 0. Check the tools first

Some apps load MCP tools only on demand, or put the server name in front of the tool name (for example `mcp__b4__b4_status`). Look for any tool whose name ends in `b4_status`, use the app's tool search if it has one, or simply call `b4_status`. Only when none of that works, tell the user that this chat has no working b4 MCP connection (not set up, or failed to start), and point to the b4-connect skill if it is installed and to https://docs.b4core.app/docs/settings/mcp/ (Russian: https://docs.b4core.app/ru/docs/settings/mcp/).

Without the tools, build the report from what the user pastes. The MCP server is part of b4's web server, so it is down too when b4 or its web interface does not start. Then the installer's report, run over SSH on the router, replaces System Info: `curl -fsSL https://raw.githubusercontent.com/DanielLavrushin/b4/main/install.sh | sh -s -- --sysinfo` (or `wget -qO-` with the same URL), plus the last lines of `errors.log` in the log directory (`/var/log/b4` by default); see https://docs.b4core.app/docs/troubleshooting/reporting/#web-interface-does-not-open. No `errors.log` means file logging is off or nothing was written, not that b4 is fine. b4 has no `--sysinfo` flag; `b4 --version` prints the version.

## 1. Before you draft

1. Compare `b4_status.version` with the user's version; if they differ, ask which router the report is for. Quote the version exactly, suffix included (`1.90.0-<suffix>`). A suffixed build is not the plain release: ask where it came from before using the changelog.
2. Find the release marked Latest at https://github.com/DanielLavrushin/b4/releases, then search https://github.com/DanielLavrushin/b4/blob/main/changelog.md (raw: https://raw.githubusercontent.com/DanielLavrushin/b4/main/changelog.md; newest first) for a FIXED, CHANGED or IMPROVED entry matching the symptom (changelog_ru.md: ИСПРАВЛЕНО, ИЗМЕНЕНО, only from 1.66.0). A match newer than the installed version, up to Latest: suggest updating and testing again, and report only if the problem stays. A match above Latest is unreleased: say the changelog lists a matching fix for an upcoming version, and suggest no update. If you cannot open the releases page, never take the changelog's top entry for the latest release (it lists unreleased versions with dates): give the installed version and ask what the bottom of b4's side menu shows (**Update available** when a newer stable release exists).
3. Suggest searching open and closed issues for the exact error line or symptom; a matching issue gets the facts as a comment.

## 2. Collect the facts, reads only

- `b4_status`: `version`, `engine`, `sets_total`, `sets_enabled`. Its `firewall_backend` is the configured choice and can read `auto`; the detected one is `firewall.backend` in `b4_diagnostics`.
- `b4_diagnostics` (fields under `diagnostics`): `system.distro`, `system.kernel`, `system.arch`, `system.is_docker`; each `kernel.capabilities` entry whose `available` is false, with its `detail`; `firewall.backend`, `firewall.nfqueue_works`, `firewall.flow_offload` with `firewall.flow_offload_safe`; whether `firewall.rule_groups` exists; `engine.start_error`; `firewall.routing` when present. It is the unmasked System Info report: take facts from it, never paste it.
- `b4_logs_tail` with `file=errors`, plus `file=update` when the problem began with an update from the web interface. Copy lines exactly, with addresses and the host names of the user's own proxies, relays and mirrors replaced by placeholders. Its `note` explains an empty result, including file logging being off. `errors.log` holds only errors, `[INIT]` startup warnings and stderr (topic `system.logging.directory`), so a warning missing from it proves nothing.
- For a site: `b4_check_domain`, then `b4_get_set` for the covering set. Ask the user to open the site once more, then call `b4_recent_connections` with `contains` set to the domain. It keeps the last 500 events, so empty means "none in that window"; its `source`, `destination` and `mac` columns are addresses.
- Watchdog: `b4_watchdog` with `action=status` (and `set` for one set); no traffic, no switch needed.
- Before the report says what a setting does, call `b4_get_topic` with its dotted key; otherwise quote the value unexplained.

## 3. Shape of the report

Required form fields: **What happened?**, **b4 version**, **Platform** and **Installation method** (both lists: ask the user; `system.distro` is only a hint), **System diagnostics (JSON)**. Optional: **Steps to reproduce**, **Debug log trace**, **Additional context**.

```
What happened?
Expected: <one line>. Instead: <one line>.

Steps to reproduce
1. Set "<name>": <targets>, <strategy>, <routing>
2. From <a LAN device | the router>, open <site> in <browser | app>
3. <what happens>
How often: <every time | some sites | after b4 has run for a while>

b4 version
<b4_status.version>

Additional context
Engine <engine>, firewall <firewall.backend>, kernel <system.kernel>
Observed: <exact lines, placeholders only>
Not verified: <one or two lines marked as a guess, or nothing>
```

- The only change allowed inside a quoted line is a placeholder.
- Keep observed apart from suspected: a guess goes under "Not verified" or is left out. No root-cause essays, code analysis or fixes the user cannot confirm.
- About 40 lines plus attachments. The developer pushes back on long AI-written reports and accepts one the poster understands and has proofread, so ask the user to read the draft and delete what they cannot confirm.
- When the developer has asked for items, collect exactly those, not a theory (in #444 his rebuild refuted the reporter's "likely cause"; his request there is in [references/attachments.md](references/attachments.md)).

## 4. Attachments and privacy

GitHub issues are public. The draft holds no host name, public or private IP address, MAC address, device name, token, password or proxy login, and only the domains the problem is about, after asking the user. Replace each value with a placeholder, the same one everywhere (`<router>`, `<device-1>`, `<wan-ip>`, `<proxy-host>`).

The attachments are System Info's **Copy JSON** (it fills the required JSON field), a debug log trace, the set's **Copy JSON**, and **Download safe copy** only when the developer asks (https://docs.b4core.app/docs/troubleshooting/reporting/; Russian: https://docs.b4core.app/ru/docs/troubleshooting/reporting/). System Info, the trace and the set's JSON are not masked, and the safe copy only in part: tell the user to open each in a text editor and replace the same kinds of values, plus personal DoH URLs and the host names of their own proxies, relays, Workers and update mirrors, error strings included. A set's **Copy JSON** keeps a proxy set's username and password (`b4_get_set` hides them): always tell the user to remove them. Before telling the user how to get an attachment or what to replace in it, open [references/attachments.md](references/attachments.md). You cannot see a file the user has not shown you, so never call it clean.

## 5. Probes and changes

The report needs neither.

- **Probe**: only with the user's consent and only if `b4_status` reports `you_can_test_and_discover` (**Allow active probes** under Settings > Integrations > MCP server; the AI app must reconnect after it is turned on). One `b4_test_domain_now` gives one observed line: "fetched from the router: through b4 <result>, bypassed <result>". It runs on the router over TCP, so a pass says nothing about the user's device, app or QUIC traffic. Never probe a private or local address.
- **Never** run Discovery (`b4_find_bypass_strategy`) or any `b4_watchdog` action other than `status`: `check` emits traffic and clears the set's cooldown and failure count (with writes on, also its heal failures and give-up), erasing what the report is about; `add`, `remove`, `enable` and `disable` change the configuration.
- **Log level for a trace**: only when the user asks and `b4_status` reports `you_can_change_settings` (**Allow configuration changes**). The app must reconnect after this switch too; until then `b4_set_config_value` is missing although the switch reads on, so the user changes **Log Level** in the web interface. Read `b4_get_topic` with `topic="system.logging.level"` first: its numbering is unusual. After `b4_set_config_value`, read `current` and the `note`: `changed: false` with "already set to this value" means nothing needed writing; otherwise b4 did not keep the value. Keep `previous`; after the trace, write it back the same way and read `current` again.
- `b4_revert_last_change` is only an immediate undo: one MCP change per call, only changes since b4 last started, never web interface edits, and refused once the watchdog, the web interface or another tool has changed the configuration since, which a trace of up to 15 minutes allows.
- Credentials, the web server, the MCP settings, the capture engine, the firewall backend and packet marks are never changed over MCP; send the user to the web interface.

## 6. Never

- cite a changelog entry as related unless it names the same feature and symptom (entries about b4's own Telegram or MTProto connections are not about the user's sites);
- summarise log rows into new ones: quote each row with its own timestamp, and keep separate bursts separate;
- that you can change a setting when the write tools (`b4_set_config_value`, `b4_edit_set_targets`, `b4_manage_set`) are not in your tool list, whatever `b4_status` reports;
- that nothing changed on b4's side because the uptime is long or the error log is empty: settings and sets change without a restart and leave no error line;
- post, comment on or open the issue, even when this app has GitHub, browser or shell tools;
- state a cause as fact, or call something a b4 bug without a matching log line or changelog entry;
- put a host name, address, MAC address, device name, token or credential in the draft;
- invent a command or flag (`--sysinfo` belongs only to the installer);
- call an attachment safe to post when you have not seen it;
- say a site works on the user's device, or that a change fixed anything: a probe shows only what the router fetched;
- suggest a version that is not on the releases page;
- present tool output from another router as the user's.

## 7. Reply

Give the draft ready to paste field by field, the placeholders you used, and each attachment with where to get it and what to replace in it, from [references/attachments.md](references/attachments.md). Keep it short.
