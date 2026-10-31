# Writing skills in this repo

This repo holds Monime's public agent skills: one folder per skill under `skills/`, named for the product or tool it covers (`dash-dots`, later `monime-cli`, `monime-api`).

## A skill's folder

```
skills/<name>/
  SKILL.md            # frontmatter, then the steps and rules every task needs
  agents/openai.yaml  # Codex display name and invocation policy
  references/*.md     # what only some tasks need, reached from SKILL.md by a link
```

- `SKILL.md` frontmatter has `name` (the folder name), `description`, `license: MIT` and, when the skill needs a tool or network, `compatibility`.
- The description says what the skill does, then "Use when" with one trigger per distinct case: files the project has, words the task uses, commands it runs.
- Keep `SKILL.md` under about 150 lines. Put a step in it when every task takes it, and push reference behind a link when only some tasks need it. Say in the link when to read the file.
- End each step on something the agent can check, such as a command that passes.
- Write rules as the thing to do. Keep a prohibition only for a hard limit, and pair it with what to do instead.
- Commands that act for a Space, move money or reach customers are run only when the user asks. Every skill that knows such a command says so.

## Facts come from the docs

Monime's docs at https://docs2.monime.io are the source of truth. Copy facts from them, and never from memory. Sample data uses Sierra Leone: SLE amounts, `+232` numbers, Freetown, Bo, Kenema.

Generated reference files say so in their first lines. Regenerate them, never edit them:

```bash
node scripts/sync-dash-dots-reference.mjs <path-to-monime-docs-source>
```

Then update the Dot version named in `skills/dash-dots/SKILL.md` if it changed.

## Adding a skill

1. Create its folder as above.
2. Add it to the table in `README.md` and to `skills` in `.claude-plugin/plugin.json`.
3. Raise `version` in `.claude-plugin/plugin.json`.
4. Run `claude plugin validate .` and fix what it reports.
5. Try it: give a fresh agent a real task with only the skill installed, and fix the skill where the agent went wrong.

Write in plain, short sentences, in the same voice as Monime's docs. Use no em dashes.
