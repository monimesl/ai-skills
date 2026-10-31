# Test a page

`testPage` runs one page's controller in a plain test, with no Dash and no simulator. It works with any runner. With Vitest:

```bash
npm install --save-dev vitest
```

Add `"test": "vitest run"` to `package.json` scripts.

```ts title="pages/cart/controller.test.ts"
import { testPage } from '@monimesl/dot-runtime/testing';
import { describe, expect, it } from 'vitest';

import cart from './controller';

describe('cart', () => {
  it('adds jollof and goes to checkout', async () => {
    const page = await testPage(cart, { options: { from: 'menu' } });

    await page.tap('add');
    await page.tap('setQuantity', { value: 3 });
    await page.tap('checkout');

    expect(page.state).toMatchObject({ items: 1, quantity: 3 });
    expect(page.navigations).toEqual([{ action: 'push', page: 'checkout', options: { items: '1' } }]);
  });
});
```

- Import `@monimesl/dot-runtime/testing` before the controller, so the globals exist when it loads.
- `env` starts empty. Fill what the page reads: `Object.assign(env, { API_URL: 'https://api.example.test' })`.
- `testPage(page, { options, host, results })` runs `onLoad` then `onShow`. `results` maps a page id to what `pushForResult` gets back (left out means `undefined`).
- Drive it with `tap(name, detail?)`, and `show()`, `hide()`, `refresh()`, `unload()`.
- Check `state`, `navigations`, `hostCalls` (the Dash calls made) and `result` (from `backWithResult`).
- A throwing handler makes `tap` reject: `await expect(page.tap('pay')).rejects.toThrow('Payment declined')`.

## Fake Dash calls

`host` maps a call name (without `this.`) to what it returns. A call with no fake fails with `unsupported`. `dash.utils` and `dash.supports` need no fake.

```ts
const page = await testPage(cancelOrder, {
  host: {
    'dash.ui.alert': () => ({ action: 'cancel' }),
    'dash.network.http.request': () => ({
      ok: true,
      status: 200,
      statusText: 'OK',
      headers: { 'content-type': 'application/json' },
      body: '{"cancelled":true}',
    }),
  },
});
await page.tap('cancel');
expect(page.hostCalls.map((call) => call.method)).toEqual(['dash.ui.alert', 'dash.network.http.request']);
```

Write a response `body` as text, even when the page reads JSON. The other way round, `hostCalls[i].params` holds a request's `body` as JSON text, so compare it with `JSON.parse(...)`.

A payment:

```ts
host: {
  'dash.payments.request': () => ({
    method: 'momo',
    paymentCodeId: 'pmc-test',
    reference: 'order-1042',
    requestId: 'req-test',
    status: 'completed',
    amount: { currency: 'SLE', value: 15000 },
  }),
},
```

## Make a call fail

```ts
import { DashError } from '@monimesl/dot-runtime';

host: {
  'dash.contacts.pickPhone': () => {
    throw new DashError('denied', 'The person said no');
  },
},
```

Use the code `cancelled` for a picker the person closed.

## Events with data

`tap` sends an event without `data`. For a handler that reads `event.data`, send the whole event:

```ts
await page.dispatch('open', createEvent('tap', {}, { id: 'order-1' }));
```

A mutation is a plain function, so it can be tested alone:

```ts
expect(cart.mutations?.add?.({ items: 1, quantity: 1 }, createEvent('tap'))).toEqual({ items: 2 });
```

## What tests don't cover

`testPage` doesn't draw the view or check that state is plain data. `monime dot build` checks the view, and the simulator shows it.
