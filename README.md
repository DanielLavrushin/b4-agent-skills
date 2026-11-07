# b4 agent skills

[Русская версия](README_ru.md)

Skills for AI assistants that help you check and adjust [b4](https://github.com/DanielLavrushin/b4) on your router through b4's built-in MCP server. They are for people who run b4. To work on b4's code, see the [b4 repository](https://github.com/DanielLavrushin/b4).

A skill is a set of instructions in the open [Agent Skills](https://agentskills.io/) format. The assistant loads it when your request matches, and follows it while it uses b4's tools.

## Skills

| Skill | What it does |
| --- | --- |
| [b4-connect](skills/b4-connect/SKILL.md) | Connects your AI app to b4's MCP server, and fixes the connection when b4's tools are missing: the entry for each app, token mistakes, certificate errors, a switch that needs a reconnect, apps that cannot reach a home router. Works without b4's tools. |
| [b4-troubleshoot](skills/b4-troubleshoot/SKILL.md) | Finds out why a site, an app or b4 itself is not working: a site that does not open, Discovery that finds nothing, Telegram or YouTube trouble, a capability shown as unavailable, something that broke after an update, a clash with firmware features or another proxy. |
| [b4-unblock-site](skills/b4-unblock-site/SKILL.md) | Makes a blocked site work: confirms the block with b4's own test, runs Discovery with your go-ahead, applies a result you approve, tests it again and says where the new set landed and how to undo it. Stops when the router already loads the site. |
| [b4-proxy-routing](skills/b4-proxy-routing/SKILL.md) | Sets up or fixes sets that send sites or devices through a VPN interface or a SOCKS5 upstream, and works out which set takes a connection: traffic that goes direct, proxied connections without a site name, "everything except these sites", DNS that breaks after a routing set is turned on. |
| [b4-find-app-hosts](skills/b4-find-app-hosts/SKILL.md) | Finds which hosts and protocols an app on one device uses (a TV's YouTube app, Discord voice, a game), whether b4 sees that traffic and which set takes it, and what to add to a set. Reports only the device you name. |
| [b4-report-issue](skills/b4-report-issue/SKILL.md) | Writes a short, checked bug report for b4's GitHub issues from facts read through b4's tools, with the router's host name, addresses and other private data left out. Checks first whether a newer release already fixes the problem. |

All of them tell the assistant to:

- read b4's state first, and look up every setting in b4's own setting descriptions instead of guessing from its name;
- compare the installed b4 version with the [latest release](https://github.com/DanielLavrushin/b4/releases), and use the [changelog](https://github.com/DanielLavrushin/b4/blob/main/changelog.md) only for what a version fixed;
- say a site loads only after b4's own test of that site succeeded;
- test a site, run Discovery or change a setting only when you ask and the matching switch in b4 is on, read the saved value back after a change, and use b4's undo for its own last change;
- keep other devices' traffic and the router's private data out of answers and reports;
- ask what else runs on the router when the evidence points outside b4, instead of guessing.

## Before you start

The skills need b4's MCP server connected to your AI app. Setting it up takes a few minutes: [Connecting an AI assistant](https://docs.b4core.app/docs/settings/mcp/). If the app does not show b4's tools, ask it to connect to b4: the b4-connect skill handles that case.

By default the assistant can only read. Two switches under **Settings > Integrations > MCP server** in b4's web interface allow more:

- **Allow configuration changes**: the assistant may change settings and sets.
- **Allow active probes**: the assistant may test whether a site loads from the router and run Discovery.

After turning a switch on, reconnect the AI app to the MCP server so it gets the new tools.

## Which AI apps

| App | Skills | Notes |
| --- | --- | --- |
| Claude Code | yes | Tested. Install with the skills CLI or as a plugin. |
| ChatGPT desktop app | yes | Tested. The app reads this repository's plugin marketplace and lists the `b4` plugin with its skills. It runs MCP servers from its local configuration, which it shares with Codex. |
| Antigravity CLI | yes | Tested: the plugin loads and its skills run. Its connection to b4 needs b4 1.90.0 or later; earlier versions reject one of its protocol messages ([b4 issue #533](https://github.com/DanielLavrushin/b4/issues/533)). |
| Claude Desktop | yes | The Code tab works like Claude Code. In Chat, the plugin installs under **Customize > Plugins**, and b4 is reached through a local bridge such as [mcp-remote](https://github.com/geelen/mcp-remote) in the app's configuration file, which passes the token as an `Authorization` header; connectors added in Settings connect from Anthropic's cloud and cannot reach a router on a home network. Not tested. |
| Codex CLI, Gemini CLI, Cursor, VS Code with GitHub Copilot | yes | These apps load Agent Skills; install with the skills CLI. Not tested with b4. |
| claude.ai | no | Its connectors connect from Anthropic's cloud and cannot reach a router on a home network. |
| ChatGPT on the web | no | Custom MCP connections are made from OpenAI's cloud and need a public HTTPS address or OpenAI's tunnel, so a router on a home network is not reachable. |
| LM Studio, Jan | no | They do not load skills. The text of a SKILL.md can go into the system prompt or the assistant's instructions. In testing, a small local model (llama3.2, 3B) could not complete these tasks with or without it. |
| Ollama | no | Ollama on its own does not connect to MCP servers. |
| b4's built-in AI explanations | no | They explain settings in the web interface and do not use b4's tools. |

## Install

### With the skills CLI

Needs Node.js. Run it in the folder you open in your AI app, or add `-g` to install for your user:

```sh
npx skills add DanielLavrushin/b4-agent-skills
```

The command asks which skills and which apps to install for. To answer up front, name the app, for example:

```sh
npx skills add DanielLavrushin/b4-agent-skills --skill '*' -a claude-code -y
```

Other app names: `codex`, `gemini-cli`, `cursor`, `github-copilot`.

### As a Claude Code plugin

In Claude Code:

```text
/plugin marketplace add DanielLavrushin/b4-agent-skills
/plugin install b4@b4-agent-skills
```

Or from a terminal:

```sh
claude plugin marketplace add DanielLavrushin/b4-agent-skills
claude plugin install b4@b4-agent-skills
```

### In the ChatGPT desktop app

Add the marketplace with the Codex CLI, restart the ChatGPT desktop app, then open **Plugins**, choose the `b4-agent-skills` source and install `b4`:

```sh
codex plugin marketplace add DanielLavrushin/b4-agent-skills
```

## Using the skills

Ask in your own words, for example:

- "discord.com stopped opening after the b4 update"
- "make meduza.io work through b4"
- "route youtube.com through my wg0 interface"
- "which servers does Discord on my PC use, and does b4 handle them?"
- "help me write a bug report for b4"
- "my AI app does not show b4's tools"

The assistant loads the matching skill by itself. To call one directly in Claude Code, type its name with a slash, for example `/b4-troubleshoot` (with the plugin: `/b4:b4-troubleshoot`).

If the problem is still not solved, the assistant lists what to attach to a report: see [Reporting a problem](https://docs.b4core.app/docs/troubleshooting/reporting/).

## Contributing

Each skill is a folder under [`skills/`](skills/) with a `SKILL.md` file and, when it needs one, a `references/` folder for detail it opens on demand. The frontmatter uses only the keys of the [Agent Skills specification](https://agentskills.io/specification) (`name`, `description`, optional `compatibility`), and `name` matches the folder name. A skill states nothing about a b4 setting that b4's own setting descriptions or source code do not back up, and it never promises that a site will work.

The plugin has two manifests with the same name, version and description: `.claude-plugin/plugin.json` for Claude Code, and the portable `plugin.json` at the root ([Agent Plugins](https://agent-plugins.org/) schema), which ChatGPT, Codex and Antigravity read and which carries the ChatGPT logo and website. Change both together. The plugin icon is `.claude-plugin/icon.png` and `assets/logo.png`.
