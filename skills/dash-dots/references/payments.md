# Take a payment

A dot never builds a checkout. It asks Dash for a payment, Dash shows its own payment sheet, and the person pays from a linked account, by card or with mobile money. The money goes to the Space that owns the dot. Exact types are under `## Payments` in [dash-api.md](dash-api.md).

## 1. Declare it

```json title="dot.json"
{ "permissions": { "payments": "Pay for ferry tickets" } }
```

Monime's review reads the reason. Dash doesn't prompt for this permission, because the person confirms each payment in the sheet.

## 2. Make the order on your server first

The business's server creates the order, works out the price, and returns an id. The dot uses that id as the payment's `reference`. Prices come from the server, never from what the dot sends.

## 3. Ask for the payment

```ts
// At the top of controller.ts:
// import { errorMessage, isDashError } from '@monimesl/dot-runtime';

async pay(state) {
  try {
    const result = await this.dash.payments.request({
      order: {
        name: 'Lungi ferry tickets',
        reference: state.orderId,
        lineItems: [
          { name: 'Adult ticket', unitPrice: { currency: 'SLE', value: 7500 }, quantity: state.adults },
        ],
      },
    });
    if (result.status === 'completed') {
      this.navigation.replace('tickets/confirmed', { orderId: state.orderId });
    } else if (result.status === 'pending') {
      this.setState({ note: "We're waiting for your payment to go through." });
    } else {
      this.setState({ note: "The payment didn't go through. Try again." });
    }
  } catch (error) {
    if (isDashError(error, 'cancelled')) {
      return;
    }
    this.setState({ note: errorMessage(error) });
  }
},
```

The order's rules:

- `unitPrice.value` is in minor units: SLE 75.00 is `7500`. All line items share one currency, `SLE` or `USD`.
- `name` is 3 to 64 characters. `reference` is up to 64 characters, and a new one for every order. Asking again with the same reference returns the first payment and never charges twice.
- 1 to 16 `lineItems`, each with a `quantity` from 1 to 100,000. An `imageUrl` must be `https`, on a host in `network.hosts`.
- Optional `description`, and up to 64 `metadata` pairs that travel with the payment and its webhooks. Keys are at most 48 characters and can't start with `@`, `_` or `monime`.

Optional request fields:

- `methods`: the ways to pay, in the order the sheet shows them. Leave it out for all three. For example `[{ type: 'momo', providers: ['m17'] }, { type: 'link' }]`. Mobile money providers are `m17` (Orange Money) and `m18` (Africell Money). A linked account is at a bank, so `link` takes only the banks `slb001`, `slb004`, `slb007`, `slb009` and `slb013`, and refuses `m17` and `m18`. `momo` also takes `phoneNumber`, like `+23276123456`.
- `financialAccountId`: another of the Space's accounts to pay into, holding the order's currency.

## 4. Handle every ending

`request` rejects before the person picks a way to pay:

| Code | Meaning |
| --- | --- |
| `cancelled` | They closed the sheet. Show nothing |
| `invalid` | Monime won't take the order (too small, a bad field). The message says what to change |
| `denied` | The dot or Space can't take payments now, the dot isn't published yet, or Dash held the call back |
| `failed` | Dash couldn't reach Monime. A retry may work |
| `timeout` | Dash stopped waiting |

Once they pick a way to pay, it resolves with `status`: `completed`, `pending`, `expired` or `cancelled`. `request` takes no `timeout`.

For a `pending` payment, check again when the person returns:

```ts
hooks: {
  async onShow() {
    const payment = await this.dash.payments.get(this.state.orderId);
    if (payment?.status === 'completed') {
      this.setState({ paid: true, note: '' });
    }
  },
},
```

`get` can also return `failed`, and `undefined` when the dot never asked for that reference.

## 5. Confirm on the server

Anyone can change what a dot reports. The business's server delivers the goods only once Monime confirms the payment, by webhook or by looking it up with the Space's API key. The result says what to look up:

| Paid with | Field |
| --- | --- |
| A card | `checkoutSessionId` |
| Mobile money | `paymentCodeId` |
| A linked account | `paymentId` |

Every result has `reference`, `amount` and `requestId` (Dash's id, which Monime support asks for), and `orderNumber` once completed.

Monime API docs for the server side: https://docs2.monime.io/docs/developer-resources/guides/webhook/introduction.

## Testing payments

- **Simulator**: the sheet is a stand-in. Pick a way to pay, then choose **Paid**, **Pending**, **Declined** or **Expired**. No money moves.
- **Live test on a phone**: payments are real and land in the Space. They work only once the dot is published and `dot.json` has its `id`. Before that, `request` and `get` reject with `denied`. Keep test orders small, and tell the user before you trigger one.
- **`testPage`**: fake `dash.payments.request`, as in [testing.md](testing.md).
