# Passkey Feature Tester

A browser-based tool to test advanced passkey features: **PRF extension** (deterministic key derivation) and **Ed25519 signatures**.

## Features Tested

### 1. PRF Extension (Pseudo-Random Function)

The WebAuthn `prf` extension enables deterministic key derivation from a passkey. Same credential + same salt = same 32-byte output every time.

**Use cases:**
- End-to-end encryption (derive encryption keys from passkeys)
- Password manager vault decryption without master password
- Non-custodial identity wallets
- Client-side data encryption

### 2. Ed25519 Signatures

Tests if your authenticator supports EdDSA (COSE algorithm -8) instead of the default ES256 (P-256).

**Benefits of Ed25519:**
- Faster signature generation/verification
- Deterministic signatures (no random nonce)
- Simpler, less prone to side-channel attacks
- Smaller signatures (64 bytes)

## Usage

```bash
# Start local server
python3 -m http.server 8080

# Open in browser
open http://localhost:8080/passkey-test.html
```

## Platform Support

### PRF Extension Support (June 2025)

| Platform | Browser | PRF Support | Notes |
|----------|---------|-------------|-------|
| Android | Chrome | ✅ Yes | Best support — Google Password Manager passkeys work |
| Android | Edge | ✅ Yes | Works |
| Android | Firefox | ❌ No | Not supported |
| Windows | Chrome/Edge | ⚠️ Partial | Security keys only — Windows Hello lacks `hmac-secret` |
| macOS | Chrome | ⚠️ Partial | Security keys work; iCloud Keychain buggy |
| macOS | Safari 18+ | ⚠️ Buggy | Platform passkeys work but with issues |
| iOS 18+ | Safari | ⚠️ Limited | Platform passkeys work; YubiKeys do NOT |
| Linux | Chrome | ✅ Yes | Security keys work well |

### Ed25519 Support

| Authenticator | Ed25519 Support |
|---------------|-----------------|
| Windows Hello | ❌ No |
| macOS Touch ID | ❌ No |
| iOS Face ID / Touch ID | ❌ No |
| Android | ❌ No |
| YubiKey 5 Series | ✅ Yes |
| YubiKey Bio | ✅ Yes |
| SoloKey v2 | ✅ Yes |
| Nitrokey 3 | ✅ Yes |

### Hardware Security Keys with PRF

| Device | PRF (`hmac-secret`) |
|--------|---------------------|
| YubiKey 5 Series | ✅ Yes |
| YubiKey Bio | ✅ Yes |
| SoloKey v2 | ✅ Yes |
| Nitrokey 3 | ✅ Yes |
| Google Titan | ⚠️ Check model |

## How PRF Works

```
Input:  credential_secret + salt
Output: HMAC-SHA256 → 32 bytes (deterministic)
```

The authenticator holds a secret tied to each credential. When you provide a salt, it computes:

```
PRF(salt) = HMAC(credential_secret, "WebAuthn PRF" || 0x00 || salt)
```

Same inputs always produce the same output — enabling encryption key derivation without storing keys.

## How Ed25519 Works

Ed25519 uses EdDSA over Curve25519. Unlike ECDSA which requires a random nonce `k` for each signature, Ed25519 derives `k` deterministically:

```
k = SHA-512(private_key_prefix || message)
```

This eliminates nonce reuse vulnerabilities that have broken ECDSA implementations in the past.

## Test Interpretation

### PRF Test

| Result | Meaning |
|--------|---------|
| ✅ PRF Supported (at creation) | Authenticator advertises PRF capability |
| ✅ Outputs identical | PRF is working correctly — deterministic |
| ❌ PRF not supported | Try a YubiKey or Android with Chrome |
| ❌ Outputs differ | Implementation bug (should not happen) |

### Ed25519 Test

| Result | Meaning |
|--------|---------|
| ✅ Ed25519 SUPPORTED | Credential created with EdDSA (-8) |
| ⚠️ Created but not Ed25519 | Authenticator used fallback algorithm |
| ❌ NotSupportedError | Authenticator doesn't support Ed25519 |

## Recommendations

**For PRF testing:**
- Best: Android + Chrome + Google Password Manager
- Good: Any platform + Chrome + YubiKey 5
- Avoid: Windows Hello, iOS with security keys

**For Ed25519 testing:**
- Use hardware security keys (YubiKey 5, SoloKey, Nitrokey)
- Platform authenticators generally don't support it

## Technical Details

### WebAuthn Extensions Used

```javascript
// PRF extension (creation)
extensions: { prf: {} }

// PRF extension (assertion)
extensions: {
  prf: {
    eval: { first: salt }  // ArrayBuffer
  }
}

// Ed25519 algorithm
pubKeyCredParams: [
  { type: "public-key", alg: -8 }  // EdDSA
]
```

### COSE Algorithm Identifiers

| Algorithm | COSE ID | Curve |
|-----------|---------|-------|
| ES256 | -7 | P-256 (secp256r1) |
| ES384 | -35 | P-384 |
| ES512 | -36 | P-521 |
| EdDSA | -8 | Ed25519 / Ed448 |
| RS256 | -257 | RSA |

## References

- [WebAuthn Level 3 Spec](https://www.w3.org/TR/webauthn-3/)
- [PRF Extension Explainer](https://github.com/w3c/webauthn/wiki/Explainer:-PRF-extension)
- [CTAP2 hmac-secret](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-hmac-secret-extension)
- [RFC 8032 - Ed25519](https://datatracker.ietf.org/doc/html/rfc8032)
- [COSE Algorithms Registry](https://www.iana.org/assignments/cose/cose.xhtml#algorithms)

## License

MIT
