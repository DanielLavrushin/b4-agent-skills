# Connection log, device and port details

Open this file when a row, a device's address or a port list does not fit the rules in SKILL.md, or to tell the user where in the web interface to read rows or make a change. Russian labels for the terms used only here: Devices = Устройства; Detail = Детали; All packets = Все пакеты; All = Всё; Manual Devices = Ручные устройства; Log Level = Уровень логирования; Targets = Цели; Bypass Domains = Домены обхода; Bypass IPs = IP обхода; General = Основные; Port Filter = Фильтр портов; Calls = Звонки; Pass call setup = Пропускать начало звонков; Handle only calls = Обрабатывать только звонки. The TCP and UDP tabs have the same names in both languages.

## The Traffic page

- To find a device: set the period to **All**, since the **Devices** panel lists only the devices seen within the selected period. A device b4 knows a name for shows by that name only, with no address. The **Detail** pane of one of its rows shows its IP and MAC, and hovering over the device in a row shows the MAC.
- To read rows when the log's window is too short: select the device under **Devices**, turn on **All packets**, set the period to **All**, use the app, and read the rows to the AI. The page collects for as long as it stays open.

## Where the user makes a change

- Domains and addresses: the set's **Targets** tab, under **Bypass Domains** or **Bypass IPs**.
- Ports: **Port Filter** on the set's TCP tab under **General**, or on its UDP tab.
- Calls: **Calls** on the set's UDP tab, with **Pass call setup** or **Handle only calls**.
- Device filtering: Settings > Core > Devices. **Enable IPv6 Support**: Settings > Core > Packet Engine.

## The `mac` column

- It is empty when b4 has no address-to-MAC mapping: for a device behind another router, for the router's own traffic, and usually for an IPv6 source. b4 takes MACs from the router's IPv4 ARP table and from **Manual Devices**.
- A device added under **Manual Devices** shows a synthetic MAC starting `02:B4:`, built from its IP. It is not the device's real MAC.
- Many phones use a randomized MAC, and a new MAC is a new device for b4.
- A MAC written with dashes (`aa-bb-cc-dd-ee-ff`) appears in the log as `AA:BB:CC:DD:EE:FF`. The filter ignores case.

## Addresses in the filter

- The trailing colon in `<device IPv4>:` stops addresses that only start the same way, not longer addresses that end the same way, and the same text can sit in the `destination` column. That is why every row needs the exact `source` check.
- The log writes IPv6 in brackets in the compressed lower-case form, which can differ from the form the device shows (leading zeros, the long form). A device may use more than one IPv6 address. If `[<address>]:` matches nothing, ask for the device's other IPv6 addresses before concluding that a host was not seen.
- The `time` column is the router's clock: use it to keep the rows from the moment the user acted.

## Action tokens with a trailing part

- `escalate->` and `dns-escalate->`: the name after the arrow is the set that takes over. On a TCP `escalate->` row, `sni_set` (or `ip_set` for an address match) still shows the previous set.
- `mtproto:` followed by the name of an MTProto secret, which is often a person's name: never quote that part. Rows of b4's MTProto web proxy have protocol `WEB` and an action starting `mtproto`.
- `dns-doh->` followed by a DoH upstream, `dns-forward->` followed by a resolver address: quote only the part before the arrow.
- `routed->`, `tunnel->`, `reroute->` followed by an interface: quote only the part before the arrow.
- `dns-hint`: the row's `domain` came from an earlier DNS answer for that address, not from the packet, so the app's TLS name may differ.
- A routing set's `direct` means the connection went out directly.

## DNS rows (destination port 53)

- A query b4 acted on for a set whose targets match the name, or an answer b4 changed: `dns-heal`, `dns-ipv6-stripped`, `dns-heal+ipv6-stripped`, `dns-ech-stripped`, `dns-servfail`.
- On a `dns-ech-stripped` row, `sni_set` may name a set that matches only the ECH name inside the answer, not the queried name in `domain`.
- Names that no set involves never show, so a missing DNS row proves nothing. DNS rows appear only while **Log Level** (Settings > System > Service) is Info or more detailed.
- With **Enable IPv6 Support** off, b4 in NFQUEUE mode still removes ECH keys from IPv6 DNS answers for the sets that ask for it, so such a DNS row can appear while no IPv6 connection rows do.

## Call rows

- `voice-exempt` marks a call packet that passed untouched: through a set whose **Calls** is **Pass call setup**, or as a later packet of a call that a set with **Handle only calls** took.
- A set with **Handle only calls** took the call when its own row for the call's first packet names it, in `ip_set`, with no action. Its later packets show the same set in `ip_set` with `voice-exempt`. It matched by port and payload, not by address.

## Ports

- b4's queue takes TCP 443 (and 80 while an enabled set has `tcp.http_methodeol` on), UDP 443 and 53, and the ports that enabled sets list. No topic covers `tcp.http_methodeol`.
- In a set that has targets, a port list narrows the set to the listed ports.
- A set without targets and with a port list acts on all traffic to those ports, except a set that handles only calls (`udp.filter_l7`), which acts only on the call packets it selects on those ports and lets the rest pass untouched.
- Either kind applies to every device unless its source devices limit it.
- `b4_list_writable_paths` with `prefix="sets[].tcp.dport_filter"` and `set` naming the set also shows a set's current list. Without `set` it shows the first set's list. No `current` means the list is empty.

## Device filtering by version

- From b4 1.88.0, a device added under **Manual Devices** is matched by its IP address; older versions ignored such devices.
- From 1.88.0, a change to device filtering applies when the settings are saved; before that, after a restart of b4.
- Whether a device is selected is in the full `queue.devices` list, which holds every device's name, MAC and IP. Asking the user is the default. Read the list only after telling the user that and getting their consent, and report only this device's entry.

## `b4_edit_set_targets` result fields

- `rewritten`: entries b4 normalised, such as a dropped `*.` or a CIDR with its host bits cleared.
- `expansion`: `geosite_categories_matching_nothing`, `geoip_categories_matching_nothing` and `unresolved_asns`.
- `moved_from_other_sets`: domains taken out of other enabled sets.
- With `changed: false` nothing is recorded for undo.
