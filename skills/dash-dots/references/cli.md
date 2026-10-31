# The `monime dot` CLI

Run every command from the dot's folder. `monime dot <command> --help` is the final word on flags. Inside a dot, the CLI uses the Dot tools the dot depends on, and the project's npm scripts call the same tools as `dotctl`.

## Set up

```bash
curl -fsSL https://cli.monime.io/install.sh | sh   # macOS and Linux
irm https://cli.monime.io/install.ps1 | iex        # Windows PowerShell
monime login                                       # opens a browser
monime space use <alias-or-id>                     # the business the dot belongs to
monime dot doctor                                  # checks login, Space, Node.js, Dot tools, dot.json, key
```

`monime login` needs the user at a browser. In CI, set `MONIME_ACCESS_TOKEN` instead. `-q` turns off every prompt.

## Develop

| Command | What it does |
| --- | --- |
| `monime dot create [DIR] --name NAME --template starter\|showcase --pm npm\|pnpm\|yarn\|bun` | New dot. With all flags it asks nothing, and `package` defaults to `com.example.<dir>`, which must change before `register` |
| `monime dot create page PATH` | Adds `pages/PATH/` with its three files and lists it in `dot.ts` |
| `monime dot dev [--port PORT] [--device]` | Simulator on port 3680 (or the next free one), rebuilding on save. Runs until stopped |
| `monime dot build` | Checks and builds into `dist/`. Errors stop it, and warnings (a `<For>` without `key`) don't |
| `monime dot preview` | Runs the release build in the simulator, without rebuilding |
| `monime dot lsp --stdio` | The `.dml` language server for editors |
| `monime dot update [--check]` | Updates the Dot tools. Keep `@monimesl/dot-runtime` and `@monimesl/dot-cli` on the same version |

The simulator shows the dot in a phone frame with Dash's bars and menu. Its payment sheet is a stand-in, files live in `.dot/files/` and secrets in `.dot/.secrets/`.

## Run on a phone

```bash
monime dot keys create        # once per computer: registers a developer key to the Monime account
monime dot dev --device       # prints a QR code
```

On the phone, Dash > My Account > Preferences > Developer settings > **Enable dot debugging** (once), then **Live test** from the Dots tab's menu, and scan. The phone and computer share a network, and the phone is signed in to the account the key belongs to. `console` output from the phone shows in the terminal. Payments here are real.

`monime dot keys list`, `keys revoke KEY_ID`, and `keys register` (for a key this computer has but the account lost).

## Ship

These act for the Space and reach real people. Run each one when the user asks for it.

| Command | What it does |
| --- | --- |
| `monime dot register` | Creates the dot in the Space once, and writes its `id` into `dot.json`. Commit that |
| `monime dot publish [--subtitle TEXT] [--description TEXT] [--wait]` | Builds and uploads a new version. The first publish needs a subtitle (64 characters at most) and a description (3,000). `--wait` fails if the version is rejected |
| `monime dot version list [--state STATE]` / `version describe VERSION` | Where each version stands |
| `monime dot tester add\|remove @DASHTAG`, `tester list` | Testers get each version once it's `ready_to_test` |
| `monime dot release [VERSION] [--to testers]` | Gives everyone the testers' version, or a named one. Releasing an older version rolls back |
| `monime dot switch-off [VERSION] --note TEXT` / `switch-on [VERSION]` | Stops Dash opening the dot (or one version) and refuses its payments, within a minute |
| `monime dot list`, `monime dot describe` | The Space's dots, and this dot's listing and state |

A version moves `checking` → `in_review` (sometimes) → `ready_to_test` → `released`, or ends `rejected` with a note. Later ones mark it `superseded`, and `switched_off` means Dash won't open it.

Before each publish, raise `version` in `dot.json`: patch for a fix, minor for something new, major for a removed feature or a redesign. Publishing fails when the version isn't higher than the last, when another Space owns the `package`, when `package` or `name` differs from the registered dot, or when there's no subtitle yet.

## CI

```bash
export MONIME_ACCESS_TOKEN="$MONIME_TOKEN"   # from the CI secret store
monime dot publish --space "$MONIME_SPACE_ID" --subtitle "Top up your meter in seconds" --wait
```
