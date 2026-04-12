# Cryptography Security

## Hashing

### Password Hashing
- **Use**: Argon2id (preferred), bcrypt, scrypt
- **Never**: MD5, SHA1, SHA256 (unsalted), plain text

### Data Integrity Hashing
- **Use**: SHA-256, SHA-3, BLAKE2
- **Never**: MD5, SHA1 (both have collision attacks)

## Encryption

### Symmetric Encryption
- **Use**: AES-256-GCM (authenticated encryption)
- **Never**: AES-ECB (reveals patterns), AES-CBC without HMAC, DES, 3DES, RC4

```javascript
import { createCipheriv, createDecipheriv, randomBytes } from 'crypto';

function encrypt(plaintext, key) {
  const iv = randomBytes(12); // 96-bit IV for GCM
  const cipher = createCipheriv('aes-256-gcm', key, iv);
  const encrypted = Buffer.concat([cipher.update(plaintext, 'utf8'), cipher.final()]);
  const tag = cipher.getAuthTag();
  // Store iv + tag + encrypted together
  return Buffer.concat([iv, tag, encrypted]).toString('base64');
}

function decrypt(ciphertext, key) {
  const data = Buffer.from(ciphertext, 'base64');
  const iv = data.subarray(0, 12);
  const tag = data.subarray(12, 28);
  const encrypted = data.subarray(28);
  const decipher = createDecipheriv('aes-256-gcm', key, iv);
  decipher.setAuthTag(tag);
  return decipher.update(encrypted) + decipher.final('utf8');
}
```

### Key Rules
1. **Never reuse nonces/IVs** with the same key — generates fresh IV per encryption
2. **Use authenticated encryption** (GCM, ChaCha20-Poly1305) — prevents tampering
3. **Key size**: AES-256 = 32 bytes of cryptographically random data
4. **Never use ECB mode** — identical blocks produce identical ciphertext (reveals patterns)

## Key Management

### Key Derivation from Passwords
```javascript
import { scryptSync, randomBytes } from 'crypto';

function deriveKey(password) {
  const salt = randomBytes(16);
  const key = scryptSync(password, salt, 32); // 256-bit key
  return { key, salt }; // store salt alongside encrypted data
}
```

### Key Storage Rules
- Never hardcode encryption keys in source code
- Use environment variables or dedicated key management services (AWS KMS, GCP KMS, HashiCorp Vault)
- Rotate keys periodically
- Use separate keys for different purposes (encryption vs signing)

## Common Mistakes

| Mistake | Risk | Fix |
|---------|------|-----|
| Using `Math.random()` for tokens | Predictable output | Use `crypto.randomBytes()` or `crypto.randomUUID()` |
| Nonce/IV reuse in AES-GCM | Catastrophic — reveals key | Generate fresh IV per encryption |
| Comparing hashes with `===` | Timing attack reveals hash | Use `crypto.timingSafeEqual()` |
| Rolling your own crypto | Unknown vulnerabilities | Use established libraries (libsodium, Web Crypto API) |
| Encryption without authentication | Ciphertext can be tampered | Use AES-GCM or encrypt-then-MAC |
| Storing encryption key next to data | Key compromise = data compromise | Separate key storage from data |

## Checklist
- [ ] Passwords hashed with Argon2id/bcrypt (never MD5/SHA)
- [ ] Encryption uses AES-256-GCM or ChaCha20-Poly1305
- [ ] Fresh IV/nonce generated for each encryption operation
- [ ] Tokens generated with `crypto.randomBytes()` (not `Math.random()`)
- [ ] Hash comparison uses timing-safe comparison
- [ ] Encryption keys stored in KMS / env vars (not in code)
- [ ] No custom cryptographic algorithms (use established libraries)
