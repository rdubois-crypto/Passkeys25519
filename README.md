# Ed25519 Passkey Support Tester

A simple browser-based tool to test if your device supports Ed25519 (EdDSA) signatures for WebAuthn passkeys.

## Background

WebAuthn passkeys typically use **ES256** (ECDSA over P-256/secp256r1) as the default signature algorithm. However, **Ed25519** (EdDSA over Curve25519) offers several advantages:

- Faster signature generation and verification
- Simpler implementation (less prone to side-channel attacks)
- Deterministic signatures (no random nonce required)
- Smaller signatures (64 bytes vs 70+ for DER-encoded ECDSA)

Ed25519 is identified as **COSE algorithm -8** in WebAuthn.

## Usage

1. Start a local server:
```bash
   python3 -m http.server 8080
```

2. Open in your browser:
```
   http://localhost:8080/ed25519-test.html
```

3. Click **"Test Ed25519 Support"** and complete the passkey prompt.

## Results

| Result | Meaning |
|--------|---------|
| ✅ Ed25519 is SUPPORTED | Your platform authenticator supports EdDSA (-8) |
| ⚠️ Credential created but NOT Ed25519 | Passkey created with a different algorithm |
| ❌ Ed25519 NOT supported | Platform rejected the algorithm (typically `NotSupportedError`) |

## Current Platform Support (2024-2025)

| Platform | Ed25519 Support |
|----------|-----------------|
| Windows Hello | ❌ No |
| macOS Touch ID | ❌ No |
| iOS Face ID / Touch ID | ❌ No |
| Android | ❌ No |
| YubiKey 5 | ✅ Yes (Ed25519 via FIDO2) |
| SoloKey | ✅ Yes |
| Nitrokey 3 | ✅ Yes |

> **Note:** Hardware security keys are more likely to support Ed25519 than platform authenticators.

## How It Works

The test requests credential creation with **only** EdDSA in the allowed algorithms:
```javascript
pubKeyCredParams: [
  { type: "public-key", alg: -8 }  // EdDSA only
]
```

If the authenticator doesn't support Ed25519, it must reject the request per the WebAuthn spec.

## References

- [WebAuthn Spec](https://www.w3.org/TR/webauthn-3/)
- [COSE Algorithms Registry](https://www.iana.org/assignments/cose/cose.xhtml#algorithms)
- [RFC 8032 - Ed25519](https://datatracker.ietf.org/doc/html/rfc8032)

## License

MIT
