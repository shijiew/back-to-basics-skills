# Using This Repository With Cursor

This repository includes a **Cursor project rule** that makes the Back to Basics guidance available for relevant agent-system architecture tasks.

## In This Repository

1. Open the repository folder in Cursor.
2. Cursor IDE Agent and Cursor CLI automatically read root [`AGENTS.md`](AGENTS.md) and [`CLAUDE.md`](CLAUDE.md).
3. Cursor also discovers [`.cursor/rules/back-to-basics.mdc`](.cursor/rules/back-to-basics.mdc).
4. The `.mdc` rule uses `alwaysApply: false`, so it applies only when Cursor judges it relevant or it is explicitly included.
5. Because this distribution repository contains every supported format, relevant Cursor requests may receive equivalent guidance from more than one file. In a target project, install only one mechanism.
6. Confirm project rules under **Settings → Rules** or the project-rules interface.

## Use the Same Guidance in Another Project

**Cursor (recommended):** Copy the project rule into the target repository:

```bash
mkdir -p .cursor/rules
curl -o .cursor/rules/back-to-basics.mdc \
  https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/.cursor/rules/back-to-basics.mdc
```

**AGENTS.md-compatible tools:** Copy [`AGENTS.md`](AGENTS.md) into the project root or merge it with existing instructions.

**Claude Code:** Install the plugin using the [`README.md`](README.md) instructions, or copy [`CLAUDE.md`](CLAUDE.md) into the project root.

## Optional Personal Agent Skill

To install the guidance as a reusable personal Cursor skill, copy [`skills/back-to-basics/SKILL.md`](skills/back-to-basics/SKILL.md) into your personal skills directory using the same layout as your other skills.

```text
~/.cursor/skills/
└── back-to-basics/
    └── SKILL.md
```

## Claude Code and Cursor

- **Claude Code:** The plugin exposes the skill from this repository. `CLAUDE.md` provides an always-on per-project alternative. Use one.
- **Cursor IDE Agent and CLI:** Root `AGENTS.md` and `CLAUDE.md` are always applied. The `.mdc` rule provides selective project-level loading.
- Cursor project rules apply to Agent Chat, not Tab completion, Inline Edit, or Bugbot reviews.
- Do not install more than one equivalent instruction mechanism unless duplicate guidance is intentional.

See the official [Cursor rules reference](https://cursor.com/docs/rules.md) and [CLI rules documentation](https://cursor.com/docs/cli/using.md#rules).

## For Contributors

[`skills/back-to-basics/SKILL.md`](skills/back-to-basics/SKILL.md) is the source of truth.

When the guidance changes:

1. Update `SKILL.md`.
2. Synchronize [`AGENTS.md`](AGENTS.md), [`CLAUDE.md`](CLAUDE.md), and [`.cursor/rules/back-to-basics.mdc`](.cursor/rules/back-to-basics.mdc).
3. Keep the routing description aligned in the skill frontmatter, Cursor rule, and Claude plugin metadata.
4. Re-run the relevant behavioral probes before publishing.
