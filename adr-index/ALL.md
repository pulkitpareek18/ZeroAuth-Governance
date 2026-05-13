# ZeroAuth ADR Index — All Repos

> **Last reviewed by:** Pulkit Pareek on 2026-05-13

This is the cross-repo index of every Architecture Decision Record. Each entry links to the canonical ADR in its product repo. When you add a new ADR anywhere, add a row here.

## Format

| # | Title | Status | Repo | Path | Date |
|---|---|---|---|---|---|

## Index

| # | Title | Status | Repo | Path | Date |
|---|---|---|---|---|---|
| 0001 | CLAUDE.md as the project constitution + prompt-suite engineering discipline | Accepted | `pulkitpareek18/ZeroAuth` | [adr/0001-prompt-suite-engineering-discipline.md](https://github.com/pulkitpareek18/ZeroAuth/blob/main/adr/0001-prompt-suite-engineering-discipline.md) | 2026-05-12 |
| 0002 | Dashboard stack — Vite, not Next.js | Accepted | `pulkitpareek18/ZeroAuth` | [adr/0002-dashboard-stack-vite-not-nextjs.md](https://github.com/pulkitpareek18/ZeroAuth/blob/main/adr/0002-dashboard-stack-vite-not-nextjs.md) | 2026-05-12 |
| 0003 | Adopt Playwright for dashboard E2E | Accepted | `pulkitpareek18/ZeroAuth` | [adr/0003-adopt-playwright-for-e2e.md](https://github.com/pulkitpareek18/ZeroAuth/blob/main/adr/0003-adopt-playwright-for-e2e.md) | 2026-05-12 |
| 0004 | Governance lives in a separate repo (B06 — split from API repo) | Accepted | `pulkitpareek18/ZeroAuth` | [adr/0004-governance-in-separate-repo.md](https://github.com/pulkitpareek18/ZeroAuth/blob/main/adr/0004-governance-in-separate-repo.md) | 2026-05-13 |

## Status legend

- **Proposed** — under discussion, not yet binding
- **Accepted** — current law
- **Deprecated** — superseded but kept for historical context (link to the replacement)
- **Rejected** — considered, not adopted (kept for "why we didn't do X")

## Open ADR candidates (to be filed)

- ADR-0005 — Adopt zod for input validation (deferred from B01 quality bar)
- ADR-0006 — Adopt Prisma (or equivalent) for schema migrations (deferred from B01 quality bar)
- ADR-0007 — Verifier service split-out from `src/services/zkp.ts` (B02, Week 2)
- ADR-0008 — Multisig owner on `DIDRegistry` contract (replaces A-07 single-key risk)
- ADR-0009 — Off-host Postgres backup destination + cadence
- ADR-0010 — At-rest encryption (AES-256-GCM) for production Postgres volumes

---

LAST_UPDATED: 2026-05-13
