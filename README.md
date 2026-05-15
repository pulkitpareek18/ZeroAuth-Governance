# ZeroAuth Governance

The cross-repo source of truth for ZeroAuth security policy, compliance mappings, the canonical threat model, ADR index, release coordination, and evidence-pack sources.

**This repo does not contain product code.** Product code lives in [`zeroauth-dev/ZeroAuth`](https://github.com/zeroauth-dev/ZeroAuth) (and future sibling repos for the verifier, IoT firmware, and mobile SDK).

## What's in here

| Folder | What it holds |
|---|---|
| [`docs/shared/`](docs/shared/) | The five shared policy documents every ZeroAuth repo's `CLAUDE.md` links back to: security policy, coding standards, naming conventions, incident response, and the DPDP §8(7) breach-notification procedure. |
| [`docs/threat-model/`](docs/threat-model/) | `canonical.md` — the cross-repo threat model. Per-component extensions for the API, verifier, IoT firmware, mobile SDK, and dashboard. |
| [`docs/compliance/`](docs/compliance/) | DPDP, IRDAI, RBI, and MeitY clause-to-control mappings. Plus `audit-format.md` (the platform-wide audit-event schema). |
| [`adr-index/ALL.md`](adr-index/ALL.md) | Cross-repo index of every ADR, with hyperlinks to the canonical ADR in its product repo. |
| [`release-coordination/`](release-coordination/) | `matrix.md` — which versions of which product repo go together. Per-release changelogs. |
| [`evidence-pack-sources/`](evidence-pack-sources/) | `CHECKSUMS.md` — hashes of every source file that feeds the SOC 2 / DPDP / pilot-buyer evidence pack. `RELEASES.md` — which evidence-pack version went to which buyer when. |

## Read me first

Open [`CLAUDE.md`](CLAUDE.md) — that's the constitution for this repo. It explains the conventions, the two-reviewer rule on shared policy, and what this repo must *not* do.

## Reviews

Changes to anything in `docs/shared/` require **two reviewers**: Pulkit (technical correctness) + Amit (governance). For DPDP / IRDAI / RBI clause-touching files, add counsel review. Enforced via [`CODEOWNERS`](CODEOWNERS).

## Build / test

```bash
npm install
npm run lint            # markdownlint
npm run check-links     # verify all relative + cross-repo references resolve
```

CI runs both gates on every PR; merge blocks if either fails.

## License

This documentation is licensed under [Creative Commons Attribution 4.0 International (CC-BY-4.0)](LICENSE). Reuse it, fork it, audit it — credit ZeroAuth (zeroauth-dev/ZeroAuth).
