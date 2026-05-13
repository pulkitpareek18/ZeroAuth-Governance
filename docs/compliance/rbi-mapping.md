# Compliance mapping — RBI Cyber Security Framework

> **Last reviewed by:** Pulkit Pareek on 2026-05-13
> **Status:** v1 — DRAFT. Applies when a tenant is an RBI-regulated entity (bank, NBFC, PSP). Counsel review pending.

This file maps the RBI Cyber Security Framework (Master Direction on Information Technology and Cyber Security) to ZeroAuth controls. ZeroAuth's role: **third-party service provider** to the regulated entity.

## Applicability

RBI requires regulated entities to oversee their service providers' cyber security postures. The regulated entity (tenant) is responsible; ZeroAuth provides the evidence pack and the cooperation needed for the entity's annual cyber audit.

## Mapping (level-1 framework — populate level-2 with counsel)

| RBI domain | Requirement | ZeroAuth control | Status |
|---|---|---|---|
| **Governance** | Board-approved cyber security policy | See [`../shared/security-policy.md`](../shared/security-policy.md). Reviewed by Pulkit + Amit; counsel review pending. | **Partial — counsel review pending** |
| **Risk management** | Documented risk assessment, updated annually | Threat model in [`../threat-model/canonical.md`](../threat-model/canonical.md). Updated quarterly minimum. | **Implemented** |
| **Access management** | Role-based, principle of least privilege, periodic review | Tenant API key scopes; admin x-api-key separate; CODEOWNERS; periodic review **— TODO: define cadence** | **Partial** |
| **Encryption** | At rest + in transit | TLS 1.3 in transit (Caddy). At rest: Postgres column encryption for sensitive fields **— TODO: enable AES-256-GCM on disk**. | **Partial** |
| **Network security** | Segmentation, firewall, IDS | UFW on VPS, Docker network isolation, no Postgres / Redis exposed to internet. IDS: TODO. | **Partial — IDS pending** |
| **Application security** | SDLC, code review, secure coding | ESLint, TypeScript strict, security-reviewer subagent invocation on PRs. CI gates merge. | **Implemented** |
| **Audit log** | Tamper-evident, retained ≥ 7 years | `audit_events` table. Retention: 7 years. Hash-chained rows on roadmap. | **Partial — hash chain pending** |
| **Incident response** | Documented runbook, regulator notification within 2–6 hours of confirmation depending on entity class | See [`../shared/incident-response.md`](../shared/incident-response.md) + [`../shared/breach-notification.md`](../shared/breach-notification.md) §5. | **Implemented — drill pending** |
| **Business continuity** | DR, RTO/RPO documented, drilled | RTO 4h, RPO 24h (target). Drill pending. | **Partial** |
| **Third-party risk** | Due diligence + ongoing oversight on every vendor | DP6 (every dep is an ADR). Annual review of major deps. | **Implemented** |
| **Vulnerability management** | Scanning + timely patching | Dependabot. DW03 weekly dep drift watcher planned. CVSS-based SLA: critical < 7 days, high < 30 days. | **Partial — DW03 pending** |
| **Cyber crisis management** | Communication tree + escalation matrix | Contact tree in [`../shared/incident-response.md`](../shared/incident-response.md). | **Implemented** |
| **Information sharing** | Cooperation with CERT-In + RBI cyber security cell | CERT-In notification embedded in breach-notification.md §5. | **Implemented (procedure)** |

## Specific timelines

- **Incident notification to RBI:** 2–6 hours depending on entity classification (basic / common / IRAC / TRAC / TRAC+). Captured in breach-notification.md §5.
- **Annual cyber audit:** the regulated entity (tenant) audits ZeroAuth as part of their cyber audit. ZeroAuth provides the evidence pack (W08 from operational suite) and cooperation.

## Open items

- AES-256-GCM at-rest encryption on production Postgres.
- IDS on the VPS (Wazuh? Falco? minimum: `fail2ban` on SSH).
- DR drill scheduled.
- Access review cadence — quarterly?
- Counsel review of the entire mapping.

---

LAST_UPDATED: 2026-05-13
NEXT_REVIEW: counsel review + after first RBI-regulated tenant pilot signs
