# Compliance mapping — DPDP Act 2023 + DPDP Rules (Draft 2025)

> **Last reviewed by:** Pulkit Pareek on 2026-05-13
> **Status:** v1 — **PROVISIONAL**. The mapping below is founders' best-effort interpretation of the statute; external counsel review is required before any of it can be represented to a tenant or regulator as authoritative. Counsel engagement is open (ADR-0005, target ~2026-07-01). Some clauses map to roadmap items rather than implemented controls; these are marked `Partial — see roadmap`.

This table maps clauses of the Digital Personal Data Protection Act 2023 (DPDP Act) and the draft DPDP Rules 2025 to specific ZeroAuth controls.

ZeroAuth's role: **Data Processor.** The tenant (e.g., HDFC Life, Star Health) is the Data Fiduciary. End-users of the tenant's app are Data Principals.

## Mapping

| DPDP clause | Topic | ZeroAuth control | Status |
|---|---|---|---|
| §4(2) | Lawful basis for processing | We process on behalf of the Data Fiduciary; their lawful basis applies. Our processing is governed by the Data Processing Agreement with each tenant. | **Implemented** (DPA template — TODO) |
| §6 | Consent | We do not collect consent from Data Principals directly. The tenant manages consent and surfaces the verification flow to their users. | **Implemented** (architectural — we receive only commitments) |
| §7 | Legitimate uses without consent | n/a — we process only on instruction from the Data Fiduciary | **n/a at processor level** |
| §8(4) | Reasonable security safeguards | See [`../shared/security-policy.md`](../shared/security-policy.md) §§2, 3, 4, 9, 10. TLS 1.3, SHA-256 + Poseidon hashing, tenant-scoped queries, structured audit log. | **Implemented** |
| §8(5) | Notify the Board of personal data breach | See [`../shared/breach-notification.md`](../shared/breach-notification.md). 72-hour clock from confirmation. | **Implemented** (procedure exists; not yet drilled) |
| §8(6) | Notify affected Data Principals | We notify the tenant (Data Fiduciary) within 6h; the tenant notifies their Data Principals. | **Implemented** (procedure exists) |
| §8(7) | Manner of notification | See breach-notification.md §4 — DPBI form via published portal. | **Implemented** (procedure pending counsel review) |
| §9 | Children's personal data | We do not knowingly process children's data. Tenants are contractually required to gate their flows. | **Partial** — no programmatic gate at ZeroAuth layer |
| §10 | Significant Data Fiduciary (SDF) obligations | If any tenant is designated an SDF, additional controls apply (DPIA, audit, DPO). Tracked per-tenant. | **Partial — see roadmap** |
| §11 | Right to access | We expose `/v1/audit` for tenant-scoped audit log; tenant relays to Data Principal. | **Implemented** |
| §12 | Right to correction / erasure | Tenant deletes via `DELETE /v1/users/:userId`; cascades to verifications + commitments. Audit-log rows for the user remain (immutable). | **Implemented** |
| §13 | Right of grievance redressal | We do not face Data Principals directly. Tenant handles. | **n/a at processor level** |
| §16 | Cross-border transfer | Data stays in `ap-south-1` (India) per [`../shared/security-policy.md` §5](../shared/security-policy.md). | **Partial** — need to verify VPS region and lock in via ADR |
| §17 | Exemptions | n/a | **n/a** |
| §27 | Powers of the Data Protection Board | We cooperate per breach-notification.md and DPDP rules | **Implemented** (procedure) |
| §28 | Inquiry by the Board | We retain audit logs 7 years per [`../shared/security-policy.md` §6.3](../shared/security-policy.md) | **Implemented** |
| §33 | Penalties | n/a — this is the regulator's side | **n/a** |
| §39 | Power to make rules | We track rule updates and re-review this mapping quarterly. Next review: 2026-08-13. | **Implemented** (process) |

## DPDP Rules (Draft 2025) — placeholder mapping

The draft DPDP Rules expand operational requirements for breach notification, consent management, and Significant Data Fiduciary designation. Mapping pending final rule publication. **Open item:** counsel-led review when final rules are notified.

## Open items

- **External DPO counsel engagement** (ADR-0005, target ~2026-07-01) — gates every "pending counsel review" line above.
- DPA template with tenants is not standardized. Each pilot SOW currently includes ad-hoc DPA clauses. Standardize before scaling (requires counsel).
- Data residency lock-in via ADR (production VPS region).
- Automated programmatic age-gate (§9) at ZeroAuth layer — currently relies on tenant.
- DPIA template for SDF tenants.

---

LAST_UPDATED: 2026-05-13
NEXT_REVIEW: 2026-08-13 (quarterly) or upon DPDP Rules 2025 publication or counsel engagement, whichever fires first
