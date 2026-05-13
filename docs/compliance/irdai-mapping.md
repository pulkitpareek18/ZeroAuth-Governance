# Compliance mapping — IRDAI Information & Cyber Security Guidelines

> **Last reviewed by:** Pulkit Pareek on 2026-05-13
> **Status:** v1 — **PROVISIONAL**. Applies when a tenant is an IRDAI-regulated entity (insurer / intermediary). External counsel review is required before this mapping can be represented to a tenant or regulator as authoritative. Counsel engagement is open (ADR-0005, target ~2026-07-01).

This file maps IRDAI's Information and Cyber Security Guidelines (the most recent revision applies — verify version against IRDAI portal) to specific ZeroAuth controls. ZeroAuth's role: **outsourced service provider** to the regulated insurer.

## Applicability

IRDAI's outsourcing framework requires the insurer (the tenant) to ensure their outsourced service providers meet specified cyber security standards. The insurer is responsible; ZeroAuth provides the evidence pack.

## Mapping

| IRDAI control area | What's required | ZeroAuth control | Status |
|---|---|---|---|
| Information classification | Identify and classify sensitive data | We process biometric **commitments** (cryptographic, not raw data) + audit metadata. Classification: Confidential (commitments), Internal (audit metadata). | **Implemented** |
| Cryptographic controls | Use industry-standard cryptography | Groth16 over BN128, Poseidon hashing, SHA-256, Ed25519, secp256k1, TLS 1.3. See [`../shared/security-policy.md` §3](../shared/security-policy.md). | **Implemented** |
| Access controls | Role-based, principle of least privilege | Tenant API key scopes (`zkp:verify`, `devices:write`, etc.); admin x-api-key separate. CODEOWNERS for repo write access. | **Implemented** |
| Audit logging | Comprehensive, tamper-evident, retained ≥ 7 years | `audit_events` table, append-only at application layer; retention policy in security-policy.md §6.3. Hash-chained rows on roadmap. | **Partial — hash chain pending** |
| Incident response | Documented runbook, drilled, regulator notification within 6 hours of confirmation | See [`../shared/incident-response.md`](../shared/incident-response.md). | **Implemented — drill pending** |
| Business continuity | Backup, recovery, DR plan | Postgres + Redis in Docker; daily backups to (TODO: off-host destination). DR: rebuild from CI artifacts + DB restore. RTO 4h, RPO 24h. | **Partial — off-host backup pending** |
| Third-party risk management | Vendor due diligence on every dep | DP6 (every dep is an ADR) enforced via `dep-add` skill. `scripts/check-dep-trail.sh` audits the lockfile against `/adr/`. | **Implemented** |
| Vulnerability management | Periodic scanning, timely patching | Dependabot enabled on `pulkitpareek18/ZeroAuth`. DW03 (weekly dep drift watcher) planned. | **Partial — DW03 pending** |
| Cyber drill | Periodic | Drill cadence in breach-notification.md §9 (semi-annual). First drill: 2026-08. | **Pending — first drill 2026-08** |
| Vendor exit clause | Customer data returnable on contract end | Tenant can `GET /v1/audit?export=full` at any time. Account closure procedure: TODO. | **Partial** |

## Specific clauses (sample — populate from current IRDAI Guidelines)

- Sub-section on biometric data — IRDAI Guidelines reference. [TODO: cite specific section + sub-clause once counsel confirms the active version.]
- Sub-section on outsourcing to overseas providers — n/a (we're India-based, data stays in `ap-south-1`).
- Sub-section on incident reporting timelines — 6h to IRDAI from confirmation. Embedded in breach-notification.md §5.

## Open items

- **External counsel engagement** (ADR-0005, target ~2026-07-01) — gates everything below that requires interpretation of the current IRDAI Guidelines version.
- Confirm current IRDAI Guidelines version applicable (2017 / 2023 revision / 2024 amendments) — needs counsel.
- Off-host Postgres backup destination.
- Standardize vendor exit / account closure procedure.
- DW03 dependency drift watcher operational.
- First cyber drill: schedule for 2026-08-XX.

---

LAST_UPDATED: 2026-05-13
NEXT_REVIEW: counsel review + after first IRDAI-regulated tenant pilot signs
