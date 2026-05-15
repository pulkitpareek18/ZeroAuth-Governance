# ZeroAuth ADR Index — All Repos

> **Last reviewed by:** Pulkit Pareek on 2026-05-13

This is the cross-repo index of every Architecture Decision Record. Each entry links to the canonical ADR in its product repo. When you add a new ADR anywhere, add a row here.

## Format

| # | Title | Status | Repo | Path | Date |
|---|---|---|---|---|---|

## Index

| # | Title | Status | Repo | Path | Date |
|---|---|---|---|---|---|
| 0001 | CLAUDE.md as the project constitution + prompt-suite engineering discipline | Accepted | `zeroauth-dev/ZeroAuth` | [adr/0001-prompt-suite-engineering-discipline.md](https://github.com/zeroauth-dev/ZeroAuth/blob/main/adr/0001-prompt-suite-engineering-discipline.md) | 2026-05-12 |
| 0002 | Dashboard stack — Vite, not Next.js | Accepted | `zeroauth-dev/ZeroAuth` | [adr/0002-dashboard-stack-vite-not-nextjs.md](https://github.com/zeroauth-dev/ZeroAuth/blob/main/adr/0002-dashboard-stack-vite-not-nextjs.md) | 2026-05-12 |
| 0003 | Adopt Playwright for dashboard E2E | Accepted | `zeroauth-dev/ZeroAuth` | [adr/0003-adopt-playwright-for-e2e.md](https://github.com/zeroauth-dev/ZeroAuth/blob/main/adr/0003-adopt-playwright-for-e2e.md) | 2026-05-12 |
| 0004 | Governance lives in a separate repo (B06 — split from API repo) | Accepted | `zeroauth-dev/ZeroAuth` | [adr/0004-governance-in-separate-repo.md](https://github.com/zeroauth-dev/ZeroAuth/blob/main/adr/0004-governance-in-separate-repo.md) | 2026-05-13 |

## Status legend

- **Proposed** — under discussion, not yet binding
- **Accepted** — current law
- **Deprecated** — superseded but kept for historical context (link to the replacement)
- **Rejected** — considered, not adopted (kept for "why we didn't do X")

## Open ADR candidates (to be filed)

- **ADR-0005 — Engage external DPO + IP counsel.** Hard deadline: before first pilot SOW signing (~2026-07-01). Blocks shipping the breach-notification procedure, DPDP/IRDAI/RBI compliance mappings, and DPA templates as authoritative to buyers. On the W05 Friday review agenda. Decision needed: which firm (specialist boutique like Ikigai Law / Spice Route / TRA Law, or a full-service firm with DPDP practice), retainer scope, monthly cost ceiling.
- ADR-0006 — Adopt zod for input validation (deferred from B01 quality bar)
- ADR-0007 — Adopt Prisma (or equivalent) for schema migrations (deferred from B01 quality bar)
- ADR-0008 — Verifier service split-out from `src/services/zkp.ts` (B02, Week 2)
- ADR-0009 — Multisig owner on `DIDRegistry` contract (replaces A-07 single-key risk)
- ADR-0010 — Off-host Postgres backup destination + cadence
- ADR-0011 — At-rest encryption (AES-256-GCM) for production Postgres volumes

---

LAST_UPDATED: 2026-05-13
