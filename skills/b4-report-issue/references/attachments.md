# Attachments: where to get each one and what to replace

Open this before telling the user how to get an attachment or what to replace in it. For a Russian reply, use the labels line in SKILL.md. Steps: https://docs.b4core.app/docs/troubleshooting/reporting/ (Russian: https://docs.b4core.app/ru/docs/troubleshooting/reporting/).

## Where to get each

- **System Info**: **System Info** under Settings > System > Service, then **Copy JSON** in the **System Diagnostics** dialog. It goes in the required **System diagnostics (JSON)** field.
- **Debug log trace**: note the current **Log Level** under Settings > System > Service, set it to **Debug** and click **Save Changes**; on the **Logs** page click **Start trace**, reproduce the problem, click **Stop & save** (a trace stops by itself after 15 minutes); set **Log Level** back to the noted value (**Info** by default), not to Info out of habit.
- **The set**: **Copy JSON** on the set's **Import/Export** tab.
- **The configuration**, only when the developer asks: **Download safe copy** under Settings > System > Backup.
- **When the web interface does not open**: the installer's `--sysinfo` output and the last lines of `errors.log` (SKILL.md section 0).

## What to replace in each

The user replaces each value with the placeholder the draft uses for it.

### System Info (the same report as `b4_diagnostics`)

- `system.hostname`
- `network.interfaces[].addrs` and `network.interfaces[].mac`
- the lines under `firewall.rule_groups`: their set entries carry LAN device addresses and ports
- `upstreams[].upstream` (the proxy address)
- the addresses under `engine.tun`
- every error string: `upstreams[].last_error`, `engine.start_error`, `firewall.dns_query_copy.error`, `firewall.routing.error`, `firewall.routing.set_errors[].error`, `geodata.asn.referenced[].last_error`. A connection error names the router's address and port and the remote address and port.

### Debug log trace, top to bottom

1. `--- system diagnostics ---`: the System Info report; the same fields and error strings as above.
2. `--- enabled sets (<n>) ---`: for each set, up to eight domains (`targets.domain_sample`); `dns.target_dns` and `dns.doh_url` (a personal DoH URL carries an account or resolver id); for a routed set, `routing.upstream` (the proxy host and port), `routing.egress_ip` and `routing.egress_gateway`. No proxy username or password is written (`routing.upstream_auth` is only true or false), and `targets.source_devices` and `dns.pins` are only counts here.
3. The log lines: the sites devices open, with device IP and MAC addresses, and error messages that can name the hosts listed under "Quoted `errors.log` and `update.log` lines".

### Set JSON (Copy JSON on Import/Export)

- the domains and addresses in `targets.sni_domains` and `targets.ip`
- `dns.doh_url`, `dns.target_dns`, and `dns.pins` (domains with their addresses, exported even when the set's DNS is off)
- `routing.egress_ip` and `routing.egress_gateway`
- `discovery.urls` (custom check URLs, which can carry credentials)
- for a proxy set, `routing.upstream.host`, `routing.upstream.port`, `routing.upstream.username` and `routing.upstream.password`. Always have the user remove the username and password: `b4_get_set` hides them, Copy JSON does not.

Device MAC addresses are not in it: the export drops `targets.source_devices`.

### Quoted `errors.log` and `update.log` lines

- addresses and ports inside the messages;
- host names of the user's own proxies, relays, Workers and update mirrors. A Telegram bridge failure lists every transport it tried, a custom WS domain or a Worker (`wsworker://<host>`) included. `update.log` holds the installer's output, which prints `Mirror: <url>` for each mirror it tries, the user's own mirrors first.

### Safe copy (Download safe copy)

Passwords, tokens, secrets, personal DoH URLs and the host names of the router and of the user's own relays and mirrors are masked. Still present: the sets' domains and addresses (`targets.ip`, `dns.target_dns`, `dns.pins`, `routing.egress_ip`, `routing.egress_gateway`), proxy addresses, device names and MAC addresses, interfaces and ports.

### Installer `--sysinfo` output

- the host name.

## What the developer asked for in #444

The reporter's "likely cause" did not hold when the developer rebuilt the setup. He asked for:

- a few Logs page lines for the failing connection, including any ending in `[tunnel->...]` or `[reroute->...]`;
- the set's JSON from its Import/Export tab;
- Copy JSON from System Info;
- the watchdog lines around "heal failed".

The watchdog's `set "<name>": heal failed` lines are warnings: they are on the Logs page and in a trace, not in `errors.log`. A `firewall refresh after heal failed` line in `errors.log` is a different event, an ERROR, not the heal failure.
