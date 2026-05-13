# Compliance mapping — MeitY guidelines + CERT-In directives

> **Last reviewed by:** Pulkit Pareek on 2026-05-13
> **Status:** v1 — DRAFT. Counsel review pending.

This file maps the Ministry of Electronics and Information Technology (MeitY) advisories and CERT-In directives applicable to ZeroAuth as an Indian-origin technology provider.

## Applicability

MeitY's advisories apply broadly to Indian intermediaries; specific directives (e.g., the April 2022 CERT-In directive on incident reporting + log retention) apply to all entities operating in India.

## Mapping

### CERT-In April 2022 directive (incident reporting + log retention)

| Requirement | ZeroAuth control | Status |
|---|---|---|
| Synchronize all system clocks to NTP server `time.nist.gov` or equivalent | VPS runs systemd-timesyncd against `time.nist.gov` and `pool.ntp.org`. **TODO: verify on production.** | **Partial** |
| Report listed cyber incidents to CERT-In within 6 hours of noticing | See [`../shared/breach-notification.md`](../shared/breach-notification.md) §5. | **Implemented (procedure)** |
| Retain ICT system logs for 180 days minimum | Audit logs retained 7 years (exceeds requirement). System logs (nginx / Caddy / Postgres / Docker) retained: **TODO — confirm retention on VPS**. | **Partial** |
| Categories of incidents to report (see directive Annexure I) | Embedded in incident-response.md §1 severity classification. | **Implemented** |
| KYC details of registered subscribers (for ISPs / data centers) | n/a — we are a SaaS provider, not an ISP / data center | **n/a** |

### MeitY advisory on biometric data handling (general guidance)

| Concern | ZeroAuth control | Status |
|---|---|---|
| Don't store raw biometric data | We accept commitments only. Raw biometric data is never accepted on `/v1/*` — middleware rejects any payload with keys `image`, `template`, `pixel`, `depth`, `frame`, `biometric_data`. See [`../shared/security-policy.md` §2.5](../shared/security-policy.md). | **Implemented** |
| Use industry-standard cryptography | Groth16, Poseidon, SHA-256, Ed25519, TLS 1.3. | **Implemented** |
| Consent management | Tenant's responsibility (Data Fiduciary in DPDP terminology). | **n/a at processor level** |

### Section 70B of the IT Act (CERT-In powers)

ZeroAuth complies with directions from CERT-In under §70B. Procedure: any direction is routed through DPO counsel; response coordinated by Amit + counsel within the directed timeframe.

## Specific clauses

- **CERT-In directive paragraph 4(iii)** — system logs retention 180 days: **need to verify and document VPS log retention.**
- **CERT-In directive paragraph 4(iv)** — incident reporting timelines per Annexure I.

## Open items

- Verify NTP sync configuration on production VPS.
- Document VPS system-log retention (rotate config in `/etc/logrotate.d/`).
- Confirm CERT-In contact channel: `incident@cert-in.org.in` per directive — verify current address.

---

LAST_UPDATED: 2026-05-13
NEXT_REVIEW: 2026-08-13 (quarterly) or upon any new MeitY / CERT-In directive
