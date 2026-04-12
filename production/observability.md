# Observability & Logging Security

## PII in Logs

```javascript
// ❌ VULNERABLE — logging sensitive data
logger.info('User login', { email: user.email, password: req.body.password });
logger.error('Payment failed', { cardNumber: req.body.card, user: req.body });
logger.info('Request', { headers: req.headers }); // may contain auth tokens

// ✅ SECURE — sanitize before logging
logger.info('User login', { userId: user.id }); // no email, no password
logger.error('Payment failed', { userId: user.id, last4: card.last4 });
logger.info('Request', { path: req.path, method: req.method, ip: req.ip });
```

### Data to NEVER Log
- Passwords (raw or hashed)
- Full credit card numbers
- Social security numbers
- API keys and tokens
- Session IDs
- Full request/response bodies (may contain secrets)
- Authorization headers
- PII not needed for debugging (email, phone, address)

### Sanitization Pattern
```javascript
function sanitizeForLogging(obj) {
  const sensitive = ['password', 'token', 'secret', 'key', 'authorization',
    'cookie', 'creditCard', 'ssn', 'cardNumber'];
  const sanitized = { ...obj };
  for (const key of Object.keys(sanitized)) {
    if (sensitive.some(s => key.toLowerCase().includes(s))) {
      sanitized[key] = '[REDACTED]';
    }
  }
  return sanitized;
}
```

## Log Injection Prevention

```javascript
// ❌ VULNERABLE — user input directly in log entries
logger.info(`User ${userInput} logged in`);
// Attack: userInput = "admin\n[ERROR] System compromised"
// Creates fake log entries that confuse incident response

// ✅ SECURE — sanitize log inputs
function sanitizeLogInput(input) {
  return String(input)
    .replace(/[\n\r]/g, ' ')    // Remove newlines
    .replace(/[\x00-\x1f]/g, '') // Remove control characters
    .slice(0, 500);              // Limit length
}
logger.info('User login', { username: sanitizeLogInput(userInput) });
```

## Structured Logging

```javascript
// ✅ Use structured logging (JSON format) — easier to parse, harder to inject
const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  redact: ['req.headers.authorization', 'req.headers.cookie', '*.password', '*.token'],
  serializers: {
    req: (req) => ({
      method: req.method,
      url: req.url,
      remoteAddress: req.remoteAddress,
    }),
    err: pino.stdSerializers.err,
  },
});

// Output:
// {"level":30,"time":1234567890,"msg":"User login","userId":"abc-123"}
```

## Security Event Monitoring

### Events to Log and Alert On
| Event | Log Level | Alert |
|-------|-----------|-------|
| Failed login attempts | WARN | After 5 failures in 5 min |
| Successful login from new device/location | INFO | If user has 2FA disabled |
| Password change | INFO | Always notify user via email |
| Permission elevation | WARN | Always |
| Account deletion | INFO | Always |
| API rate limit exceeded | WARN | If sustained |
| Unusual data access patterns | WARN | After threshold |
| Configuration changes | INFO | Always in audit log |
| Failed authorization checks | WARN | After pattern detected |
| Webhook signature verification failures | ERROR | Always |

## Checklist
- [ ] No PII in logs (passwords, tokens, card numbers, SSN)
- [ ] Request/response bodies not logged raw
- [ ] Log injection prevented (newline/control char sanitization)
- [ ] Structured logging format (JSON) used
- [ ] Sensitive fields redacted automatically
- [ ] Security events logged with sufficient detail
- [ ] Log retention policy defined
- [ ] Alerting configured for critical security events
- [ ] Log access restricted to authorized personnel
- [ ] Production logs don't include debug/verbose output
