# Page patterns

Recipes for the pages most dots need. Each one passes `monime dot build` and `npm run typecheck` once its page exists, its starting state is in `state.ts` and `.env` has `API_URL`. Look up any component or call in [components.md](components.md) and [dash-api.md](dash-api.md).

## A page that loads from your server

Load in `onLoad`, show a skeleton while loading, and offer a retry when it fails. `load` is an action, so the retry button can call it too.

```ts title="pages/meter/state.ts"
export default {
  meter: '',
  balance: 0,
  amount: null as number | null,
  loading: false,
  error: '',
};
```

```ts title="pages/meter/controller.ts"
import state from './state';

export default definePage({
  initialState: state,
  hooks: {
    async onLoad(options) {
      this.setState({ meter: options.meter ?? '' });
      await this.actions.load();
    },
    async onRefresh() {
      await this.actions.load();
    },
  },
  mutations: {
    setAmount: (_state, event: ValueEvent<number | null>) => ({ amount: event.detail.value }),
  },
  actions: {
    async load(state) {
      this.setState({ loading: true, error: '' });
      try {
        const response = await this.dash.network.http.request<{ balance: number }>({
          url: `${env.API_URL}/meters/${state.meter}`,
          as: 'json',
        });
        if (!response.ok) {
          throw new Error();
        }
        this.setState({ balance: response.body.balance });
      } catch {
        this.setState({ error: "We couldn't load your meter." });
      } finally {
        this.setState({ loading: false });
      }
    },
    confirm(state) {
      this.navigation.push('confirm', { meter: state.meter, amount: String(state.amount) });
    },
  },
});
```

```xml title="pages/meter/view.dml"
<View>
  <If condition="{{ loading }}">
    <Skeleton shape="text" lines="2" />
  </If>
  <ElseIf condition="{{ error }}">
    <Notice tone="danger" title="Not loaded">{{ error }}</Notice>
    <Button variant="secondary" onClick="load">Try again</Button>
  </ElseIf>
  <Else>
    <Text variant="display" numeric>{{ balance }} kWh</Text>
    <AmountInput label="Top-up amount" value="{{ amount }}" onChange="setAmount" />
    <Footer>
      <Button onClick="confirm" disabled="{{ !amount }}">Top up</Button>
    </Footer>
  </Else>
</View>
```

- `onRefresh` makes the page pull-to-refresh. Without it the page can't be pulled.
- `onShow` runs when the page comes back to the top, which is the moment to refresh data another page changed.
- `env.API_URL` comes from `.env`. Put the address there, and override it in `.env.development` or `.env.production`.

## A list the person taps

`data-*` attributes tell the handler which row was tapped. `onEndReached` loads more.

```xml
<List onEndReached="loadMore">
  <For each="{{ payments }}" item="payment" key="id">
    <ListItem
      icon="receipt"
      title="{{ payment.name }}"
      value="SLE {{ payment.amount }}"
      data-id="{{ payment.id }}"
      onTap="open"
    />
  </For>
</List>
<If condition="{{ !loading && payments.length === 0 }}">
  <EmptyState icon="receipt" title="No payments yet" text="Payments you make show here." />
</If>
```

```ts
actions: {
  open(_state, event: DataEvent<{ id: string }>) {
    this.navigation.push('payment', { id: event.data.id });
  },
  async loadMore(state) {
    if (state.loading || state.allLoaded) {
      return;
    }
    this.setState({ loading: true });
    try {
      const url = `${env.API_URL}/payments?skip=${state.payments.length}`;
      const response = await this.dash.network.http.request<Payment[]>({ url, as: 'json' });
      if (response.ok) {
        this.setState({
          payments: [...this.state.payments, ...response.body],
          allLoaded: response.body.length === 0,
        });
      }
    } finally {
      this.setState({ loading: false });
    }
  },
},
```

`loadMore` spreads `this.state.payments`, not `state.payments`, because the list may have changed during the `await`.

## A form

A field shows what's in state and reports changes. Validate in the action, and show the problem with the field's `error`.

```xml
<Input label="Phone number" type="phone" prefix="+232" value="{{ phone }}" error="{{ phoneError }}" onChange="setPhone" />
<Select label="Network" value="{{ network }}" onChange="setNetwork">
  <Option value="orange" label="Orange" />
  <Option value="africell" label="Africell" />
</Select>
<Switch label="Save this number" checked="{{ save }}" onChange="setSave" />
<Button onClick="send" loading="{{ sending }}">Send airtime</Button>
```

```ts
mutations: {
  setPhone: (_state, event: ValueEvent) => ({ phone: event.detail.value, phoneError: '' }),
  setNetwork: (_state, event: ValueEvent) => ({ network: event.detail.value }),
  setSave: (_state, event: CheckedEvent) => ({ save: event.detail.checked }),
},
actions: {
  async send(state) {
    if (state.phone.replace(/\D/g, '').length !== 8) {
      this.setState({ phoneError: 'Enter 8 digits, such as 76 123456.' });
      return;
    }
    this.setState({ sending: true });
    // ...
  },
},
```

- `AmountInput` sends `null` when empty, so type its event `ValueEvent<number | null>`.
- `CodeInput` sends `complete` once the last digit is in. Check the code in that handler.
- `Tabs` only record the chosen tab. Show each tab's content with `<If>`.

## Search as the person types

```ts title="pages/search/state.ts"
export type Shop = { id: string; name: string };

export default { results: [] as Shop[] };
```

```ts title="pages/search/controller.ts"
import { debounce, latest, unlessCancelled } from '@monimesl/dot-runtime';
import state, { type Shop } from './state';

const typingPause = debounce(400);
const search = latest();

export default definePage({
  initialState: state,
  actions: {
    async find(_state, event: ValueEvent) {
      if (!(await typingPause(this.dash))) {
        return;
      }
      const url = `${env.API_URL}/shops?q=${encodeURIComponent(event.detail.value)}`;
      const answer = await unlessCancelled(search(this.dash.network.http.request<Shop[]>({ url, as: 'json' })));
      if (answer?.ok) {
        this.setState({ results: answer.body });
      }
    },
  },
});
```

```xml title="pages/search/view.dml"
<View>
  <Input type="search" placeholder="Find a shop" onChange="find" />
  <For each="{{ results }}" item="shop" key="id">
    <Text>{{ shop.name }}</Text>
  </For>
</View>
```

## Sheets and toasts

Both follow state. A `Sheet` shows while `open` is true and sends `close` when tapped outside. A `Toast` shows its `message` and sends `hide`, where you clear the message so it can show again.

```xml
<Sheet open="{{ selected !== null }}" title="Payment" onClose="closeSheet">
  <Button variant="secondary" onClick="copyReference">Copy reference</Button>
</Sheet>
<Toast message="{{ toast }}" tone="success" onHide="clearToast" />
```

```ts
mutations: {
  closeSheet: () => ({ selected: null }),
  clearToast: () => ({ toast: '' }),
},
actions: {
  async copyReference(state) {
    await this.dash.system.copyText(state.selected ?? '');
    this.setState({ selected: null, toast: 'Copied' });
  },
},
```

## Ask the person, then act

```ts
const { action } = await this.dash.ui.alert({
  title: 'Cancel this order?',
  actions: [
    { id: 'keep', label: 'Keep order', role: 'cancel' },
    { id: 'cancel', label: 'Cancel order', role: 'destructive' },
  ],
});
if (action === 'cancel') {
  // ...
}
```

A dot can open at most 6 dialogs (alerts, pickers, the scanner, share, dial, the payment sheet) in any 30 seconds, and none while Dash is in the background.

## A picker that may be refused

```ts
import { isDashError, unlessCancelled } from '@monimesl/dot-runtime';

async scan() {
  try {
    const code = await unlessCancelled(this.dash.media.scanCode());
    if (code) {
      this.setState({ meter: code.text });
    }
  } catch (error) {
    if (isDashError(error, 'denied', 'unsupported')) {
      this.setState({ typeMeter: true }); // Let them type it instead.
      return;
    }
    throw error;
  }
},
```

`scanCode` needs `"camera"` in `dot.json` `permissions`.

## Sign the person in

```ts
import { errorMessage, isDashError } from '@monimesl/dot-runtime';

async signIn() {
  try {
    const person = await this.dash.account.signIn({ mode: 'simple' });
    this.setState({ name: person.fullName, phone: person.alias, passId: person.accountIdToken });
  } catch (error) {
    if (isDashError(error, 'denied')) {
      this.setState({ typeDetails: true }); // Let them type their name and number.
      return;
    }
    this.setState({ note: errorMessage(error) });
  }
},
```

It needs `"account"` in `dot.json` `permissions`, and Dash asks the person first. `alias` is their phone number, or their email address when they have none. `accountIdToken` is the same for the person on any phone in this dot, and different in every other dot. A fresh `nonce` gives an ID for that sign-in only.

The dot sends `accountIdToken` to the server itself, so the server can't tell it from a made-up one. Use it to recognise the person and fill in their details, never to open their records or money.

## Pick something on another page

```ts title="pages/profile/controller.ts"
async chooseGenre() {
  const genre = await this.navigation.pushForResult<string>('pick-genre');
  if (genre) {
    this.setState({ genre });
  }
},
```

```ts title="pages/pick-genre/controller.ts"
pick(_state, event: DataEvent<{ genre: string }>) {
  this.navigation.backWithResult(event.data.genre);
},
```

`pushForResult` resolves `undefined` when the person goes back without choosing.

## A page a share link can open

```ts title="pages/concert/controller.ts"
export default definePage({
  initialState: state,
  share: { options: ['concert'] },
  hooks: {
    onLoad(options) {
      const concert = CONCERTS.find((item) => item.id === options.concert) ?? null;
      this.setState({ concert, notFound: concert === null });
    },
  },
});
```

The link is `https://dashit.to/d/<code>/concert?concert=jazz-night`. Anyone can type one by hand, so `onLoad` handles an option that matches nothing. Only the listed options travel in the link (up to 20).

## Keep data on the phone

```ts
await this.dash.storage.cache.set('fares', fares, { ttl: 60 * 60 * 1000 }); // null after an hour
const cached = await this.dash.storage.cache.get('fares');

await this.dash.storage.secret.set('session', token); // encrypted

await this.dash.storage.file.write('meters.json', JSON.stringify(meters));
```

Reading a file that isn't there rejects with `missing`, which on a first visit only means nothing is saved yet. Storage lives on one phone and is never backed up, so keep what matters on the server.

## Send an order once

Make one key per order and send it as `Idempotency-Key`, so a double tap or a retry doesn't make two orders.

```ts
// When the order starts
this.setState({ orderKey: await this.dash.crypto.randomUUID() });

// When it's sent
await this.dash.network.http.request<{ orderId: string }>({
  url: `${env.API_URL}/orders`,
  method: 'POST',
  headers: { 'Idempotency-Key': state.orderKey },
  body: { meter: state.meter, amount: state.amount },
  as: 'json',
});
```

## Brand colours and Dash's bars

```ts title="dot.ts"
export default defineDot({
  pages: ['index', 'confirm'],
  theme: {
    colors: { primary: { light: '#0168CC', dark: '#6CB4FF' } },
    shape: 'soft',
  },
  configs: { navigationBar: { title: 'Bo Energy' } },
  hooks: {
    onError(error) {
      console.error(error);
    },
  },
});
```

Set as few colours as you can, and Dash derives the rest. A page sets its own bar with `configs.navigationBar` in `definePage` or `this.setNavigationBar()`. A title that names Dash or Monime shows the dot's name instead. When a page hides the bar, keep its top right corner free for Dash's menu capsule.
