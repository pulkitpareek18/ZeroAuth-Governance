# Evidence Pack Releases

> **Last reviewed by:** Pulkit Pareek on 2026-05-13

Records every evidence pack assembly: which buyer received which version when, and what was in it.

When a buyer asks "what was in the evidence pack you gave us in May 2026," we look up the row here, then verify the checksums against `CHECKSUMS.md` at that commit.

## How to update

When the operational suite's W08 evidence-pack assembler runs, it appends a row here with:

- Version (semver-ish: `vYYYY-MM-DD.N`)
- Buyer
- Date issued
- Manifest (which files + their SHA-256 at that commit)
- Channel (email + sharing-link / USB / printed)

## Log

| Version | Buyer | Date | Manifest | Channel | Notes |
|---|---|---|---|---|---|
| _none yet_ | — | — | — | — | First buyer pilot pending — first evidence pack will issue at SOW signing |

## Manifest template

When the first release goes out, populate per this template:

```text
Manifest for vYYYY-MM-DD.N:
- docs/shared/security-policy.md @ <sha-256>
- docs/shared/coding-standards.md @ <sha-256>
- docs/shared/incident-response.md @ <sha-256>
- docs/shared/breach-notification.md @ <sha-256>
- docs/threat-model/canonical.md @ <sha-256>
- docs/compliance/dpdp-mapping.md @ <sha-256>
- (any others per buyer's request — IRDAI for an insurer, RBI for a bank)
- pulkitpareek18/ZeroAuth @ <commit SHA> (API contract + ADRs + threat model)
```

---

LAST_UPDATED: 2026-05-13
