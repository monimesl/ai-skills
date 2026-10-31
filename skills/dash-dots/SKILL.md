---
name: dash-dots
description: Build, test and ship Dash Dots, the mini apps that run inside Monime's Dash app, with the Dot framework and the `monime dot` CLI. Use when a project has a `dot.json` or `.dml` files, or a task mentions a dot, Dash Dots, `@monimesl/dot-runtime`, `dotctl` or `monime dot`.
license: MIT
compatibility: Needs Node.js 22.12 or newer and the Monime CLI. Reads https://docs.monime.io when the bundled reference doesn't cover a question.
---

# Dash Dots

A dot is a mini app inside Dash, Monime's money app for Sierra Leone. It belongs to a business's **Space** (its Monime account), takes payments in Dash's own payment sheet, and runs on Android and iOS from one project. You build it with **Dot**: each page is a folder with a view (`view.dml`), its starting data (`state.ts`) and a controller (`controller.ts`).

Dot is its own platform, not React or the web. A page has no DOM, CSS, `fetch`, `setTimeout` or npm UI library. What it shows is Dot components, and what it does on the phone goes through `this.dash`. Write a component, prop, event or call only once you have seen it in a source below.

## Sources of truth

Check them in this order. The first two always match the Dot version the dot builds with.

1. **The installed packages.** `node_modules/@monimesl/dot-dash/dist/index.d.ts` types every `dash` call, with `@since` for its Dash API version. `node_modules/@monimesl/dot-service/dist/page-*.d.ts` types `definePage`, `defineDot` and `this` in a controller, and `testing.d.ts` beside it types `testPage`. `@monimesl/dot-runtime` re-exports them and holds `dot.schema.json`. A newer Dot can have calls the references below don't list yet.
2. **The build.** `monime dot build` checks every component, prop, binding, handler and page id, and names the line that's wrong.
3. **This skill's references.** The long ones are indexed by `##` heading: search for the name and read that section, not the whole file.
   - [views.md](references/views.md): `view.dml` syntax, meaning bindings, events, `data-*`, `If`/`ElseIf`/`Else`, `For` and layout. Read it before your first view.
   - [components.md](references/components.md): every component's props, events and children, the shared value lists and the icon names. Read it before you write a view.
   - [dash-api.md](references/dash-api.md): every `this.dash` call with its types and errors, and the helpers in `@monimesl/dot-runtime`. Read it before you call Dash.
   - [pages-and-dot.md](references/pages-and-dot.md): `definePage`, `defineDot`, `this` in controllers, event types, the theme, `testPage` and every `dot.json` field.
   - [patterns.md](references/patterns.md): recipes for a page that loads, lists, forms, search, sheets and toasts, results from a page, and share links.
   - [payments.md](references/payments.md): taking a payment from order to server-side confirmation.
   - [testing.md](references/testing.md): testing controllers with `testPage`.
   - [cli.md](references/cli.md): every `monime dot` command, developer keys, and the register, publish and release flow.
4. **The docs** at https://docs.monime.io/docs/products/dash-dots, all in one file at https://docs.monime.io/llms-dots.txt. Monime's docs MCP server (https://docs.monime.io/mcp) has `get_dash_dots_reference_monime` for one component, one `dash.*` call or `dot.json`.

## Build loop

1. **Orient.** In an existing dot, read `dot.json`, `dot.ts` and the pages you will touch. For a new one, run `monime dot create <dir> --name "<Name>" --template starter --pm npm`, then `npm install`, and set `package` in `dot.json` to a reverse domain the business owns. The template's `"id": "<monime-Internal-Id>"` is a placeholder that `monime dot register` replaces later. The `showcase` template has a page for each feature, worth creating in a scratch folder to copy from. Done when `monime dot build` passes before you change anything.
2. **Add pages** with `monime dot create page <path>`. It writes the three files and adds the page id to `dot.ts`.
3. **Write each page** in order: the state's shape in `state.ts`, the view, then the controller's mutations, actions and hooks. Look up each component in [components.md](references/components.md) and each call in [dash-api.md](references/dash-api.md) as you use it.
4. **Check.** Run `monime dot build`, then `npm run typecheck`. The build writes `.dot/pages.d.ts` and `.dot/env.d.ts`, so typecheck only knows a new page or `.env` name after a build. Done when both pass with no errors and no warnings.
5. **Test** the controllers whose logic branches (payments, failed requests, a permission the person refused) with `testPage`, as in [testing.md](references/testing.md). Done when every branch has a test that passes.
6. **Run it.** `monime dot dev` serves the simulator at http://localhost:3680 and rebuilds on save. It runs until stopped, so start it in the background. To try it on a phone, see [cli.md](references/cli.md).

## Rules to check first

1. **Mutations or actions.** A mutation is a pure, synchronous `(state, event) => changes` that returns only the keys that change. Anything that awaits, calls `this.dash` or opens a page is an action, which changes state with `this.setState()`. Handlers take only `(state, event)` and return nothing, and `this.actions.x()` resolves to `void`. So an action hands its result on through state, and shared logic goes in plain functions, in the controller file or in a folder like `lib/` beside `pages/`. Hooks can call `this.dash` and `this.actions`, but only actions have `this.navigation`.
2. **State lives in `state.ts`.** Every page keeps all three files. The build checks the view's bindings against `state.ts`'s default export, so put the page's whole starting state there and pass it as `initialState: state`. State is plain data: use strings, numbers, booleans, `null`, arrays and objects, and keep dates as text like `'2026-10-24'`. Give a value that starts empty its type: `amount: null as number | null`. Replace values, never edit them in place, and pass a whole new object for a nested change. After an `await`, read `this.state` or pass a function to `setState`, because an action's `state` argument is from when it started.
3. **One root `<View>` per page.** Text goes in `<Text>`, except in components that take it directly, such as `Button` and `Badge`. A binding that is the whole attribute keeps its type (`disabled="{{ count >= 5 }}"` is a boolean). Bindings are plain expressions over state, with no function calls and no `?.`. Work values out in the controller, keep them in state, and guard a value that may be `null` with `<If>`. Every `<For>` gets a `key` (`key="*this"` for plain strings or numbers). Lay out with `View` props (`direction`, `gap`, `padding`, `align`, `justify`, `background`, `radius`), since there's no CSS.
4. **Handlers get `(state, event)`.** Type the event: `ValueEvent`, `ValueEvent<number>`, `CheckedEvent`, `IndexEvent`, or `DataEvent<{ id: string }>` for rows. A row says which item it is with `data-*` attributes, which arrive camelCased in `event.data` (`data-account-id` becomes `accountId`).
5. **Globals and imports.** `definePage`, `defineDot`, `env` and the event types are global, so never import them. Helpers come from the package: `import { isDashError, unlessCancelled, errorMessage, debounce, latest, DashError } from '@monimesl/dot-runtime'`.
6. **The network goes through Dash.** Call `this.dash.network.http.request<Shape>({ url, as: 'json' })`. `as: 'json'` needs the type argument, and without `as` the body is text. List every host the dot calls or loads a picture from in `dot.json` `network.hosts`, or the call fails with `denied`. A 404 or 500 still resolves, so check `response.ok`. Dash keeps no cookies, so send tokens in a header. Wait with `this.dash.system.delay(ms)`. Use `localhost` only while developing, because `monime dot build` refuses it.
7. **Every `dash` call can reject with a `DashError`.** Branch on `isDashError(error, 'denied')` and its siblings, wrap pickers and the scanner in `unlessCancelled(...)` so that closing one isn't an error, and show `errorMessage(error)` otherwise. Catch what you expect inside the action, because an uncaught error shows the person nothing.
8. **Permissions are declared.** Using `camera`, `photoLibrary`, `files`, `contacts` or `payments` needs an entry in `dot.json` `permissions`, whose value is the reason shown to the person ("Scan the QR code on your ferry ticket"). Declare only what the dot uses, and offer another way when the answer is `denied`.
9. **Navigation passes strings.** Launch options are strings, so convert numbers in `onLoad` and pass ids, not objects. Every page you open must be in `dot.ts` `pages`, and the first one is home. Leave a paid order with `replace` or `reset`, so that Back can't return to it. A share link's `share.options` is a literal string list, and `onLoad` checks what the link carries.
10. **Secrets stay on the server.** `.env` is built into the dot and anyone can read it. API keys, including the Space's Monime API key, belong on the business's server, which the dot calls. Keep the person's own session token in `this.dash.storage.secret`.
11. **Money is in minor units.** SLE 75.00 is `7500` in `dash.payments.request` (an `AmountInput` counts the same way unless it has `unit="major"`). Use a new order `reference` for each order. Treat the result as something to show, and deliver only after the business's server confirms the payment with Monime. See [payments.md](references/payments.md).
12. **Images.** Local images live in `statics/` as PNG, JPEG, GIF or WebP files of 1 MB or less (no SVG), and you reference them as `src="statics/logo.png"`. Remote images need their host in `network.hosts`.
13. **The CLI owns some files.** `.dot/` is written by the CLI, so leave it alone. `dot.json` `package` can't change after publishing, and raise `version` (semver) before every publish.

## Ask before you act for the Space

Some commands reach real people or act for the business: `monime dot register`, `publish`, `release`, `tester add` and `tester remove`, `switch-off` and `switch-on`. Run them when the user asks for that step, not as part of building. Payments in a live test on a phone move real money into the Space.
