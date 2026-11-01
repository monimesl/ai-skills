# Monime AI skills

Agent skills for building on [Monime](https://monime.io). Give them to Codex, Cursor, Claude Code or any agent that reads skills, and it builds with Monime's products the way Monime's docs describe.

| Skill | What it does |
| --- | --- |
| [`dash-dots`](skills/dash-dots/SKILL.md) | Build, test and ship Dash Dots, mini apps that run inside Dash, with the Dot framework and the `monime dot` CLI |

More skills will follow for the Monime CLI and the Monime API.

## Install

The skills use the open [Agent Skills](https://agentskills.io) format: a `SKILL.md` and plain Markdown references, with nothing tied to one agent. Choose one way to install them. Installing more than one gives you each skill twice.

**With skills.sh**, for Codex, Cursor, Claude Code, OpenCode, Windsurf and others. It copies the skills into the folders your agents read:

```bash
npx skills@latest add monimesl/ai-skills
```

To add one skill only, name it: `npx skills@latest add monimesl/ai-skills --skill dash-dots`.

**By hand:** copy `skills/dash-dots` into `.agents/skills/` in your project, or into your agent's own skills folder, such as `.claude/skills/` for Claude Code. To have it in every project, use the same folder in your home directory.

**As a Claude Code plugin**, which stays up to date when you update the marketplace:

```bash
claude plugin marketplace add monimesl/ai-skills
claude plugin install monime-skills@monime
```

## Using a skill

Skills load themselves when the task needs them. Open a dot's folder, or ask your agent for a dot, and `dash-dots` steps in:

> Make a dot for a Freetown cookshop: a menu page, a cart, and payment with Orange Money.

The skill teaches the agent Dot's rules, where to look up every component and Dash call, and how to check its work with `monime dot build`. It asks you before it publishes or releases anything for your Space.

## Docs

Everything here comes from Monime's docs at [docs2.monime.io](https://docs2.monime.io). For Dash Dots, start at the [developer guide](https://docs2.monime.io/docs/products/dash-dots/developers).

## License

[MIT](LICENSE)
