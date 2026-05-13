# Release Coordination Matrix

> **Last reviewed by:** Pulkit Pareek on 2026-05-13

This matrix records which versions of which ZeroAuth product repo are known-compatible. Before cutting any release in any repo, check this file. Cross-repo coordinated releases get a row in `changelogs/`.

## How to read

Each row is a "compatibility set" — a specific combination of repo versions that has been tested end-to-end and is supported as a unit.

Currently most repos don't exist yet, so the matrix is sparse. As new repos come online (verifier in Week 2, IoT in Week 3, SDK in Week 5), this file expands.

## Current matrix

| Set ID | Date | API (`ZeroAuth`) | Verifier | IoT firmware | Mobile SDK | Dashboard | Governance (`ZeroAuth-Governance`) | Notes |
|---|---|---|---|---|---|---|---|---|
| `pre-release-1` | 2026-05-12 | `0d1741d` (main) | inline in API | n/a | n/a | included in API | `pre-release-1` | First production deploy — central API + dashboard live on zeroauth.dev. Verifier inline. |
| `pre-release-2` | (post-B02) | TBD | TBD | n/a | n/a | included in API | TBD | Planned: verifier service split |

## Compatibility rules

- A repo MAY ship a patch release (`vX.Y.Z+1`) without bumping the matrix
- A repo MUST coordinate via this matrix when it bumps minor or major
- Breaking API contract changes (`/v1/*` shape) require a new matrix row AND a corresponding entry in `changelogs/`

## Open items

- Adopt semver versioning across repos (currently using commit SHAs)
- Automate matrix update from CI release events
- DW07 (release dispatcher) integrates with this file

---

LAST_UPDATED: 2026-05-13
