---
name: b4-unblock-site
description: Step-by-step procedure for making a site that the ISP's DPI blocks open through b4, the DPI-bypass service on the user's router, with b4's MCP tools - confirm the block, search for a working strategy with Discovery, apply it as a set, verify it and clean up. Load it before acting whenever the user asks b4 to unblock, fix or make work a specific site or service, asks to run Discovery or find a strategy, asks to apply a Discovery result, or reports that Discovery found nothing, said no bypass is needed, or found a strategy that stopped working - even when it looks like a single tool call. Russian requests too - "разблокируй сайт", "сделай, чтобы открывался", "подбери стратегию", "запусти Дискавери", "Дискавери ничего не находит", "обход не нужен", "стратегия перестала работать". Needs b4's MCP tools with active probes and configuration changes allowed.
compatibility: Needs a connection to b4's MCP server (the b4_* tools). Testing a site and Discovery need "Allow active probes"; applying a result needs that and "Allow configuration changes"; editing or deleting sets and undo need "Allow configuration changes", plus "Allow active probes" when the change would start a watchdog. Both switches are in b4's web interface. A Discovery run takes minutes and sends real traffic from the router.
---

# Unblocking a site with b4

b4 is a DPI-bypass and traffic-routing service on the user's router. Discovery, the tool `b4_find_bypass_strategy`, tries b4's strategies on a site until one loads it, and turns the winner into a set. If the failure is not a block on the router's path, hand back to the b4-troubleshoot skill if it is installed. Open [references/results-and-apply.md](references/results-and-apply.md) when a verdict, note or label says something this page does not cover. Reply in the user's language; in Russian, use b4's Russian interface labels: Settings > Integrations > MCP server = Настройки > Интеграции > MCP-сервер; Allow configuration changes = Разрешить изменение настроек; Allow active probes = Разрешить активные проверки; Discovery = Дискавери; Start = Начать; Apply as a set = Применить как сет; Create a new set = Создать новый сет; Create set = Создать сет; Write the strategy into an existing set = Записать стратегию в существующий сет; Replace strategy = Заменить стратегию; Save Changes = Сохранить; TLS version for the probes = Версия TLS для проверок; Pinned addresses = Закреплённые адреса; ECH keys removed = Ключи ECH убраны; Enable Watchdog = Включить мониторинг. A set is "сет". Sets = Сеты (a top-level page, not under Settings); Logs = Логи (a top-level page with Start trace = Начать трассировку); Log Level = Уровень логирования (Settings > System > Service); Update available = Доступно обновление.

## 0. Check the tools first

Apps may load MCP tools only on demand, or prefix them with the server's name (`mcp__b4__b4_status`, or `mcp_` and the server's name in VS Code). Look for a tool whose name ends in `b4_status`, use the app's tool search, or simply call `b4_status`; the same goes for every tool below. Only if none of that works, say this chat has no working b4 MCP connection, suggest the b4-connect skill if it is installed, and point to https://docs.b4core.app/docs/settings/mcp/ (Russian: https://docs.b4core.app/ru/docs/settings/mcp/). Do not guess at b4's state.

## 1. Switches and the go-ahead

1. `b4_status`: `you_can_test_and_discover` is **Allow active probes**, `you_can_change_settings` is **Allow configuration changes**, both under Settings > Integrations > MCP server in b4's web interface, not the AI app. Tests and Discovery need the first; apply needs both; set edits and undo need the second, plus the first if they would start a watchdog. After the user turns one on and clicks **Save Changes**, the AI app must reconnect to see the new tools.
2. Tests and runs send real traffic from the router; a run takes minutes, holds firewall rules and keeps the watchdog from healing. Say so and get one go-ahead for the two-way test, the run and step 6's probes. Ask before probing any other host, and again before applying.
3. If a switch is off, say which and where, and stop at the step that needs it. On the **Discovery** page the user can enter the site, click **Start**, then **Apply as a set** on a card with a strategy, then **Create a new set** and **Create set**, or **Write the strategy into an existing set** and **Replace strategy** for the set already taking the site.

## 2. Confirm the block

1. `b4_check_domain` with the bare host (no scheme or path). In `matches`, the enabled row with `handles: true` is the set that takes the name; only it applies. It reads names only, not IP, GeoIP or ASN targets.
2. `b4_test_domain_now` with the host, `mode` at `both`. It fetches only `https://<host>/` from the router, without ECH or QUIC, unlike a browser. By its `note`:
   - loads either way, or censored but loads through b4: the router loads it, so no strategy will help. Do not start Discovery or change any set, even with the user's go-ahead. The cause is on the device side: check step 6.3, then hand over to b4-troubleshoot if it is installed. Only if the user names a specific page or link that fails, test that exact URL first.
   - loads with b4 bypassed but fails through b4: a b4 setting breaks it. No Discovery; hand back to b4-troubleshoot.
   - fails both ways: run Discovery, for the `handles` set if any. Without one the results may compare nothing (a set can still take the site by address).
   - DNS_FAKE: a forged DNS answer; Discovery checks DNS first.
   - a short reply or off-site redirect: the probe cannot tell. Test the host the note names (ask first), or pass a failing page's link whole to Discovery.
3. Never target a private or local address; a non-ASCII name goes in as punycode (`xn--`).
4. `b4_test_domain_now` fetches from the router itself, so its rows in `b4_recent_connections` carry the router's own address, not a device's: a public source address, or the router's own LAN address, in rows for the tested host right after your test, is that test. Never read such rows as the user's device or its DNS.

## 3. Run Discovery

1. With a `handles` set, pass `set`: the run tests its strategy first and apply writes into it, changing it for every site of the set; say so. Empty `domains` probes the set's stored Discovery addresses: name them when asking for the go-ahead (adding the site: see the reference). Without a set, pass `domains`; apply creates a new set.
2. A set with `routing.enabled` true is refused: its traffic is routed, not on the direct path Discovery tests (VPN or proxy routes: b4-proxy-routing).
3. `action=start`, `domains` comma-separated, at most 5: bare names or the failing link whole. Leave `skip_dns` unset: the DNS check finds forged answers.
4. One run at a time, shared with watchdog heals; if one is running, check it with `action=status`.
5. After start, tell the user it is running. If you can wait (the app gives you a wait or sleep tool, or the user asked you to carry on until it finishes), wait about a minute between `action=status` calls, never call it back to back, and give up after about ten checks. If you cannot wait, stop there and call `action=status` once, when the user asks. When status says `finishing`, or that it is still checking page hosts (up to about two minutes), one more call after a short wait is expected; `action=cancel` then skips only the page-hosts step and keeps the results. Mid-run, a listed strategy is unconfirmed and apply is refused.

## 4. Read the result

Per entry in `domains`, read `verdict`, then the top-level `note`.

- Report the strategy the verdict names (`preset` where present), not `best_preset`, which apply may not install.
- `unconfirmed`: loaded once, never confirmed; apply only if the user accepts that.
- the link answers HTTP N even without b4: nothing was tested, not "nothing found". Ask for the link of a page or file the site serves.
- `alt-address`, `dns-redirect`, `ech-strip` (**ECH keys removed**): DNS-side fixes that change no packets, so a pass from the router's probe proves nothing for them; rely on step 6.3. Ask whether the devices use the router's DNS and, for `ech-strip`, whether the browser has its own secure DNS (it then still sends ECH).
- `works_without_b4` (no set needed; apply refuses it), `transport_blocked` (only a proxy or VPN route helps), `gateway_intercepted`, an unresolved name, or nothing worked: pass on the verdict and stop; start no runs unasked. After nothing worked, the docs say a run with **TLS version for the probes** at TLS 1.2 (web interface only) sometimes helps, else a VPN or proxy. **Pinned addresses** are web-interface only.
- a set run: `set_verdict.status` and `meaning`. Apply writes only for `covered`; `current_works`: nothing to write (device still failing: step 6.3, b4-troubleshoot); other statuses: report and stop.

## 5. Apply, only when the user asks

1. First say what it will do. `action=apply` with `domain` (`id` = start's `suite_id`, optional `name`) creates an enabled set at position 1, ahead of the others, moving each name it lists out of other enabled sets listing that exact name. With `set`, it writes only the bypass strategy into that set.
2. Read `changed`, `applied_set`, `moved_from_other_sets` and `note`. Tell the user the set's name and position, the strategy the note names, each moved name with the set it left, and how to remove the set. No `changed`: nothing was written; report the note (an `applied_set` then is an existing set). After an error, check `b4_list_sets` before saying nothing changed.

## 6. Verify

1. After the last write, `b4_test_domain_now` with `mode=through_b4`, three separate calls. All three must load; any failure means it does not work from the router.
2. Failing after a confirmed verdict: report both results, trust the probe, offer undo. DNS-side fixes: step 4.
3. Ask the user to open the site, then `b4_recent_connections` with `contains` = the host. `sni_set` (or `ip_set`) names the set that handled it; another set means the new strategy has no effect. It keeps 500 events: empty means none in that window. The device result is the user's report, not a test.

## 7. Page hosts

Apply does not add the page's other hosts. Offer only those the per-domain `verdict`, or the apply `note`, lists without a label in parentheses, via `b4_edit_set_targets` (`set` = the set applied to or named in the note, `kind=sni_domains`, `add` = the hosts). Labelled hosts only on request. Adding moves a domain out of every other enabled set; report what moved, then repeat step 6.

## 8. Readback, undo and clean up

- After every write, compare what the tool reports back with what you meant; say when they differ (`changed: false`: b4 did not keep it).
- `b4_revert_last_change` restores the configuration from before the latest MCP change, moved names included; again goes one more back. It does not undo web-interface edits or changes from before b4 last started, and is refused when the watchdog, the web interface or another tool changed the configuration since, or when it would start a watchdog with **Allow active probes** off. Read its note.
- Remove a test set with `b4_manage_set` (`action=delete`, `set`, `confirm_name` = its current name). Refused for a set with DNS pins or upstream credentials: the user deletes it in the web interface. Deleting does not return moved names; offer to add them back.
- Credentials, the web server, MCP settings, the capture engine, the firewall backend and packet marks never change over MCP: send the user to the web interface.
- Before explaining a setting, call `b4_get_topic` with its dotted key; never infer a default or unit from a name.

## 9. Later: the watchdog

For a site that works and later stops, mention the watchdog; turn it on only on request. `b4_watchdog` with `set` and `action=enable` turns on that set's watchdog, which then rewrites the set's strategy by itself when checks keep failing. `enabled` in its result is the global **Enable Watchdog** switch: while false, nothing is checked. `action=enable` without `set` turns it on for every watched set and the global domain list, also only on request. Both calls need both switches. The web-interface path is in the reference.

## 10. Never claim

- that you can change a setting when the write tools (`b4_set_config_value`, `b4_edit_set_targets`, `b4_manage_set`) are not in your tool list, whatever `b4_status` reports;
- that the site works without step 6's three loads after the last change, and for a DNS-side fix not even then; never promise it keeps working;
- that nothing is blocked because the router's probe loads the site;
- that a verdict, an apply or a `b4_check_domain` match makes it work;
- a start, apply, addition or deletion you did not make;
- a strategy other than the one the verdict or apply note names.

## 11. Reply

Briefly report the two-way test result, what Discovery found and whether it was confirmed, what was written where, the three verification results, how to undo, and the next step if it still fails.
