# Threat model — Verifier service (`@zeroauth/verifier`)

> Extends [`canonical.md`](canonical.md).
> **Status:** v1 — promoted from stub on 2026-05-15 after the verifier service was wired into production via PRs [#35](https://github.com/zeroauth-dev/ZeroAuth/pull/35) (deploy), [#36](https://github.com/zeroauth-dev/ZeroAuth/pull/36) (healthcheck hotfix), [#37](https://github.com/zeroauth-dev/ZeroAuth/pull/37) (SQLite audit log + hash chain). Implementation language: TypeScript on `snarkjs` per [ADR-0006](https://github.com/zeroauth-dev/ZeroAuth/blob/main/adr/0006-verifier-typescript-not-rust.md) in the API repo.

## Component description

A standalone Express service running in its own Docker container (`zeroauth-verifier`) on the same Docker network as the API. Bound to `127.0.0.1:3001` on the network; no host port binding. Loopback-only is the trust boundary — the verifier authenticates nothing on its own; it trusts every request because only `zeroauth-prod` can reach it.

Exposes three endpoints to the API container:

- `POST /verify` — accepts a Groth16 proof + 3 public signals; returns `{ verified, verifierAuditId, latencyMs, circuitVersion, structuralFallback }`. Synchronous. Appends one row to the SQLite audit log per request.
- `GET /health` — version + vkey status + uptime + audit-log stats (`rowCount`, `nextSequence`, `lastEntryHashPrefix`). Used by the API's `/api/health` aggregate.
- `GET /audit/stats`, `GET /audit/verify-chain` — read-only introspection for ops + the evidence-pack assembler (W08).

Persistent state:

- SQLite WAL-mode database at `/app/data/audit.db` (Docker named volume `verifier-audit-data`). Append-only `verifier_events` table with hash-chained rows.
- Groth16 verification key file at `/app/circuits/build/verification_key.json` (read-only, baked into the image).

## Surfaces

| Surface | Exposure | Notes |
|---|---|---|
| `POST /verify` | Loopback only (Docker network, IP `zeroauth-verifier:3001`) | The single mutating route. Synchronous. Predictable latency. |
| `GET /health`, `/audit/*` | Loopback only | Read-only. Used by API health aggregation + ops tooling. |
| Verification key file | Read at startup, cached in memory; refused at startup if missing | Replacement requires container restart. |
| SQLite database file | Mounted as a Docker volume on the host | Single-instance write access via the verifier process; readable by `docker exec` for ops. |
| No outbound network calls | Process explicitly never makes egress requests | Confirmed by the dep tree audit (express + snarkjs + winston + uuid + better-sqlite3 — none make network calls in this code path). |

## Identified attacks (A-V-NN)

### A-V01 — Verifier audit-log tamper via direct SQLite write

| | |
|---|---|
| Class | Repudiation / tampering (STRIDE: R, T) |
| Surface | `/app/data/audit.db` on the verifier container's volume |
| Description | An attacker with `root` on the VPS could mount the volume, drop the append-only SQL triggers, and modify or delete rows. The verifier process itself can't be tricked into doing this because the triggers block UPDATE + DELETE at the engine level — the attacker has to bypass via direct file access. |
| Mitigation | **Hash chain on every row.** `entry_hash = sha256(canonical(row excl entry_hash) concat prev_hash)`. Any tamper breaks the chain. `GET /audit/verify-chain` walks the table and reports the first bad sequence + the reason (entry_hash mismatch vs prev_hash mismatch). |
| Detection | The chain check runs on demand today; a periodic cron should run it daily and alert on `ok: false`. **Open work item.** |
| Test status | `verifier/tests/audit-log.test.ts` has tamper-detection tests for both classes (entry_hash mismatch + prev_hash linkage mismatch) — drops the trigger via `DROP TRIGGER`, mutates a row, asserts verifyChain catches it. |
| Residual risk | **Medium-low.** Tamper is detectable but not preventable against an attacker with VPS root. Reduces to "we will know the tamper happened" not "the tamper can't happen." Cross-chain anchoring to Base L2 (planned v3) would harden this further. |

### A-V02 — Verification key swap on disk between deploys

| | |
|---|---|
| Class | Spoofing / tampering (STRIDE: S, T) |
| Surface | `/app/circuits/build/verification_key.json` inside the verifier image |
| Description | An attacker who can rebuild + push a new container image (e.g. compromised GitHub Actions deploy key) could swap in a vkey for which they have the corresponding zkey, then mint proofs that pass verification despite not knowing the patent-genuine secret. |
| Mitigation | (a) The image is built from the committed `circuits/build/verification_key.json` via the `verifier-build` Dockerfile stage — any swap requires a git push, which is reviewed. (b) The vkey is also baked into the build hash; running `docker images zeroauth-verifier:latest --format '{{.ID}}'` before and after a deploy lets ops detect unexpected changes. (c) **Open work item:** sign the vkey at trusted-setup time and verify the signature at verifier startup; refuse to start on signature mismatch. |
| Test status | No automated test today. Hard to test without infrastructure (would need a known-bad vkey + a sandboxed verifier). Tracked as a Week 7 evidence-pack task. |
| Residual risk | **Medium.** The supply-chain story is "trust the GitHub Actions deploy key and the committed vkey." Acceptable for v0; gets harder when the team grows past 1 engineer. |

### A-V03 — Side-channel attacks on the snarkjs verifier

| | |
|---|---|
| Class | Information disclosure (STRIDE: I) |
| Surface | The `snarkjs.groth16.verify` call path inside the verifier process |
| Description | Timing variations in field-arithmetic operations could leak information about either the proof or the verification key. Cache-timing attacks against a shared-CPU host (the VPS is shared with no co-tenants today, but cloud move could change that) could extract the vkey bit-by-bit. |
| Mitigation | snarkjs is JavaScript; it does NOT have constant-time field operations. **The mitigation today is that the verifier is loopback-only** — only the API container talks to it, and proofs come from authenticated tenant API keys that already have to bypass the rate limiter. A timing attack would need ~tens of thousands of requests, which is impractical against a single-tenant rate-limited surface. |
| Test status | Not tested. Side-channel resistance requires specialized tooling (FLUSH+RELOAD, etc.) that's out of scope for v0. |
| Residual risk | **Medium.** Acceptable for the current single-VPS shape. A multi-region deployment (Week 9+) reopens this; revisit then. Plan A's Rust + arkworks would have moved this from "acceptable" to "very low" because arkworks ships constant-time field implementations. We accepted the trade-off in ADR-0006. |

### A-V04 — Resource exhaustion via crafted proof inputs

| | |
|---|---|
| Class | Denial of service (STRIDE: D) |
| Surface | `POST /verify` request body deserialization + the snarkjs Groth16 verify path |
| Description | A crafted proof or oversized publicSignals could trigger excessive CPU (verification is the latency-critical path; the per-call cost is ~25ms steady-state) or memory (the JSON body parser allows up to 128KB; a proof envelope is normally <2KB). |
| Mitigation | (a) `express.json({ limit: '128kb' })` caps the request body. (b) Input validation rejects non-shape requests at 400 before they reach the verifier (missing proof, publicSignals length ≠ 3, non-array). (c) The API container's rate limiter (`authenticateTenantApiKey` middleware) is upstream of the verifier — the verifier sees no traffic unless the caller has a valid tenant API key. |
| Test status | Shape rejection covered by `verifier/tests/server.test.ts`. Resource-exhaustion fuzzing covered by `tests/property/` (proptest) **— TODO, not yet written.** |
| Residual risk | **Low.** Single-VPS scale is not the constraint that makes this attack practical. |

### A-V05 — Cross-tenant verification via spoofed `tenantId` in `/verify` request

| | |
|---|---|
| Class | Spoofing / Elevation of privilege (STRIDE: S, E) |
| Surface | `POST /verify` request body field `tenantId` |
| Description | The verifier records the `tenantId` from the request body into the audit log. The API container is the only sanctioned caller and *should* pass its own tenant context, but if a bug in `src/services/zkp.ts` ever forwarded a client-supplied tenant id, the audit log would record it as truth without re-validating. The verifier has no auth, so it can't tell. |
| Mitigation | (a) Trust boundary is the Docker network — only the API container can reach `127.0.0.1:3001`, so a malicious tenant can't directly spoof the verifier. (b) The API container's `verifyViaService` function in `src/services/zkp.ts` constructs the verifier request from server-side state (the resolved `tenantContext`), not from the client-side proof envelope. (c) **Long-term:** add a shared-secret HMAC header on every verifier request from the API, verify it in the verifier middleware, refuse unsigned requests. Roadmap item. |
| Test status | Not yet tested. Requires a unit test that asserts `verifyViaService` never reads tenantId from the API's request body. |
| Residual risk | **Low** — the trust boundary is the load-bearing mitigation, and that's enforced by Docker networking. The audit-log record is a forensic concern: if it ever misattributes a verification to the wrong tenant, the chain still validates (entry_hash is computed from whatever's in the row), but the forensic chain becomes misleading. |

## Open items (not yet `A-V-NN`)

- **Periodic chain verification cron** — `GET /audit/verify-chain` exists but isn't run on a schedule. Should run daily + alert on failure. Pre-evidence-pack-publish should also run it.
- **Off-host backup of the SQLite audit log** — the `verifier-audit-data` Docker volume isn't backed up. A VPS disk failure or accidental `docker volume rm` deletes the entire verifier history. Add a nightly `sqlite3 .backup` snapshot pushed to an off-host bucket.
- **Reproducible build provenance** — better-sqlite3's source build via node-gyp is non-deterministic on alpine arm64-musl (no prebuilt binary). ADR-0006 accepts this trade-off vs Plan A's Rust+arkworks story, but for the evidence pack, we should at least pin the `apk` package versions in the Dockerfile so the build inputs are stable across CI runs.
- **vkey signature at trusted-setup time** — A-V02's most-impactful mitigation. The Powers of Tau ceremony output gets a signature; the verifier verifies it at startup; refuses to start on mismatch.
- **Constant-time Groth16 verify** — A-V03's mitigation if/when we go multi-region. Either swap to a constant-time JS library, or move to Plan A (Rust + arkworks).
- **HMAC on API→verifier requests** — A-V05's deeper mitigation. Tracked.

## Operational notes

- The verifier's Docker container restarts pick up audit-log state from the persistent volume — no reinitialization, no genesis row rewrite. The hash chain spans container lifetimes.
- The verification key file is COPYed into the image at build time; runtime swap requires `docker compose --profile prod up -d --build --force-recreate zeroauth-verifier` + the new vkey checked into the repo.
- The SQLite database grows ~250 bytes per verification. At 10K verifications/day (pilot-scale) that's ~2.5 MB/day, ~1 GB/year. Sustainable without pruning for several years. When pruning becomes necessary, the design is "snapshot then truncate" — a snapshot becomes the new genesis row.

## Pairs with

- API-repo's [`src/services/zkp.ts`](https://github.com/zeroauth-dev/ZeroAuth/blob/main/src/services/zkp.ts) — the HTTP client that calls into this verifier.
- API-repo's [`verifier/`](https://github.com/zeroauth-dev/ZeroAuth/tree/main/verifier) workspace — the verifier source.
- [ADR-0006](https://github.com/zeroauth-dev/ZeroAuth/blob/main/adr/0006-verifier-typescript-not-rust.md) — language + architecture decision.
- [Verifier design doc](https://github.com/zeroauth-dev/ZeroAuth/blob/main/docs/design/verifier-service-split.md) — full migration plan.
- [`canonical.md`](canonical.md) — cross-repo threat model. A-02 (replayed proof verification) primary mitigation lives in the verifier today.
- [`api.md`](api.md) — the API-side component threat model. A-02's API-side responsibility (timestamp window + nonce format) stays there.

---

LAST_UPDATED: 2026-05-15
NEXT_REVIEW: after the first pilot SOW signing OR after any A-V finding upgrade to High/Critical
