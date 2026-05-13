# ZeroAuth Shared Security Policy

> **Last reviewed by:** Pulkit Pareek (technical), Amit Dua (governance) on 2026-05-13
> **Status:** v1 — initial draft. Sections §3 (cryptographic primitives) breach windows, §5 (data residency), §6 (audit logging), and §7 (vulnerability disclosure) are marked **PROVISIONAL** pending external DPO counsel engagement (see ADR-0005 in `pulkitpareek18/ZeroAuth` — engagement target before first pilot SOW signing, ~2026-07-01). Until counsel is engaged, the DPO function is filled jointly by Pulkit + Amit; risks of operating without privileged communications are accepted by the founders.

This is the security policy every ZeroAuth repo agrees to. Every product repo's `CLAUDE.md` MUST link to this file. When a product repo's local policy contradicts this file, this file wins; the product repo updates.

## §1. Scope

This policy applies to every ZeroAuth artifact: the central API (`pulkitpareek18/ZeroAuth`), the verifier service (planned), the IoT terminal firmware (planned), the mobile SDK (planned), the dashboard (currently in API repo), the Solidity contracts (`contracts/`), the Circom circuit (`circuits/`), and the docs site.

It does NOT apply to:

- The marketing site at `https://zeroauth.dev/` (public marketing surface — handled by content policy, not security policy)
- Internal operational documents (lead lists, deck drafts, contracts under negotiation)

## §2. Identity & secrets

1. **Tenant API keys.** Format `za_(live|test)_<48 hex>`. Stored as SHA-256 hashes at rest. Never logged in plaintext. Never returned in API responses except the one-time creation reveal.
2. **Console JWTs.** HS256 today, RS256 on roadmap. 24-hour expiry. Refresh requires re-authentication for now (no refresh tokens until RS256 + JWKS lands).
3. **Admin x-api-key.** Static, stored in VPS `/opt/zeroauth/.env`, rotated on every team change. Never committed to a repo.
4. **No password storage outside the console signup path.** The `/v1/*` surface authenticates via API key only.
5. **No raw biometric data over the wire, ever.** Endpoints reject any payload containing keys named `image`, `template`, `pixel`, `depth`, `frame`, or `biometric_data`. The verifier accepts proofs and SHA-256 hashes only. Input buffers are GC'd immediately after hashing.

## §3. Cryptographic primitives

1. **Hashing for commitments.** Poseidon (BN128 field). Implementation: snarkjs / circomlib.
2. **Hashing for general integrity.** SHA-256.
3. **ZK proofs.** Groth16 over BN128. Trusted setup: Powers of Tau ceremony output, frozen by ADR before any deploy.
4. **Signatures off-chain.** Ed25519 (libsodium).
5. **Signatures on-chain.** secp256k1 (Solidity 0.8 default).
6. **TLS.** TLS 1.3 only. Caddy auto-TLS via Let's Encrypt. HSTS preload enabled.
7. **Breach disclosure window for the verifier circuit:** if a soundness break is discovered in the deployed Groth16 setup, the verifier is rotated within **48 hours** and all customers are notified within **72 hours** per §3.4.

## §4. Tenant isolation

1. Every query that returns customer data MUST be scoped by `(tenant_id, environment)` in the WHERE clause. Enforced in middleware (`src/middleware/tenant-auth.ts` in `pulkitpareek18/ZeroAuth`), not in handlers.
2. No admin endpoint reveals data from more than one tenant in a single response.
3. Cross-tenant access requires explicit ADR + customer consent on file + 30-day audit-log review.

## §5. Data residency

1. Customer biometric commitments + audit events: stored in PostgreSQL in `ap-south-1` (India). [TODO: verify VPS region and lock in via ADR — DPDP §16 has implications.]
2. Backups: same region, encrypted at rest (AES-256-GCM). [TODO: confirm encryption-at-rest is actually enabled on the production VPS Postgres.]
3. No customer data is sent to any service outside India without prior written consent. [TODO: this includes LLM provider calls — confirm we never send customer data into prompts.]

## §6. Audit logging

1. Every state-changing API request writes one row to `audit_events`. Schema: see [`/docs/compliance/audit-format.md`](../compliance/audit-format.md).
2. Audit log is append-only at the application layer. Hash-chained rows (planned ADR — currently un-chained).
3. Audit log retention: 7 years (per IRDAI §6.2 — longest applicable retention).
4. Audit log export to customer on request via `GET /v1/audit`.

## §7. Vulnerability disclosure

Reports go to `security@zeroauth.dev`. Response within 72 hours. Coordinated disclosure window: 90 days. Hall of fame at <https://zeroauth.dev/security>. [TODO: confirm `security@` mailbox exists and routes to Pulkit + Amit.]

## §8. Dependencies

Every new dependency is an ADR. Process is in [`pulkitpareek18/ZeroAuth: .claude/skills/dep-add/SKILL.md`](https://github.com/pulkitpareek18/ZeroAuth/blob/main/.claude/skills/dep-add/SKILL.md). No exceptions; the supply-chain risk is too high.

## §9. Network ingress

1. Production HTTP surface terminates at Caddy. Only `:443` open to the internet (plus `:80` for ACME).
2. Database, Redis, internal verifier-to-API traffic stays on the Docker network; not exposed to host.
3. SSH on `:22` is allowlisted to known operator IPs only. [TODO: confirm `ufw`/`iptables` rules on production VPS.]

## §10. Code review

1. Every PR runs `lint + typecheck + test` in CI.
2. PRs touching auth, crypto, audit, tenant boundaries, key handling, or network ingress MUST run the [`security-reviewer` subagent](https://github.com/pulkitpareek18/ZeroAuth/blob/main/.claude/agents/security-reviewer.md). Don't ask — just invoke.
3. PRs touching `circuits/`, `contracts/`, `src/services/zkp.ts`, `src/services/identity.ts`, or anywhere a hash/commitment scheme is introduced MUST run the [`cryptographer-reviewer` subagent](https://github.com/pulkitpareek18/ZeroAuth/blob/main/.claude/agents/cryptographer-reviewer.md).

## §11. Language we forbid in writing

To prevent overstating what ZeroAuth is, we NEVER use these phrases in code, comments, docs, commit messages, or marketing copy:

- "AI-powered" / "leveraging AI" — the verifier is cryptography, not AI
- "deepfake-immune" without the qualifier "at the visual spoofing class at the verification layer"
- "Dr. Pulkit" — Pulkit Pareek is "Senior Software Engineer"
- "production stack" — use "live reference implementation"

## §12. Changes to this policy

**Two-reviewer rule:** Pulkit + Amit. Both sign off on every change.

**External counsel review** required for any change to §3 (cryptographic primitives), §5 (data residency), §6 (audit logging), or §7 (vulnerability disclosure) — **once counsel is engaged**. Until then, those changes go through the founders' joint sign-off and the affected section retains the `PROVISIONAL` marker. See ADR-0005 (open) for the engagement target (~2026-07-01).

---

LAST_UPDATED: 2026-05-13
NEXT_REVIEW: 2026-08-13 (quarterly)
