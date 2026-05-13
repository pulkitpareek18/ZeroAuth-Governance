# Evidence Pack Source Checksums

> **Last reviewed by:** Pulkit Pareek on 2026-05-13

This file records SHA-256 checksums of every source file that feeds the SOC 2 / DPDP / pilot-buyer evidence pack. The W08 evidence-pack assembler from the operational suite reads this file as ground truth.

When a source file changes, the checksum updates here (via the `evidence-pack-update` skill — to be installed). When a buyer asks "what was in the evidence pack at SOW signing," we look up the row in `RELEASES.md` and the checksums here.

## How to update

```bash
# When a source file changes, regenerate its row:
shasum -a 256 docs/shared/security-policy.md
# Update the corresponding row below.
```

CI runs `npm run check-checksums` and fails the build if any tracked file's actual SHA-256 differs from its row here.

## Tracked sources

### Shared policy

| File | SHA-256 | Last updated |
|---|---|---|
| `docs/shared/security-policy.md` | _pending — populate after first commit_ | 2026-05-13 |
| `docs/shared/coding-standards.md` | _pending_ | 2026-05-13 |
| `docs/shared/naming-conventions.md` | _pending_ | 2026-05-13 |
| `docs/shared/incident-response.md` | _pending_ | 2026-05-13 |
| `docs/shared/breach-notification.md` | _pending_ | 2026-05-13 |

### Threat model

| File | SHA-256 | Last updated |
|---|---|---|
| `docs/threat-model/canonical.md` | _pending_ | 2026-05-13 |
| `docs/threat-model/api.md` | _pending_ | 2026-05-13 |
| `docs/threat-model/verifier.md` | _pending_ | 2026-05-13 |
| `docs/threat-model/iot.md` | _pending_ | 2026-05-13 |
| `docs/threat-model/sdk.md` | _pending_ | 2026-05-13 |
| `docs/threat-model/dashboard.md` | _pending_ | 2026-05-13 |

### Compliance mappings

| File | SHA-256 | Last updated |
|---|---|---|
| `docs/compliance/dpdp-mapping.md` | _pending_ | 2026-05-13 |
| `docs/compliance/irdai-mapping.md` | _pending_ | 2026-05-13 |
| `docs/compliance/rbi-mapping.md` | _pending_ | 2026-05-13 |
| `docs/compliance/meity-mapping.md` | _pending_ | 2026-05-13 |
| `docs/compliance/audit-format.md` | _pending_ | 2026-05-13 |

### Cross-repo references (informational — not checksummed here)

These live in product repos; their hashes are tracked in the product repo's own evidence-pack manifest:

- `pulkitpareek18/ZeroAuth: docs/api_contract.md`
- `pulkitpareek18/ZeroAuth: docs/error_codes.md`
- `pulkitpareek18/ZeroAuth: docs/threat_model.md` (will be deprecated in favor of `canonical.md` here)
- `pulkitpareek18/ZeroAuth: adr/*.md`

---

LAST_UPDATED: 2026-05-13
NEXT_REFRESH: every PR that touches a tracked source file
