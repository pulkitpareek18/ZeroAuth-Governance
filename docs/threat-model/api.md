# Threat model — API component (`zeroauth-dev/ZeroAuth`)

> Extends [`canonical.md`](canonical.md). When this file and the canonical disagree, the canonical wins; update this file.
> **Last reviewed by:** Pulkit Pareek on 2026-05-13

Most attacks in the canonical apply primarily to this component (the API is the broadest attack surface today). This file holds component-specific mitigation detail.

## A-01 — Cross-tenant data read

**Mitigation in this repo:** `src/middleware/tenant-auth.ts` resolves the tenant from the API key on every request and sets `(req as any).tenantContext = { tenantId, environment }`. Every service-layer function in `src/services/platform.ts` takes those as parameters and embeds them in the SQL WHERE. Express middleware augmentation is planned (see [`zeroauth-dev/ZeroAuth: CLAUDE.md`](https://github.com/zeroauth-dev/ZeroAuth/blob/main/CLAUDE.md) — "until we ship Express module augmentation").

**Test coverage:** `tests/central-api.test.ts` exercises the scoping at the router layer.

**Component-specific residual:** SQL injection in a future raw-SQL handler would bypass the middleware. Mitigated by using `pg` parameterized queries everywhere; CI lint rule (TODO) should reject string-concat SQL.

## A-02 — Replayed proof verification

**Mitigation in this repo:** `src/services/zkp.ts` validates a 5-minute timestamp window and nonce format on `POST /v1/auth/zkp/verify`.

**Component-specific gap:** Issued-nonce binding table doesn't exist. Open issue: create `issued_nonces (nonce, tenant_id, environment, issued_at, consumed_at)` with a constraint that `consumed_at` can only transition NULL → timestamp.

**Test coverage:** `tests/zkp.test.ts` covers timestamp window + nonce format. Missing: within-window replay test.

## A-03 / A-04 — Forged SAML / OIDC assertions via demo callback

**Mitigation in this repo:** `src/middleware/demo-auth-gate.ts` returns 503 unless `ENABLE_DEMO_AUTH=true`. Flag is **off** in production (`/opt/zeroauth/.env` on VPS has no such variable).

**Component-specific verification:** `curl https://zeroauth.dev/api/auth/saml/callback` should return 503. Confirmed during May 2026 review.

**Re-enable procedure:** before flipping `ENABLE_DEMO_AUTH=true` in production, the routes MUST be replaced with `@node-saml/node-saml` (SAML) and `openid-client` (OIDC). Both replacements require their own ADR + threat-model entry.

## A-05 — Credential stuffing on console signup

**Mitigation in this repo:** `src/routes/console.ts:authLimiter` — `express-rate-limit` with 10 attempts / 15 min / IP on `/api/console/signup` and `/api/console/login`. Password policy in `src/services/tenants.ts:validatePassword` — ≥12 chars, ≥1 letter + ≥1 digit, denylist of common passwords.

**Component-specific gap:** Limiter is skipped under `NODE_ENV=test` (intentional, but test for the 429 must flip that). Open issue. Email enumeration via status code: signup returns 409 on existing email vs 201 on new — leaks. Long-term: return 201-with-no-action on existing-email signup; send the discrepancy via email (out-of-band).

## A-06 — Replay of revoked API key

**Mitigation in this repo:** `authenticateApiKey` in `src/middleware/tenant-auth.ts` re-reads `api_keys` table on every request. Rejected if `status != 'active'`. In-memory rate-limit counter sharing across keys of the same tenant is by design.

## A-09 — Console JWT theft via dashboard XSS

**Mitigation in this repo:** Helmet CSP — `default-src 'self'`; `script-src 'self' 'unsafe-inline'` (TODO: remove unsafe-inline for dashboard once we audit landing-page inline scripts); `script-src-attr 'none'`; `frame-ancestors 'self'`; `object-src 'none'`.

The dashboard React build (`dashboard/dist/`) should contain no inline `<script>` blocks beyond the Vite-emitted entrypoint shim. Integration test: TODO.

## A-10 — Dashboard tenant inference from request

**Mitigation in this repo:** Every console route reads `tenantId` from `(req as any).console.tenantId` (set by `verifyConsoleToken`), never from body/query. Reviewer-enforced today.

**Component-specific gap:** No automated probe test. Open issue: write a Jest test that constructs a console JWT for tenant A, probes every `/api/console/*` route with `{ tenantId: <B-id> }` in body and query, asserts the response is scoped to A or 403.

## Surfaces unique to this component

- `POST /api/leads/{pilot,whitepaper}` — public, unauth, writes to `leads` table. Currently NO rate limit on this endpoint. Open: A-11 (lead-form abuse) — should be added.

---

LAST_UPDATED: 2026-05-13
