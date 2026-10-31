# Monime AI skills

Agent skills for building on [Monime](https://monime.io). Give them to Claude Code, Codex, Cursor or any agent that reads skills, and it builds with Monime's products the way Monime's docs describe.

| Skill | What it does |
| --- | --- |
| [`dash-dots`](skills/dash-dots/SKILL.md) | Build, test and ship Dash Dots, mini apps that run inside Dash, with the Dot framework and the `monime dot` CLI |

More skills will follow for the Monime CLI and the Monime API.

## Install

Choose one way. Installing both gives you each skill twice.

**Claude Code plugin** (stays up to date when you update the marketplace):

```bash
claude plugin marketplace add monimesl/ai-skills
claude plugin install monime-skills@monime
```

**skills.sh** (copies the skills into your project, for Cursor, Codex, OpenCode, Windsurf and others):

```bash
npx skills@latest add monimesl/ai-skills
npx skills@latest add monimesl/ai-skills --skill dash-dots
```

**By hand:** copy `skills/dash-dots` into `.claude/skills/` (Claude Code) or `.agents/skills/` in your project, or into the same folder in your home directory for every project.

## Using a skill

Skills load themselves when the task needs them. Open a dot's folder, or ask your agent for a dot, and `dash-dots` steps in:

> Make a dot for a Freetown cookshop: a menu page, a cart, and payment with Orange Money.

The skill teaches the agent Dot's rules, where to look up every component and Dash call, and how to check its work with `monime dot build`. It asks you before it publishes or releases anything for your Space.

## Docs

Everything here comes from Monime's docs at [docs.monime.io](https://docs.monime.io). For Dash Dots, start at the [developer guide](https://docs.monime.io/docs/products/dash-dots/developers).

## License

[MIT](LICENSE)
