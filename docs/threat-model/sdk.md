# Threat model — Mobile SDK (planned, Week 5)

> Extends [`canonical.md`](canonical.md).
> **Status:** stub. Mobile SDK does not exist yet. B04 (Week 5 build prompt) bootstraps it. Target platforms: iOS (Swift), Android (Kotlin).

## Surfaces (planned)

- Outbound HTTPS to `https://api.zeroauth.dev/v1/*` (TLS 1.3, pinned certificate)
- Customer-app integration point via public SDK methods (`registerUser`, `verifyProof`, `getDevices`, …)
- On-device biometric capture (TouchID / FaceID / Android BiometricPrompt)
- Local secure storage (iOS Keychain, Android Keystore) for tenant API key

## Attacks (canonical mapping)

- **A-01 — Cross-tenant data read** — the customer app embeds the tenant API key; key leak = tenant compromise.
- **A-02 — Replayed proof verification** — proofs generated on-device, same as IoT.
- **Reverse-engineering of the app to extract the API key** — out-of-scope for v1; addressed in v2 by routing through a customer backend.

## SDK-specific concerns (to be expanded post-B04)

- Certificate pinning across platforms (must survive iOS app-store TLS-cert rotation)
- Biometric-prompt UX — the SDK must NEVER receive the biometric template itself; only the platform's authentication result + a locally-derived secret
- API key embedding — public SDK methods take the tenant key as a parameter; the app developer's job is to fetch it from their backend, not ship it in the binary
- Symbol stripping + obfuscation — non-security but reduces reverse-engineering effort

---

LAST_UPDATED: 2026-05-13
