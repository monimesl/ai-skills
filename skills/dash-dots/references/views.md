# view.dml

A page's view is much like JSX without the JavaScript. The tags are Dot components (see [components.md](components.md)), conditions and lists are tags too, and the only code is a small expression inside `{{ }}` that shows the page's state. The build checks every name in it and names the line that's wrong. It reports one error at a time.

```xml title="pages/index/view.dml"
<View>
  <Text variant="title">Hello, {{ name }}</Text>
  <View variant="card" direction="row" justify="between" align="center">
    <Text>{{ count }} {{ count === 1 ? 'tap' : 'taps' }}</Text>
    <Button onClick="increment" disabled="{{ count >= 5 }}">Add one</Button>
  </View>
</View>
```

The view runs in its own layer, apart from the controller, and the two share no memory. The view only reads state and reports taps and typing. The controller changes state and sends back only the keys that changed, which is why state must be plain data.

## The root

Wrap the whole page in one `<View>`. It fills the screen with roomy padding, and `<View padding="none">` goes edge to edge. Put a `Footer` last to keep the main button at the bottom while the page scrolls, or a `Pinned` first or last to keep anything else on the top or bottom edge. Use a `Pinned` or a `Footer` at the bottom, not both. Comments are `<!-- ... -->`.

## Bindings

`{{ ... }}` reads the page's state, as declared in `state.ts`, and the view follows when the state changes.

- A binding is one small expression. It can use state names and their properties (`order.total`, `items[0]`, `items.length`), text in quotes, numbers, `true`, `false` and `null`, math (`+ - * / % **`), comparisons (`=== !== == != < <= > >=`), logic (`! && || ??`) and the ternary (`paid ? 'Paid' : 'Due'`).
- It can't call functions, change values, use `?.`, or build lists, objects or template strings. Work the value out in the controller and keep it in state. Guard a value that may be `null` with `<If condition="{{ concert }}">` around the parts that read it.
- When the binding is the whole attribute, the value keeps its type, so `disabled="{{ count >= 5 }}"` is a boolean and `tone="{{ noteTone }}"` passes the string through. Mixed with words, it's text: `title="Meter {{ meter }}"`. Any non-text prop takes a plain value or one whole binding.
- A true-or-false prop on its own means true: `<Text numeric>`.

## Events

Event attributes start with `on` and name a mutation or action in the controller: `onClick="save"`, `onChange="setMeter"`. The handler gets `(state, event)`, and `event.detail` holds what the component reports (each component's **Events** table says what). Anything that can be tapped dims under the finger, with no props to set. The event types are global:

| Type | For |
| --- | --- |
| `ValueEvent` | A field's text, in `detail.value`. `ValueEvent<number>` for `Slider` and `Stepper`, `ValueEvent<number \| null>` for `AmountInput` |
| `CheckedEvent` | `detail.checked`, from `Switch` and `Checkbox` |
| `IndexEvent` | `detail.index`, from `Carousel` and `Scroll` |
| `DataEvent<Data>` | Any event, with its `data-*` attributes in `event.data` |

## Data attributes

When many rows run the same handler, `data-*` attributes say which was tapped. They arrive camelCased, so `data-account-id` is `accountId`:

```xml
<For each="{{ accounts }}" item="account" key="id">
  <ListItem title="{{ account.name }}" data-account-id="{{ account.id }}" onTap="open" />
</For>
```

```ts
open: (_state, event: DataEvent<{ accountId: string }>) => ({ selected: event.data.accountId }),
```

## Conditions

```xml
<If condition="{{ status === 'paid' }}">
  <Notice tone="success" title="Paid">Your token is on its way.</Notice>
</If>
<ElseIf condition="{{ status === 'pending' }}">
  <Spinner />
</ElseIf>
<Else>
  <Button onClick="pay">Pay</Button>
</Else>
```

`ElseIf` (as many as you need) and `Else` come straight after an `If` or `ElseIf`. None of the three take events, so put handlers on what's inside.

## Hide or lock a View

An `<If>` removes what's inside, so it starts afresh when it comes back. To keep it alive, set `hidden` on a `View`: what's typed stays, and so does a `Scroll`'s position. A `Sheet` inside a hidden `View` stays shut.

```xml
<View hidden="{{ tab !== 'details' }}" gap="medium">
  <Input label="Your name" value="{{ name }}" onChange="setName" />
</View>
```

`disabled` dims a `View`, and nothing in it takes a tap. Screen readers skip it, so say why outside it:

```xml
<Text>The waiting list opens on Friday.</Text>
<View disabled="{{ !listOpen }}" variant="card" gap="medium">
  <Input label="Your phone number" placeholder="+23276123456" />
  <Button>Join the waiting list</Button>
</View>
```

`ListItem`, `GridItem`, `Link`, `Tab`, `Disclosure` and `Option` take `disabled` too. A tap on something disabled stops there, so it doesn't reach a tappable `View` around it.

## Lists

```xml
<For each="{{ orders }}" item="order" index="i" key="id">
  <Text>{{ i + 1 }}. {{ order.title }}</Text>
</For>
```

`item` and `index` name each item and its position. `key` names a property unique to each item, or `key="*this"` for a list of plain strings or numbers. A `<For>` without `key` is a build warning.

## Layout

There's no CSS. A `View` lays out its children:

- They stack in a column. `direction="row"` puts them side by side, and `wrap` lets a row flow onto new lines.
- `gap` and `padding` take `none`, `small`, `medium` or `large`. `paddingTop`, `paddingBottom`, `paddingStart` and `paddingEnd` pad one side.
- `justify` places children along the direction and `align` across it. `grow` takes the room left over.
- `variant="card"` draws a card. The theme can add variants of its own (see [theming.md](theming.md)).
- Style props follow the theme:
  - `background`: `background`, `surface`, `primary`, `muted`, `info`, `success`, `warning`, `danger`, or a custom colour or gradient from the theme.
  - `border` (`none`, `subtle`, `strong`) and `borderTone`, the outline's colour, such as `primary` or a custom colour.
  - `radius` (`none` to `large`, or `full`), and `corners` (`all`, `top`, `bottom`) for which corners it rounds.
  - `elevation`, a shadow: `none`, `low`, or `high` for what floats over everything else.
  - `opacity`: `full`, `soft` or `faint`, to let what's behind show through.
  - `enter`: `none`, `fade` or `rise`, how it comes in when an `<If>` shows it or `hidden` turns off. Phones set to reduce motion skip it.
  - `width` and `height` in points or `fill`, `minHeight`, `maxWidth`, and `aspect` such as `16:9`.
- On a `primary` background, give text `tone="onPrimary"`.

`<Stack>` layers its children, the first at the bottom, and each one covers the whole stack. Place each layer's content with its own `justify`, `align` and padding. Everything a `<For>` or `<If>` shows in a `Stack` is its own layer. For text over a photo, put an `Image` first.

Text goes inside `<Text>`, except in components that take it directly, such as `Button` and `Badge`. Mark amounts `numeric` so their digits line up, and use `caps` for a small label in spaced capitals.

## Make a picture of a View

Give a `View` an `id`, and `this.captureView(id)` in an action makes a PNG of it, such as a ticket to share. It draws the whole `View`, even the part scrolled out of sight, and gives back a `DotFile` to show, share, upload or save.

```xml
<View id="ticket" variant="card" gap="small">
  <Text variant="title">Lungi ferry</Text>
  <QRCode value="{{ ticketCode }}" size="160" />
</View>
<Button onClick="shareTicket">Share ticket</Button>
```

```ts
async shareTicket() {
  const picture = await this.captureView('ticket');
  await this.dash.system.share({ file: picture });
},
```

- An `id` takes letters, numbers, `-` and `_`, and each `View` on a page needs its own.
- A new picture of a `View` replaces the last one, and older file objects of it show the new picture. Save it with `dash.storage.file.save` to keep it.
- An `Image` in the `View` must load within 15 seconds, and one from another site needs that server to allow CORS.
- It rejects if the page has no such `View`, if the `View` isn't showing or is too big, or if an `Image` won't load.
