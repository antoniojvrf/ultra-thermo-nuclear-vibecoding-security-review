# React Native / Expo Security

## Secure Token Storage

```javascript
// ❌ VULNERABLE — AsyncStorage is NOT encrypted
import AsyncStorage from '@react-native-async-storage/async-storage';
await AsyncStorage.setItem('authToken', token); // readable by any app with root

// ✅ SECURE — use platform keychain/keystore
import * as SecureStore from 'expo-secure-store'; // Expo
await SecureStore.setItemAsync('authToken', token);

// Or react-native-keychain for bare React Native
import * as Keychain from 'react-native-keychain';
await Keychain.setGenericPassword('auth', token);
```

## API Key Protection

```javascript
// ❌ CRITICAL — API keys in JS bundle (extractable)
const response = await fetch('https://api.openai.com/v1/chat/completions', {
  headers: { Authorization: `Bearer ${OPENAI_API_KEY}` } // in the bundle!
});

// ✅ SECURE — proxy through your backend
const response = await fetch('https://your-api.com/api/ai/chat', {
  headers: { Authorization: `Bearer ${userToken}` }, // user auth only
  body: JSON.stringify({ message: userInput }),
});
// Backend makes the OpenAI call with the secret key
```

## Expo Environment Variables
```
# ❌ All EXPO_PUBLIC_ vars are in the JS bundle
EXPO_PUBLIC_STRIPE_SECRET_KEY=sk_live_...  # EXPOSED
EXPO_PUBLIC_DATABASE_URL=...                # EXPOSED

# ✅ Only public identifiers
EXPO_PUBLIC_API_URL=https://your-api.com
EXPO_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...
```

## Deep Link Validation
```javascript
// ❌ VULNERABLE — blindly trusting deep link parameters
Linking.addEventListener('url', ({ url }) => {
  const parsed = Linking.parse(url);
  if (parsed.path === 'reset-password') {
    resetPassword(parsed.queryParams.token); // token could be manipulated
  }
});

// ✅ SECURE — validate deep link parameters server-side
Linking.addEventListener('url', ({ url }) => {
  const parsed = Linking.parse(url);
  if (parsed.path === 'reset-password') {
    // Verify token with server before taking action
    const response = await fetch(`${API_URL}/auth/verify-reset-token`, {
      method: 'POST',
      body: JSON.stringify({ token: parsed.queryParams.token }),
    });
    if (response.ok) {
      navigation.navigate('ResetPassword', { token: parsed.queryParams.token });
    }
  }
});
```

## Certificate Pinning
```javascript
// For high-security apps, pin server certificates
// Prevents MITM attacks even with compromised CA
import { fetch } from 'react-native-ssl-pinning';

const response = await fetch(url, {
  sslPinning: {
    certs: ['server-cert'], // sha256 hash of server cert
  },
});
```

## Checklist
- [ ] Auth tokens stored in SecureStore/Keychain (NOT AsyncStorage)
- [ ] No API keys in JavaScript bundle
- [ ] API calls proxied through backend for secret keys
- [ ] No secrets in EXPO_PUBLIC_ environment variables
- [ ] Deep link parameters validated server-side
- [ ] Sensitive screens protected with biometric auth (optional)
- [ ] Debug mode and logging disabled in production builds
- [ ] ProGuard/R8 enabled for Android release builds
- [ ] App Transport Security (ATS) not disabled on iOS
