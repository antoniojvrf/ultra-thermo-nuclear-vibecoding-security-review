# Payment Security (Stripe)

## Client-Side Price Manipulation — Critical

```typescript
// ❌ CRITICAL — price from client request
export async function POST(req: Request) {
  const { price, productName } = await req.json();
  const session = await stripe.checkout.sessions.create({
    line_items: [{ price_data: { unit_amount: price, currency: 'usd',
      product_data: { name: productName } } }],
    mode: 'payment',
  });
}
// Attacker sends { price: 1, productName: "Premium Plan" } → pays $0.01

// ✅ SECURE — look up price server-side
export async function POST(req: Request) {
  const { productId } = await req.json();
  const product = await db.products.findUnique({ where: { id: productId } });
  if (!product) return new Response('Not found', { status: 404 });

  const session = await stripe.checkout.sessions.create({
    line_items: [{ price: product.stripePriceId }], // price from DB, not client
    mode: 'payment',
  });
}
```

## Webhook Signature Verification

```typescript
// ❌ VULNERABLE — no signature verification
export async function POST(req: Request) {
  const event = await req.json(); // anyone can fake this
  if (event.type === 'checkout.session.completed') {
    await activateSubscription(event.data.object.customer);
  }
}

// ✅ SECURE — verify webhook signature
import Stripe from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function POST(req: Request) {
  const body = await req.text(); // raw body required for signature
  const sig = req.headers.get('stripe-signature')!;

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(body, sig, process.env.STRIPE_WEBHOOK_SECRET!);
  } catch (err) {
    return new Response('Invalid signature', { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object as Stripe.Checkout.Session;
      await activateSubscription(session.customer as string);
      break;
    case 'customer.subscription.deleted':
      const sub = event.data.object as Stripe.Subscription;
      await deactivateSubscription(sub.customer as string);
      break;
  }

  return new Response('OK', { status: 200 });
}
```

## Subscription Status Validation

```typescript
// ❌ VULNERABLE — trusting client-side subscription status
// Client sends: { isPremium: true } in headers or cookies

// ✅ SECURE — check subscription server-side
async function requireSubscription(userId: string) {
  const user = await db.users.findUnique({
    where: { id: userId },
    include: { subscription: true }
  });

  if (!user?.subscription ||
      user.subscription.status !== 'active' ||
      new Date(user.subscription.currentPeriodEnd) < new Date()) {
    throw new Error('Active subscription required');
  }

  return user.subscription;
}
```

## Quantity and Discount Manipulation
```typescript
// ❌ VULNERABLE — client controls quantity and discount
const session = await stripe.checkout.sessions.create({
  line_items: [{
    price: priceId,
    quantity: req.body.quantity, // attacker sets quantity: 0
  }],
  discounts: [{ coupon: req.body.couponCode }], // attacker uses expired/invalid coupon
});

// ✅ SECURE — validate quantity, verify coupon server-side
const quantity = Math.max(1, Math.min(parseInt(req.body.quantity) || 1, 100));

let discounts = [];
if (req.body.couponCode) {
  try {
    const coupon = await stripe.coupons.retrieve(req.body.couponCode);
    if (coupon.valid) discounts = [{ coupon: coupon.id }];
  } catch { /* invalid coupon — skip */ }
}
```

## Checklist
- [ ] Prices looked up server-side (never from client)
- [ ] Webhook signatures verified (stripe.webhooks.constructEvent)
- [ ] Webhook endpoint uses raw body (not parsed JSON)
- [ ] Subscription status checked server-side (not from client/cookie)
- [ ] Quantities validated and bounded server-side
- [ ] Coupon codes verified via Stripe API before applying
- [ ] Stripe secret key is server-side only (no NEXT_PUBLIC_ prefix)
- [ ] Webhook secret stored in environment variable
- [ ] Idempotency: handle duplicate webhook events gracefully
