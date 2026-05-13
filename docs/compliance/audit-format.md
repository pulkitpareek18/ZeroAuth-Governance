# Audit Event Format (platform-wide)

> **Last reviewed by:** Pulkit Pareek on 2026-05-13
> **Status:** v1

This file is the canonical schema for an audit event across every ZeroAuth component. Every component that writes audit data MUST conform.

## Storage

- Single Postgres table: `audit_events`
- Owner: API repo's `src/services/platform.ts:recordAuditEvent()`
- Other components (verifier, IoT, SDK) emit audit events by calling the API; they do NOT write directly to the table.

## Schema

```sql
CREATE TABLE audit_events (
  id              TEXT PRIMARY KEY,          -- monotonic increasing per-tenant; collision-free across tenants because also keyed by tenant_id
  tenant_id       UUID NOT NULL,             -- the tenant the event belongs to
  environment     TEXT NOT NULL,             -- 'live' | 'test'
  actor_type      TEXT NOT NULL,             -- 'tenant_api_key' | 'console_jwt' | 'admin' | 'system'
  actor_id        TEXT,                      -- nullable; the API key id, console JWT subject, or admin actor
  action          TEXT NOT NULL,             -- dotted verb-phrase: 'api_key.created', 'device.registered', 'verification.failed'
  entity_type     TEXT NOT NULL,             -- the type of entity acted on: 'api_key', 'device', 'user', 'verification', 'attendance'
  entity_id       TEXT,                      -- nullable; the specific entity affected
  status          TEXT NOT NULL,             -- 'success' | 'failure'
  summary         TEXT,                      -- one-line human readable summary
  metadata        JSONB,                     -- structured details: scopes, key prefix, error code, etc.
  created_at      TIMESTAMPTZ NOT NULL,      -- UTC, server time
  -- planned: chain_prev_hash TEXT NOT NULL,  -- the SHA-256 hash of the previous row in this tenant+env chain
  -- planned: chain_self_hash TEXT NOT NULL   -- SHA-256(serialized(this row, excluding chain_self_hash) || chain_prev_hash)
);

CREATE INDEX audit_events_tenant_env_created ON audit_events (tenant_id, environment, created_at DESC);
CREATE INDEX audit_events_entity ON audit_events (tenant_id, environment, entity_type, entity_id);
```

## Action vocabulary

Format: `<entity>.<verb>`. Past tense, snake_case.

| Action | Entity type | Status | Description |
|---|---|---|---|
| `api_key.created` | `api_key` | success | A new tenant API key was minted |
| `api_key.rotated` | `api_key` | success | An existing API key was rotated (old revoked, new created) |
| `api_key.revoked` | `api_key` | success | An API key was revoked |
| `device.registered` | `device` | success / failure | A device was registered to the tenant |
| `device.updated` | `device` | success | Device metadata was updated |
| `device.deleted` | `device` | success | Device was deleted |
| `user.registered` | `user` | success / failure | A user was registered (commitment stored) |
| `user.updated` | `user` | success | User metadata was updated |
| `user.deleted` | `user` | success | User was deleted; verifications cascade |
| `verification.succeeded` | `verification` | success | A proof was verified successfully |
| `verification.failed` | `verification` | failure | A proof was rejected (with reason in `metadata.reason`) |
| `attendance.recorded` | `attendance` | success | An attendance event was logged |
| `console.signup` | `tenant` | success | A new tenant signed up via the console |
| `console.login` | `tenant` | success / failure | A console login attempt |
| `console.logout` | `tenant` | success | A console logout |
| `tenant.suspended` | `tenant` | success | A tenant was suspended (admin action) |
| `tenant.reactivated` | `tenant` | success | A tenant was reactivated |
| `zkp.verify` | `verification` | success / failure | Legacy alias for `verification.{succeeded,failed}` — being phased out |
| `cross_tenant_query_blocked` | `system` | failure | Defensive: a cross-tenant query was attempted and the WHERE-clause guard fired. (Planned A-01 audit signal.) |
| `auth.token_reuse` | `console_jwt` | failure | Same `jti` replayed from a new IP within a short window. (Planned A-09 audit signal.) |

New actions require a PR that updates this file AND the canonical threat model (`canonical.md`).

## Required fields by actor type

| `actor_type` | `actor_id` content |
|---|---|
| `tenant_api_key` | UUID of the api_keys row that authenticated this request |
| `console_jwt` | UUID of the tenant operator (the JWT subject) |
| `admin` | the admin x-api-key fingerprint (first 12 chars of SHA-256), never the key itself |
| `system` | NULL — used for internal events (cleanup, retention sweeps) |

## Forbidden in `metadata`

- Plaintext API keys
- Plaintext JWTs
- Plaintext passwords / password hashes
- Biometric-derived raw data
- Full email addresses (use a SHA-256 hash if linking is required)
- Cross-tenant references (an audit row for tenant A must never include tenant B's identifiers)

## Retention

- **7 years** per [`../shared/security-policy.md` §6.3](../shared/security-policy.md).
- Soft-deletion on tenant deletion: rows are retained but tagged with `metadata.tenant_deleted_at`.

## Tamper-evidence (planned)

The hash chain (`chain_prev_hash`, `chain_self_hash` columns above, currently commented out) is planned for v2. The chain is keyed by `(tenant_id, environment)` — each tenant's audit log is its own chain.

Per-row hash:

```text
chain_self_hash = SHA-256(
  id || tenant_id || environment || actor_type || actor_id ||
  action || entity_type || entity_id || status || summary ||
  metadata || created_at || chain_prev_hash
)
```

A periodic job verifies the chain integrity; deviations alert the operator.

Cross-chain anchoring (to Base Sepolia or equivalent L2) is a v3 feature.

---

LAST_UPDATED: 2026-05-13
