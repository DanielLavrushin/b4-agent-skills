# Discovery results, apply and page hosts: details

Open this when a probe note, a Discovery verdict, an apply note or a page-host label says something SKILL.md does not cover. Every rule of SKILL.md still applies: consent before probes, both switches and the user's request before writes, readback, and no claim that a site works.

## The router's probe (`b4_test_domain_now`)

- It cuts any link to its host and fetches only `https://<host>/`. A deeper page that fails is tested only by passing its link whole in Discovery's `domains`.
- It sends no ECH and no QUIC. A network that drops only ClientHellos with ECH lets it load "either way" while a browser with the site's ECH key fails. Discovery probes a site that publishes an ECH key the way a browser with the key connects, and the ClientHello then names the key's public name, such as `cloudflare-ech.com`.
- Discovery does not test QUIC either.

## `b4_check_domain` rows

- `matches` holds one row per set and name; `covered_by_domain` only says per name whether any enabled set matches.
- `relation` `covers` means the set lists only a subdomain of the name, so it does not match the site itself.
- When no row has `handles: true`, the set that takes the traffic (by address, GeoIP or ASN) shows only in `b4_recent_connections`, under `ip_set`.

## A run for a set

- With `domains` empty, the run probes the set's stored Discovery addresses (`discovery.urls` in `b4_get_set`). With anything in `domains`, it probes only those, so to probe the site and the stored addresses together, list all of them, at most 5.
- Entries are kept once per host: a later entry on a host already listed is dropped without a message. When a stored address is a page on the site's own host, pass that address instead of the bare name.
- `routing.mode` names a route only when `routing.enabled` is true: an interface, a proxy, the Telegram WebSocket bridge or block. With `routing.enabled` false it is "interface" by default and means nothing.

## Status

- `finishing`: the search is done, but the run is still removing its firewall rules; apply is refused until it ends. Call status once more after a few seconds.
- Checking page hosts: the search and the confirmation pass are over, and the run tests the other hosts the confirmed pages load, for at most about two minutes. `action=cancel` now skips only that step: the run ends complete with every result kept, and apply works once status reports it finished.

## Strategy names

- `preset` names the strategy apply installs. `best_preset` and `best_kb_per_sec` can belong to another, faster strategy. On builds whose result has no `preset`, the verdict names the offered strategy; report the one the verdict names.
- A run grouped several sites under one winner: one apply creates one set for all of them; applying again for another of them writes nothing and returns that set as `applied_set` without `changed`.

## DNS-side fixes

- `alt-address`: the set pins the name to up to three addresses found during the run that complete a TLS handshake, so devices get those in DNS answers. The pins are fixed at the time of the run; a new run with the DNS check on finds new ones. **Pinned addresses** can be added or changed only in the web interface.
- `dns-redirect`: a DNS fix; it changes no packets.
- `ech-strip` (**ECH keys removed**): the set removes ECH keys from DNS answers and changes no packets. A browser with its own secure DNS still sends ECH, so it needs one of the other strategies the result lists.
- A set holding pins can be deleted or reset only in the web interface.

## Apply without `set`

- The new set is enabled and goes to position 1, ahead of the sets already there.
- It takes every name it lists out of other enabled sets that list that exact name: the run's other grouped sites, and an ECH public name the set targets as well. A geosite or parent-domain entry in another set stays where it is.
- A set limited to source devices that still matches the name can still win for its devices (`b4_get_topic` with `targets.source_devices`); `b4_recent_connections` shows which set took the traffic.
- A finished run stays in Discovery history, so a later apply works by domain without a new run. From history it carries the strategy but not the geo categories or addresses a live apply adds.
- No `changed`, with `applied_set`: the site is already covered by an existing set with the same strategy; that set is `applied_set`, and nothing new was created.

## Apply with `set`

- It writes the bypass strategy into the set: TCP, fragmentation, fakes and, when the result uses them, a DNS redirect, pinned addresses or ECH removal. The set's domains and addresses stay.
- Only for the set verdict `covered`. For `current_works` nothing is written and the result has no `applied_set`, only `set_verdict` and the note. Other statuses are refused with "nothing written".
- When the set is disabled, the note says nothing matches it until it is enabled.

## Page hosts

- A per-domain verdict names the page hosts that failed without b4 and loaded with the strategy. During status the top-level note says only that a page needs other hosts; after apply the note names them and the set to add them to. A set run's apply attaches no page-host note: take the hosts from the verdict.
- A host followed by a label in parentheses is offered only on request: a tracker; part of another service, better in a set of its own; only named in the page's code; an embedded frame; an icon; a connection hint; the page's base address; where the page redirects.
- The verdict also names hosts that stay blocked even with the strategy; adding them does not help.
- Only the page itself is read: hosts its stylesheets and scripts load in turn are not found. Such hosts may show in `b4_recent_connections` while the user opens the site. Testing one with `b4_test_domain_now` is a new probe: ask first.

## Watchdog in the web interface

The user turns on **Enable Watchdog** under Settings > Discovery and **Keep this set working with the watchdog** on the set's Discovery tab, then clicks **Save Changes**. In Russian: Watchdog = Мониторинг; Enable Watchdog = Включить мониторинг; Keep this set working with the watchdog = Поддерживать работу сета с помощью мониторинга; Settings > Discovery = Настройки > Дискавери; Save Changes = Сохранить.
