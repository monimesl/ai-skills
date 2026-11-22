# Theming

A dot has no CSS. Its look comes from the theme in `dot.ts`, the props in its views, and a page's own colours. Exact types are under `## Theme` in [pages-and-dot.md](pages-and-dot.md). Source: https://docs2.monime.io/docs/products/dash-dots/developers/theming

## The theme

```ts title="dot.ts"
export default defineDot({
  pages: ['index'],
  theme: {
    scheme: 'system',
    colors: {
      primary: { light: '#0168CC', dark: '#6CB4FF' },
      background: { light: '#F4F7FB', dark: '#0B0F14' },
    },
    shape: 'soft',
  },
});
```

- `scheme`: `system` (the default) follows the phone's light or dark mode. `light` or `dark` picks one.
- `shape`: how round corners are, `square`, `subtle`, `soft` (the default) or `rounded`.
- `colors`, each with a `light` and `dark` value in `#RRGGBB`: `primary` (primary buttons, links, chosen chips, switches), `background` (the page), `surface` (cards, lists, sheets, fields), `text`, `muted` (quiet fills such as secondary buttons), `border`, `subtle` (captions and hints), and the tones `info`, `success`, `warning` and `danger`.

Set as few colours as you can: many dots set only `primary`. Dash works out the rest from your `background`, and picks dark or light `text` when you leave it out. `monime dot dev` warns about a text colour or tone that's hard to read, and about any text on a page that can barely be read on what's behind it. Where `primary` wouldn't read, outline and ghost buttons, links and tabs use the text colour.

## Custom colours and gradients

```ts title="dot.ts"
theme: {
  colors: {
    custom: {
      gold: { light: '#F5C542', dark: '#D4A72C' },
      ink: { light: '#1C1C1E', dark: '#1C1C1E' },
    },
  },
  gradients: {
    goldenHour: { from: 'gold', to: 'warning', direction: 'diagonal' },
  },
},
```

```xml
<View background="goldenHour" radius="large" padding="large">
  <Text variant="title" tone="ink">Salone Sounds Live</Text>
</View>
```

- A custom colour works as a `View`'s `background` or `borderTone`, a `Text`'s `tone` and an `Icon`'s `tone`. Name it in camelCase, don't reuse a built-in name like `surface`, and check it in light and dark mode.
- A gradient is only a `View`'s `background`. Each end is a theme colour, a custom colour, or `transparent` to fade out, such as behind text on a picture. `direction` is `down` (the default), `right` or `diagonal`. Pick text that reads on both ends.

## Fonts and text size

A dot uses the phone's own fonts, and can't add its own.

```ts title="dot.ts"
theme: { font: 'rounded', headingFont: 'serif', headingWeight: 'medium', textSize: 'large' },
```

- `font`: `system` (the default), `rounded`, `serif` or `mono`. A phone with no rounded font, like most Android phones, uses its system font.
- `headingFont` and `headingWeight` (`regular`, `medium`, `bold`) set apart a `Text`'s `display`, `title` and `subtitle`, and the titles of sheets and empty states. A `Text`'s own `weight` wins.
- `textSize`: `small`, `regular` (the default) or `large`, a tenth smaller or larger. Typed text, amounts, codes and small count pills keep their size.

## Spacing and shadows

```ts title="dot.ts"
theme: { density: 'comfortable', elevation: 'raised' },
```

- `density` scales the gaps and padding a view names, and a card's own padding: `compact` is three quarters of `regular` (the default), `comfortable` a quarter more. Spacing inside components stays.
- `elevation: 'raised'` gives every card and list a soft shadow in place of its hairline. A `View`'s own `elevation` and `border` win, and a card with `elevation="none"` keeps its hairline.

## Component defaults and your own variants

```ts title="dot.ts"
theme: {
  components: {
    Button: { size: 'large' },
    View: {
      variants: {
        card: { elevation: 'low', border: 'none' },
        ticket: { variant: 'card', background: 'muted', borderTone: 'gold' },
      },
    },
  },
},
```

```xml
<View variant="ticket">
  <Text>Salone Sounds Live</Text>
</View>
```

- A default applies to every use of a component, or to one of its variants. A prop in the markup wins, and a variant's default wins over the component's.
- Defaults set props that choose a look, like `size`, `tone` or `padding`, never what a component shows, like `icon` or `status`. While you develop, the dot won't start with a default the component doesn't take, and the error says what it does take.
- `View`, `Text` and `Button` can have variants of your own, named in camelCase. Set `variant` inside one to start from a variant the component already has. Dot warns about a variant that doesn't exist.

## Your own icons

```ts title="dot.ts"
theme: { icons: { 'ticket-stub': 'statics/icons/ticket-stub.svg' } },
```

```xml
<Icon name="ticket-stub" tone="success" />
<Chip label="VIP" icon="ticket-stub" />
```

- Use the name wherever Dot's icons go. It's drawn in one colour, its `tone`, whatever colours the file has.
- Name it in lowercase words joined by dashes, and don't reuse the name of one of Dot's icons (the list is under `### Icon names` in [components.md](components.md)).
- The SVG must hold only shapes, in UTF-8, of 1 MB or less. The build refuses one with scripts, stylesheets or links. Export it from the design tool as plain shapes.

## A page's own colours

A page can set any theme colour, tones included, in `configs.colors` in `definePage`. The colours it leaves out stay as the theme has them.

```ts
configs: { colors: { background: { light: '#FFF3E0', dark: '#2B1A0E' } } },
```

While it runs, an action changes them with `this.setColors({ background: { light: '#E8F5E9', dark: '#0F2414' } })`. Give both modes, or the other keeps its colour. `{ light: undefined, dark: undefined }` goes back to the theme's.

To change the whole theme as the dot starts, such as dark mode the person chose earlier, call `this.setTheme({ ...this.theme, scheme: 'dark' })` in `onLaunch` in `dot.ts`. It replaces the whole theme, so spread the current one in.
