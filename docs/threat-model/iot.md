# Threat model — IoT terminal firmware (planned, Week 3)

> Extends [`canonical.md`](canonical.md).
> **Status:** stub. IoT firmware does not exist yet. B03 (Week 3 build prompt) bootstraps it. Hardware: Orange Pi 5, MFS100 fingerprint, Astra Pro Plus camera.

## Surfaces (planned)

- Outbound HTTPS to `https://api.zeroauth.dev/v1/*` (TLS 1.3, pinned certificate)
- Local biometric capture via USB (MFS100, Astra Pro Plus)
- Local secure storage of device API key + tenant ID

## Attacks (canonical mapping)

- **A-01 — Cross-tenant data read** — applies if the device API key is compromised; mitigated by scoping the device key to a single `(tenant_id, environment)` at registration.
- **A-02 — Replayed proof verification** — primary attack class for IoT, since proofs are generated on-device.
- **Local secret extraction** — root access to the Orange Pi exposes the device API key. Out-of-scope for v1; addressed in v2 with TPM-backed key storage.

## IoT-specific concerns (to be expanded post-B03)

- Liveness / anti-spoofing at the camera layer — printed photo rejection is Demo 1 in the four-demo battery
- Airplane-mode offline queue — Demo 2; firmware buffers up to N verifications and syncs on reconnect
- Physical tamper detection — case-open switch or motion sensor; out-of-scope for v1
- Firmware update channel — signed updates only, signing key in `/opt/zeroauth/.env` (separate from API admin key)

---

LAST_UPDATED: 2026-05-13
