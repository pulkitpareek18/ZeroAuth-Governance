# ZeroAuth Coding Standards

> **Last reviewed by:** Pulkit Pareek on 2026-05-13
> **Status:** v1 — initial draft. Covers TypeScript only today; Solidity, Circom, Swift, Kotlin, and Rust standards land when those repos materialize.

This file is the shared style baseline. Per-repo `CLAUDE.md` files may extend, never relax.

## §1. Languages and versions

- **TypeScript:** 5.x, `strict: true`. No `any` in exported signatures except localized `(req as any).tenantContext` until Express module augmentation lands.
- **Node.js:** 20 LTS in CI + production. `tsx` for dev.
- **Solidity:** 0.8.x. Hardhat for builds.
- **Circom:** 2.1.x. snarkjs for proving / verification.
- **Bash:** POSIX-portable scripts. `set -euo pipefail` at the top.

## §2. Module / import style

- ESM in new code (`import`/`export`). CommonJS only where a dependency forces it.
- No deep imports across module boundaries; export through the package's `index.ts`.
- No circular imports. CI fails on circular dep detection.

## §3. Error handling

- API handlers return `{ error: '<machine_code>', message: '<human readable>' }` with appropriate HTTP status.
- The machine codes are enumerated in [`pulkitpareek18/ZeroAuth: docs/error_codes.md`](https://github.com/pulkitpareek18/ZeroAuth/blob/main/docs/error_codes.md). New error codes get added there before being thrown.
- Stack traces never leak to API responses. Winston JSON log captures them server-side only.
- Never swallow errors silently. `catch (e) { /* nothing */ }` is a CI failure (lint rule).

## §4. Logging

- Structured JSON via Winston (TypeScript repos).
- Required fields on every line: `level`, `time`, `service`, `tenant_id` (if scoped), `environment` (if scoped), `request_id`, `event`.
- **Never log:** API key plaintext, JWT plaintext, biometric-derived raw data, customer PII outside the tenant_id pointer.
- Log levels: `debug` for dev only; `info` for state changes; `warn` for recoverable; `error` for failures; `fatal` for unrecoverable.

## §5. Tests

- Every endpoint has a request-level test before the implementation ships. Write the "wrong tenant rejected" test before "right tenant accepted."
- Unit tests live next to the file (`foo.ts` + `foo.test.ts`) for new code; legacy tests in `tests/*.test.ts`.
- Test framework: Jest 29 for backend, Vitest 3 for the dashboard, Playwright 1.60+ for E2E.
- CI gates merge on `npm test` passing across all suites.

## §6. Naming

- Variables: `camelCase`. Constants: `SCREAMING_SNAKE_CASE`. Types/classes: `PascalCase`. Files: `kebab-case.ts` or `camelCase.ts` consistent with surrounding files.
- API surfaces use `snake_case` for JSON keys (e.g. `tenant_id`, `external_id`) to match the database column convention.
- See [`naming-conventions.md`](naming-conventions.md) for the full env-var / service-name / log-field naming rules.

## §7. Comments

- Comment the *why*, not the *what*. The code is the what.
- Top-of-file comments explain the file's role + when to touch it.
- TODO comments include a date and an issue link: `// TODO(2026-05-13, #42): pull this into a service.`

## §8. Commits

- Plain English subject + body explaining why. Conventional Commits is NOT enforced.
- Keep commits small. One logical change per commit.
- Reference the threat-model attack number (`A-NN`) when a commit implements or modifies a mitigation.

## §9. PR discipline

- Open PRs as **draft** when work is in progress; mark **ready for review** when CI is green.
- Use plan mode for any change touching 5+ files OR any of the auth/crypto/audit/tenant boundary paths.
- Every PR description has a "How tested" section and a "Risk" section.

## §10. Dependencies

Adding any direct dependency requires an ADR. Use the `dep-add` skill ([`pulkitpareek18/ZeroAuth: .claude/skills/dep-add/SKILL.md`](https://github.com/pulkitpareek18/ZeroAuth/blob/main/.claude/skills/dep-add/SKILL.md)). No exceptions.

## §11. What we DO NOT do

- No `npm install` without going through `dep-add`
- No `--force` pushes to `main`
- No CI bypass (`--no-verify`, `[skip ci]`)
- No raw biometric handling — ever
- No "Dr. Pulkit", no "AI-powered", no "production stack" (see security-policy.md §11)

---

LAST_UPDATED: 2026-05-13
NEXT_REVIEW: 2026-08-13 (quarterly)
