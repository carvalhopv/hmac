# Capacitor HMAC Plugin

> A native Capacitor plugin for HMAC-SHA256 signing. Keeps your secret in the native layer (Keystore/Keychain) and exposes a simple JavaScript API for generating request signatures.

## Table of Contents

* [Features](#features)
* [Installation](#installation)
* [Usage](#usage)

  * [Initializing the Plugin](#initializing-the-plugin)
  * [Signing Data](#signing-data)
* [API Reference](#api-reference)
* [Platform Details](#platform-details)

  * [Android](#android)
  * [iOS](#ios)
* [Security Considerations](#security-considerations)
* [Development](#development)
* [Contributing](#contributing)
* [License](#license)

## Features

* **Native HMAC-SHA256** implementation on Android (Kotlin/Java) and iOS (Swift)
* Secret stored securely using Android Keystore and iOS Keychain
* Exposes a single `sign(data: string)` method in JavaScript
* Minimal dependencies
* Tested with Capacitor 5.x and Quasar, Ionic, or any Capacitor-based frontend

## Installation

```bash
# From your Capacitor project root
npm install @carvalhopv/capacitor-hmac-plugin
npx cap sync
```

Then, rebuild your native projects:

```bash
npx cap copy android && npx cap open android
# – or –
npx cap copy ios    && npx cap open ios
```

## Usage

### Initializing the Plugin

No extra initialization is required. The plugin is automatically registered with Capacitor.

### Signing Data

Use the plugin in your JavaScript/TypeScript code to generate an HMAC signature:

```ts
import { HmacPlugin } from '@carvalhopv/capacitor-hmac-plugin';

async function getSignature(rawData: string): Promise<string> {
  try {
    const result = await HmacPlugin.sign({ data: rawData });
    return result.signature; // Base64-encoded HMAC-SHA256
  } catch (err) {
    console.error('HMAC signing failed', err);
    throw err;
  }
}

// Example interceptor (Axios)
api.interceptors.request.use(async config => {
  const timestamp = Date.now().toString();
  const method = (config.method || 'GET').toUpperCase();
  const path = new URL(
    `${config.baseURL?.replace(/\/$/, '')}/${config.url?.replace(/^\//, '')}`
  ).pathname;
  const body = config.data ? JSON.stringify(config.data) : '{}';
  const rawData = `${method}:${path}:${timestamp}:${body}`;

  const signature = await getSignature(rawData);

  config.headers['x-timestamp'] = timestamp;
  config.headers['x-signature'] = signature;
  return config;
});
```

## API Reference

| Method            | Parameters         | Returns                          | Description                                                          |
| ----------------- | ------------------ | -------------------------------- | -------------------------------------------------------------------- |
| `HmacPlugin.sign` | `{ data: string }` | `Promise<{ signature: string }>` | Generates a Base64-encoded HMAC-SHA256 signature for the given data. |

## Platform Details

### Android

* Secret is stored in code at `android/src/main/java/.../HmacPlugin.kt`
* Uses `javax.crypto.Mac` with `HmacSHA256`
* Optionally integrates with Android Keystore for hardware-backed key protection
* ProGuard/R8 rules are provided to keep plugin classes intact

### iOS

* Secret is stored in `ios/Plugin/Plugin/HmacPlugin.swift`
* Uses CommonCrypto’s `CCHmac` API
* Optionally stores the key in the iOS Keychain

## Security Considerations

* **Secret Management**: The secret is compiled into the native binary—use ProGuard/R8 on Android and strip symbols on iOS to make extraction harder.
* **Attestation**: For stronger protection, combine with SafetyNet/Play Integrity or DeviceCheck/ATT.
* **Key Rotation**: If your secret is compromised, you must release a new app version with a rotated key.
* **Rate Limiting**: Complement client-side signing with server-side rate-limits, mTLS, and token expiry for defense-in-depth.

## Development

1. Clone this repo

   ```bash
   git clone https://github.com/carvalhopv/capacitor-hmac-plugin.git
   cd capacitor-hmac-plugin
   ```
2. Install dependencies

   ```bash
   npm install
   ```
3. Build and test

   ```bash
   npm run build
   npm test
   ```
4. Link into your sample Capacitor app for manual testing

   ```bash
   npm link
   cd ../your-capacitor-app
   npm link @carvalhopv/capacitor-hmac-plugin
   npx cap sync
   ```

## Contributing

Contributions, issues, and feature requests are welcome!

## License

MIT

*Patrick V. Carvalho*
