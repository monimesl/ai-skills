# view.dml

A page's view looks like HTML, but it's built only from Dot components (see [components.md](components.md)) and shows the page's state. The build checks every name in it and names the line that's wrong. It reports one error at a time.

```xml title="pages/index/view.dml"
<View>
  <Text variant="title">Hello, {{ name }}</Text>
  <View variant="card" direction="row" justify="between" align="center">
    <Text>{{ count }} {{ count === 1 ? 'tap' : 'taps' }}</Text>
    <Button onClick="increment" disabled="{{ count >= 5 }}">Add one</Button>
  </View>
</View>
```

## The root

Wrap the whole page in one `<View>`. It fills the screen with roomy padding, and `<View padding="none">` goes edge to edge. Put a `Footer` last to keep the main button at the bottom while the page scrolls. Comments are `<!-- ... -->`.

## Bindings

`{{ ... }}` reads the page's state, as declared in `state.ts`, and the view follows when the state changes.

- Bindings can use state names and their properties (`order.title`, `items.length`), literals, arithmetic, comparisons, `!`, `&&`, `||` and `? :`.
- Bindings can't call functions or use `?.`. Work the value out in the controller and keep it in state. Guard a value that may be `null` with `<If condition="{{ concert }}">` around the parts that read it.
- When the binding is the whole attribute, the value keeps its type, so `disabled="{{ count >= 5 }}"` is a boolean and `tone="{{ noteTone }}"` passes the string through. Mixed with words, it's text: `title="Meter {{ meter }}"`. Any non-text prop takes a plain value or one whole binding.
- A true-or-false prop on its own means true: `<Text numeric>`.

## Events

Event attributes start with `on` and name a mutation or action in the controller: `onClick="save"`, `onChange="setMeter"`. The handler gets `(state, event)`, and `event.detail` holds what the component reports (each component's **Events** table says what). The event types are global:

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
- `variant="card"` draws a card.
- Style props follow the theme: `background` (`background`, `surface`, `primary`, `muted`, `info`, `success`, `warning`, `danger`, or a custom colour from the theme), `border` (`none`, `subtle`, `strong`), `radius` (`none` to `large`, or `full`), `width`/`height` in points or `fill`, and `aspect` such as `16:9`.
- On a `primary` background, give text `tone="onPrimary"`.

`<Stack>` layers its children, the first at the bottom, and each one covers the whole stack. Place each layer's content with its own `justify`, `align` and padding. For text over a photo, put an `Image` first.

Text goes inside `<Text>`, except in components that take it directly, such as `Button` and `Badge`. Mark amounts `numeric` so their digits line up.
