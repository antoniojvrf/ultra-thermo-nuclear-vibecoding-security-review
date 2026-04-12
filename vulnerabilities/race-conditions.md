# Race Conditions

## TOCTOU (Time-of-Check to Time-of-Use)

### What It Is
A gap between checking a condition and acting on it allows an attacker to change the state between the check and the action.

### Vulnerable Pattern — Double Spend
```javascript
// VULNERABLE — check-then-act without atomicity
app.post('/api/transfer', async (req, res) => {
  const { amount, toUserId } = req.body;
  const user = await db.users.findUnique({ where: { id: req.user.id } });

  if (user.balance >= amount) {  // CHECK
    // ⚠️ Attacker sends 2 requests simultaneously
    // Both pass the check before either deducts
    await db.users.update({
      where: { id: req.user.id },
      data: { balance: user.balance - amount }  // USE — stale value!
    });
    await db.users.update({
      where: { id: toUserId },
      data: { balance: { increment: amount } }
    });
  }
});
```

### Secure Pattern — Atomic Operations
```javascript
// SECURE — atomic update with WHERE clause
app.post('/api/transfer', async (req, res) => {
  const { amount, toUserId } = req.body;

  // Atomic: check and deduct in single query
  const result = await db.$transaction(async (tx) => {
    const updated = await tx.$executeRaw`
      UPDATE users SET balance = balance - ${amount}
      WHERE id = ${req.user.id} AND balance >= ${amount}
    `;

    if (updated === 0) throw new Error('Insufficient balance');

    await tx.users.update({
      where: { id: toUserId },
      data: { balance: { increment: amount } }
    });

    return { success: true };
  });

  res.json(result);
});
```

## Common Race Condition Scenarios

| Scenario | Risk | Prevention |
|----------|------|-----------|
| Balance/credit deduction | Double-spend money/credits | Atomic DB transactions with WHERE clause |
| Coupon/promo code redemption | Code used multiple times | Unique constraint + atomic update |
| Inventory management | Overselling items | `UPDATE ... WHERE stock > 0` atomic |
| Vote/like counting | Duplicate votes | Unique constraint on (user_id, item_id) |
| File operations | Symlink attacks, write to wrong file | Atomic file operations, use temporary files |
| Account creation with unique email | Duplicate accounts | Unique constraint at database level |
| Rate limit counters in application memory | Bypass rate limits | Use atomic Redis INCR or database |

## Mitigation Strategies

### 1. Database-Level Atomicity
```sql
-- Atomic decrement with check
UPDATE accounts
SET balance = balance - 100
WHERE user_id = $1 AND balance >= 100
RETURNING balance;
-- Returns 0 rows if insufficient balance
```

### 2. Idempotency Keys
```javascript
// Prevent duplicate operations
app.post('/api/payment', async (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];
  if (!idempotencyKey) return res.status(400).json({ error: 'Missing idempotency key' });

  // Check if already processed
  const existing = await db.operations.findUnique({ where: { key: idempotencyKey } });
  if (existing) return res.json(existing.result); // Return cached result

  // Process and store result atomically
  const result = await db.$transaction(async (tx) => {
    await tx.operations.create({ data: { key: idempotencyKey, status: 'processing' } });
    const paymentResult = await processPayment(req.body);
    await tx.operations.update({
      where: { key: idempotencyKey },
      data: { status: 'completed', result: paymentResult }
    });
    return paymentResult;
  });

  res.json(result);
});
```

### 3. Distributed Locks (Redis)
```javascript
import { Redis } from 'ioredis';
const redis = new Redis();

async function withLock(key, ttlMs, fn) {
  const lockKey = `lock:${key}`;
  const lockValue = crypto.randomUUID();

  // Acquire lock
  const acquired = await redis.set(lockKey, lockValue, 'PX', ttlMs, 'NX');
  if (!acquired) throw new Error('Resource locked');

  try {
    return await fn();
  } finally {
    // Release lock (only if we own it)
    const script = `
      if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("del", KEYS[1])
      end
      return 0
    `;
    await redis.eval(script, 1, lockKey, lockValue);
  }
}
```

## Checklist
- [ ] Financial operations use database transactions
- [ ] Balance checks and deductions are atomic (single query with WHERE)
- [ ] Unique constraints prevent duplicate entries at DB level
- [ ] Idempotency keys used for payment/critical operations
- [ ] Rate limit counters use atomic increment (Redis INCR or DB)
- [ ] No check-then-act patterns without atomicity guarantees
