# b4 agent skills

[Русская версия](README_ru.md)

Skills for AI assistants that help you check and adjust [b4](https://github.com/DanielLavrushin/b4) on your router through b4's built-in MCP server. They are for people who run b4. To work on b4's code, see the [b4 repository](https://github.com/DanielLavrushin/b4).

A skill is a set of instructions in the open [Agent Skills](https://agentskills.io/) format. The assistant loads it when your request matches, and follows it while it uses b4's tools.

## Skills

| Skill | What it does |
| --- | --- |
| [b4-troubleshoot](skills/b4-troubleshoot/SKILL.md) | Finds out why a site, an app or b4 itself is not working: a site that does not open, Discovery that finds nothing, Telegram or YouTube trouble, a capability shown as unavailable, something that broke after an update, a clash with firmware features or another proxy. |
| [b4-proxy-routing](skills/b4-proxy-routing/SKILL.md) | Sets up or fixes sets that send sites or devices through a VPN interface or a SOCKS5 upstream, and works out which set takes a connection: traffic that goes direct, proxied connections without a site name, "everything except these sites", DNS that breaks after a routing set is turned on. |

Both skills tell the assistant to:

- read b4's state first: status, diagnostics, the error log, the sets, the recent connections;
- look up every setting in b4's own setting descriptions instead of guessing from its name;
- compare the installed b4 version with the [changelog](https://github.com/DanielLavrushin/b4/blob/main/changelog.md) and point out a fix in a newer release;
- say a site loads only after b4's own test of that site succeeded;
- test a site or change a setting only when you ask and the matching switch in b4 is on, read the saved value back after a change, and use b4's undo for its own last change;
- ask what else runs on the router when the evidence points outside b4, instead of guessing.

## Before you start

The skills need b4's MCP server connected to your AI app. Setting it up takes a few minutes: [Connecting an AI assistant](https://docs.b4core.app/docs/settings/mcp/).

By default the assistant can only read. Two switches under **Settings > Integrations > MCP server** in b4's web interface allow more:

- **Allow configuration changes**: the assistant may change settings and sets.
- **Allow active probes**: the assistant may test whether a site loads from the router and run Discovery.

After turning a switch on, reconnect the AI app to the MCP server so it gets the new tools.

## Which AI apps

| App | Skills | Notes |
| --- | --- | --- |
| Claude Code | yes | Tested. Install with the skills CLI or as a plugin. |
| Claude Desktop | yes | The Code tab works like Claude Code. In Chat, the plugin installs under **Customize > Plugins**, and b4 is reached through a local bridge such as [mcp-remote](https://github.com/geelen/mcp-remote) in the app's configuration file, which passes the token as an `Authorization` header; connectors added in Settings connect from Anthropic's cloud and cannot reach a router on a home network. Not tested. |
| claude.ai | no | Its connectors connect from Anthropic's cloud and cannot reach a router on a home network. |
| Codex CLI, Gemini CLI, Cursor, VS Code with GitHub Copilot | yes | These apps load Agent Skills; install with the skills CLI. Not tested with b4. |
| ChatGPT | no | Custom MCP connections are made from OpenAI's cloud and need a public HTTPS address or OpenAI's tunnel, so a router on a home network is not reachable. |
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

## Using the skills

Ask in your own words, for example "discord.com stopped opening after the b4 update" or "route youtube.com through my wg0 interface". The assistant loads the matching skill by itself. To call one directly in Claude Code, type `/b4-troubleshoot` or `/b4-proxy-routing` (with the plugin: `/b4:b4-troubleshoot`, `/b4:b4-proxy-routing`).

If the problem is still not solved, the assistant lists what to attach to a report: see [Reporting a problem](https://docs.b4core.app/docs/troubleshooting/reporting/).

## Contributing

Each skill is a folder under [`skills/`](skills/) with a `SKILL.md` file. Its frontmatter uses only the keys of the [Agent Skills specification](https://agentskills.io/specification) (`name`, `description`, optional `compatibility`), and `name` matches the folder name. A skill states nothing about a b4 setting that b4's own setting descriptions or source code do not back up, and it never promises that a site will work.
