# B4 Agent Skills

B4 Agent Skills is a community-maintained marketplace of reusable skills for
working on B4 projects. Skills follow the open
[Agent Skills format](https://agentskills.io/): each skill is a directory
containing a `SKILL.md` file with YAML frontmatter and instructions for an
agent.

## Browse skills

Published skills live in the [`skills/`](skills/) directory. Each directory
under `skills/` is one independently installable skill. See the skill's
`SKILL.md` for its purpose and usage.

## Install a skill

Use the [skills CLI](https://github.com/vercel-labs/skills) to list skills in
this repository and install the ones you need:

```sh
npx skills add DanielLavrushin/b4-agent-skills
```

The CLI can install skills for supported coding agents. Refer to its help for
agent-specific options.

## Add a skill

1. Create a directory under `skills/` using a short, lowercase, hyphenated
   name.
2. Add a `SKILL.md` with `name` and `description` frontmatter followed by
   concise instructions. The `name` must match the directory name.
3. Include supporting scripts or reference material in that skill's directory
   when needed, and link to them from `SKILL.md`.
4. Verify the instructions are specific to B4 work, safe to follow, and useful
   without requiring unstated context.
5. Open a pull request describing when the skill should be used and how to
   validate it.

Example:

```text
skills/
└── example-skill/
    └── SKILL.md
```

```markdown
---
name: example-skill
description: Use this skill when ...
---

# Example skill

Instructions for the agent go here.
```

There are no published skills yet. Contributions that fill this marketplace
with tested, focused B4 skills are welcome.
