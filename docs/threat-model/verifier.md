# Threat model — Verifier service (planned, Week 2)

> Extends [`canonical.md`](canonical.md).
> **Status:** stub. The verifier service does not exist yet as a separate component — its code lives inline in `pulkitpareek18/ZeroAuth: src/services/zkp.ts`. B02 (Week 2 build prompt) splits it out. This file gets fleshed out when that PR opens.

## Surfaces (planned)

- Internal HTTPS endpoint `POST /verify` (loopback only, behind the API)
- Trusted setup files: `circuits/identity_proof.zkey`, `circuits/identity_proof.vkey.json`, `circuits/identity_proof_js/identity_proof.wasm` (loaded at startup)

## Attacks (canonical mapping)

- **A-02 — Replayed proof verification** — primary mitigation for the within-5-minute replay class lives in the verifier service once split (issued-nonce binding moves here).
- **Soundness break in the Groth16 setup** — this is the catastrophic case. The verifier rotation procedure is in [`../shared/security-policy.md` §3.7](../shared/security-policy.md): 48-hour window from suspicion to rotation, 72-hour window for customer notification.

## Verifier-specific concerns (to be expanded post-B02)

- Side-channel attacks on the WASM proving runtime — out-of-scope for v1; documented for completeness.
- WASM file integrity at startup — should verify SHA-256 against `evidence-pack-sources/CHECKSUMS.md` and refuse to start on mismatch.
- vkey provenance — verification key was generated during the Powers of Tau ceremony output processing. The ceremony itself is documented in [TODO: ADR in product repo].

---

LAST_UPDATED: 2026-05-13
