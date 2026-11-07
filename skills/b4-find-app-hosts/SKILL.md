---
name: b4-find-app-hosts
description: Step-by-step procedure for finding which hosts and protocols an app on one device uses - a TV's YouTube app, Discord voice, a game, a phone app - whether b4, the DPI-bypass and routing service on the user's router, sees that traffic, which set takes it, and what to add to a set, using b4's MCP tools. Load it before answering whenever the user says one app or one device fails while the rest of the network works, asks which domains, addresses or ports an app uses, asks what to add to a set for an app, or reports YouTube failing on a TV or phone, Discord voice not connecting or a game that cannot reach its servers - even when the app's usual domains or a geosite category look like the obvious answer. Russian requests too - "какие домены использует приложение", "ютуб не работает на телевизоре", "не подключается голос в дискорде", "игра не видит серверы", "что добавить в сет". Needs b4's MCP tools.
compatibility: Needs a connection to b4's MCP server (the b4_* tools). Changes need "Allow configuration changes" and a test fetch needs "Allow active probes" in b4's web interface.
---

# Finding the hosts an app uses

b4 is a DPI-bypass and traffic-routing service on the user's router. Open [references/connection-log.md](references/connection-log.md) for rows, addresses and port lists the rules below do not settle, and for where the user reads rows or makes a change. Reply in the user's language. When you reply in Russian, use the labels of b4's Russian interface: Settings > Integrations > MCP server = Настройки > Интеграции > MCP-сервер; Allow configuration changes = Разрешить изменение настроек; Allow active probes = Разрешить активные проверки; System Info under Settings > System > Service = Информация о системе в Настройки > Система > Сервис; Copy JSON = Копировать JSON; Import/Export = Импорт/Экспорт; Discovery = Дискавери; Traffic = Трафик; Settings > Core > Devices = Настройки > Основные > Устройства; Settings > Core > Packet Engine = Настройки > Основные > Движок пакетов; Enable IPv6 Support = Поддержка IPv6. Sets = Сеты (a top-level page, not under Settings); Logs = Логи (a top-level page with Start trace = Начать трассировку); Log Level = Уровень логирования (Settings > System > Service); Update available = Доступно обновление.

## 0. Check the tools first

Apps may load MCP tools only on demand or put the server name in front, such as `mcp__b4__b4_status`; this skill uses the bare names. Look for a tool whose name ends in `b4_status`, use the app's tool search, or simply call `b4_status`. Only if none of that works, tell the user this chat has no working b4 MCP connection (not set up, or failed to start); do not guess at b4's state. Point them to the b4-connect skill if installed and to https://docs.b4core.app/docs/settings/mcp/ (Russian: https://docs.b4core.app/ru/docs/settings/mcp/).

## 1. Privacy

The log and the device list cover every device on the network, and tool output may reach the model's provider. Filter every `b4_recent_connections` call by the device, except the consented call in section 2. `contains` matches anywhere in a line, so keep only rows whose `source` address (before the port) or `mac` equals the device's, and drop the rest unread. Read `queue.devices`, `queue` or the whole config only with consent: each holds every device's name, MAC and IP. Report nothing about other devices.

## 2. Identify the device

Ask for the device's IP address, and its MAC if known (its network settings, the router's client list, or b4's **Traffic** page). The `mac` column is often empty, so the address is the main filter. If the user cannot find it, ask before an unfiltered call and say it sends other devices' connections to the AI provider. Only with consent: have them use the app, call `b4_recent_connections` with `limit` 20 and no filter, and show only the distinct `source` and `mac` values, never hosts.

## 3. Capture the app's traffic

1. `b4_status` (version, `engine`, switches), `b4_list_sets`, and `b4_get_config` with `section="queue.ipv6"`.
2. Have the user use the app now (start a video, join a call, start a match), then call `b4_recent_connections` with `contains` set to `<device IPv4>:` and `limit` 200. With `queue.ipv6` true, also ask for the device's IPv6 address and query `[<address>]:`; IPv6 rows usually have no MAC. If the device has a MAC in the log, query it too, written with colons.
3. The log holds the last 500 events of all devices, often under a minute: query right after the user acts, and again while they continue. On `truncated`, query again sooner instead of raising `limit`. Empty means nothing in that window.
4. If the window stays too short, have the user read you the rows on the **Traffic** page (reference file).

## 4. Read each row literally

- `protocol` is `TCP`, `UDP` or `WEB` (b4's MTProto web proxy); UDP with `1.3` in `tls` is QUIC.
- `domain` can be a name b4 learned earlier for the address, not one the packet carried (often with action `dns-hint` on TCP).
- `sni_set` took the connection by name; `ip_set` without one: by address, learned address, port list or call detection. With both filled, `sni_set` handled it. Never say a set targets an address unless `b4_get_set` shows it.
- A routing set's rows (action `proxy`, `direct`, `mtproto-ws`) name it in `sni_set` whatever matched.
- Port-53 rows are DNS queries or answers b4 acted on for a set, never a list of all the app looked up.
- The action is a raw token for what b4 did (`block`, `voice-exempt`), never a device label. After `escalate->` or `dns-escalate->` comes the set that takes over: report that set. For other tokens with `->` or `:` (an MTProto secret's name, a DNS upstream, an interface), quote only the part before it. `voice-exempt` alone does not show a set took a call.
- Group rows by protocol and domain, else by destination.

## 5. Keep three outcomes apart

For each host or address, say which holds:

1. **Taken by a set**: `sni_set` or `ip_set` names it on a connection row; a port-53 row alone is the lookup, not the connection.
2. **Seen, no set**: both empty; section 8 proposes targets.
3. **Not seen**: no row. Before saying b4 never saw it, get a fresh capture and check:
   - ports: topics `tcp.dport_filter` and `udp.dport_filter` say which ports reach b4; a set with routing on takes traffic its own way (topic `routing.mode`; the b4-proxy-routing skill, if installed);
   - IPv6: with `queue.ipv6` false, b4 handles no IPv6 connections, so IPv6 hosts never show;
   - device filtering: `b4_get_config` with `section="queue.devices.enabled"` and `section="queue.devices.wisb"`. With `enabled` true only selected devices get DPI bypass, or all but them with `wisb` true; none selected means all, so `enabled` true alone never shows that a device is left out. Ask the user whether their device is selected under **Settings > Core > Devices**. A device left out may show no connection rows, though DNS rows can appear;
   - TUN mode (`engine` is `tun`): some of the device's IPv4 rows can carry a router address and no MAC, so the section 1 check drops them; say a host may be hidden this way.

If the device shows no rows at all, follow the b4-troubleshoot skill if installed.

## 6. Coverage of each name and address

- `b4_check_domain` with the names, comma-separated, at most 32 per call (on `truncated`, check the rest). Read `relation`, `via` and `enabled`. `handles` ignores the device: for a set with source devices, trust `sni_set` in this device's rows.
- `b4_get_set` for each set involved, with topics `targets.source_devices` and `targets.source_devices_exclude`: a set may apply to some devices only, or to all but them.
- `b4_geo_lookup` with `action="find_domain"` and `domain` lists the geosite categories holding a name, and `used_by_set`; `action="preview"` with `category` shows what one, often broad, would add. An unknown category matches nothing. A nameless address: `action="lookup_ip"` with `ip`.

## 7. Calls, voice and games

Before saying anything about UDP or ports, read `b4_get_topic` for `udp.dport_filter`, `udp.filter_stun`, `udp.filter_quic`, `udp.filter_l7`, `udp.mode` and `tcp.dport_filter`. For Discord voice, follow https://docs.b4core.app/docs/guides/discord-voice/ (Russian: https://docs.b4core.app/ru/docs/guides/discord-voice/): a set without targets that handles only calls, plus Discord's TCP ports on its domain set. A port list narrows a set with targets but makes a set without targets act on those ports (reference file): keep 443 in a TCP list and tell the user who a change affects.

## 8. Propose additions as text

For each host, name the set (existing or new) and the target, and say which hosts stay uncovered. When `b4_check_domain` showed a name already held by another enabled set, say so in the proposal: adding it with `b4_edit_set_targets` (or the web interface's add-to-set action) removes it from every other enabled set; typing it into the set editor does not.

- `sni_domains`: a bare name covers every subdomain; there is no wildcard syntax. Prefer the parent domain the rows share.
- `geosite_categories`: only one that `find_domain` or `preview` showed holding these hosts.
- `ip` or `asns` for nameless addresses; an ASN covers every range a network announces, unrelated services on a cloud or CDN included.
- Ports and QUIC handling only after section 7.

## 9. Applying, only when asked

Only when the user asks, and only if `b4_status` reports `you_can_change_settings` (**Allow configuration changes** under Settings > Integrations > MCP server; the AI app must reconnect after it is turned on). Otherwise give the change as text and where to make it.

- New set: `b4_manage_set` with `action=create` and `name`; it lands last in match priority unless `position` says otherwise (`first`, `before:<set>`, `after:<set>`), and the first enabled match wins, so tell the user where it landed. Then add its targets.
- Targets: `b4_edit_set_targets` with `set`, `kind` and `add`. A domain added here leaves every other enabled set. Tell the user what `rewritten`, `moved_from_other_sets`, `expansion` and `note` report, and read the set back with `b4_get_set`.
- Ports: read the set's own list with `b4_get_set` first (`b4_list_writable_paths` without `set` shows the first set's), then call `b4_set_config_value` with `path="sets[<set id or name>].tcp.dport_filter"` (or `.udp.dport_filter`) and `value` holding the old entries plus the new: the list is replaced wholesale, and invalid entries vanish silently. Compare the reported `current` with what you meant.
- Undo: `changed: false` means nothing was kept or recorded; do not revert, that would undo an earlier change. If `changed` is true but the result is wrong (`current`, an unwanted move), call `b4_revert_last_change` and say so. It undoes the last MCP change since b4 last started, one per call, never web-interface edits, and is refused when anything else changed the configuration since.
- Never over MCP: device filtering, **Enable IPv6 Support** (Settings > Core > Packet Engine), credentials, the web server, the MCP settings, the capture engine, the firewall backend and packet marks. Send the user to the web interface.

## 10. Check the result

Have the user retry the app and read the device's rows again: its hosts should now show the set. That shows the set took the traffic, not that the app works; only the retry shows that. With consent, and only if `b4_status` reports `you_can_test_and_discover` (**Allow active probes**), `b4_test_domain_now` fetches `https://<name>/` from the router: it says nothing about the device, the app, UDP, QUIC or other ports, and a set limited to source devices (not an exclude list) never applies to it. If the app still fails with its hosts in a set, the strategy is next: the b4-troubleshoot skill, if installed.

## 11. When to stop and report

- the device shows no rows at all while the app is in use;
- a needed port is not captured, or device filtering leaves the device out: the user decides;
- you need data only the user has: the device's address, the **Traffic** rows, whether the device is selected, or the set's JSON (**Copy JSON** on its **Import/Export** tab).

## 12. Never claim

- why an app is slow or failing: the log shows which set took a flow, not the cause; say what you saw and what would test it;
- that you can change a setting when the write tools (`b4_set_config_value`, `b4_edit_set_targets`, `b4_manage_set`) are not in your tool list, whatever `b4_status` reports;
- that the app will work after targets are added;
- that b4 never saw a host without a fresh capture and the section 5 checks;
- what a host does beyond its name, unless called a guess;
- a port, default or setting's meaning without `b4_get_topic`, beyond this skill's facts for `queue.ipv6`, `queue.devices` and `tcp.http_methodeol`, which have no topic; for other settings without one, say you are unsure;
- anything about another device's traffic;
- that you changed or tested anything you did not.

## 13. Reply

One line per host: protocol, name or address, the set that took it ("none" or "not seen" otherwise), and the proposed target. Then the change, where to make it, and how the user checks it. Keep it short.
