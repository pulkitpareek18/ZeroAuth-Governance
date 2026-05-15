# Governance — CLAUDE.md

This file is the constitution for the ZeroAuth governance repo. Claude reads it at every session start. It overrides any contradicting inline comments or sub-folder READMEs.

## Read this first

- This file (you are here)

There is no other file to read first. This is the top of the chain.

## Project overview

The governance repo is the **single source of truth for everything that crosses repos**:

- The shared security policy that every other repo's CLAUDE.md links to
- The canonical threat model (each product repo has its own component-scoped threat model; the canonical is here)
- The canonical ADR index across all repos
- The compliance mapping (DPDP, IRDAI, RBI, MeitY) — clauses to control mappings
- The shared CHECKSUMS for the evidence pack
- The release coordination across repos (which repo's vX.Y.Z release goes with which other repo's release)

It does not contain product code. It is documentation, policy, and cross-cutting metadata only.

## Sibling repos that link here

| Repo | URL | What links here |
|---|---|---|
| `zeroauth-dev/ZeroAuth` | <https://github.com/zeroauth-dev/ZeroAuth> | Its `CLAUDE.md`, `docs/threat_model.md`, `adr/` (all link to the canonical versions here) |
| `zeroauth-dev/ZeroAuth-Verifier` | (planned, Week 2 — B02) | Its CLAUDE.md will link here |
| `zeroauth-dev/ZeroAuth-IoT` | (planned, Week 3 — B03) | — |
| `zeroauth-dev/ZeroAuth-Mobile-SDK` | (planned, Week 5 — B04) | — |

## What this repo contains

```text
/docs/
  /shared/
    security-policy.md         ← THE shared rules every repo follows
    coding-standards.md        ← style across repos
    naming-conventions.md      ← service names, env vars, log fields
    incident-response.md       ← runbook
    breach-notification.md     ← DPDP §8(7) procedure with counsel routing
  /threat-model/
    canonical.md               ← the cross-repo threat model
    api.md                     ← component-specific extension (zeroauth-api)
    verifier.md                ← component-specific extension
    iot.md                     ← component-specific extension
    sdk.md                     ← component-specific extension
    dashboard.md               ← component-specific extension
  /compliance/
    dpdp-mapping.md            ← canonical DPDP Act 2023 mapping
    irdai-mapping.md           ← canonical IRDAI Cybersecurity Guidelines mapping
    rbi-mapping.md             ← canonical RBI Cyber Security Framework mapping
    meity-mapping.md           ← canonical MeitY guidelines mapping
    audit-format.md            ← what an audit log entry is across the platform
/adr-index/
  ALL.md                       ← every ADR from every repo, with cross-references
/release-coordination/
  matrix.md                    ← which versions of which repos go together
  changelogs/                  ← per-release notes spanning all repos
/evidence-pack-sources/
  CHECKSUMS.md                 ← hashes of all evidence-pack source files
  RELEASES.md                  ← which evidence-pack version went to which buyer when
/CODEOWNERS                    ← every file has an owner
/.claude/                      ← Claude Code skills + agent definitions specific to this repo
```

## What this repo must NOT do

- Never contain product code
- Never contain customer data or buyer-specific information (those live in `/zeroauth/customers/<buyer>/` in the operational tree, outside any git repo)
- Never contain secrets
- Never duplicate content from product repos that should be in those product repos (link, don't copy)

## Conventions

- **Format**: markdown only
- **Cross-references**: relative paths to other files in this repo; for cross-repo references, name the repo and the path: `zeroauth-dev/ZeroAuth: /docs/api_contract.md`
- **Versioning**: every shared policy doc has a `LAST_UPDATED` field at the bottom; `RELEASES.md` records which version of each shared doc was in force at each evidence-pack publication
- **Reviews**: changes to anything in `/docs/shared/` require two reviewers (Pulkit + Amit). External DPO counsel review is required for DPDP-touching files **once counsel is engaged** — until then, the founders sign off jointly and the file carries a `PROVISIONAL` banner. Enforced via `CODEOWNERS`

## Standing instructions

1. **A change in this repo is consequential.** A change to `security-policy.md` propagates to every product repo via their CLAUDE.md. Always consider the blast radius before merging.

2. **Two-reviewer rule on shared policy.** Pulkit + Amit on technical policy. Add external DPO counsel review when DPDP, IRDAI, RBI, or IP clauses are affected — **once counsel is engaged**. Until then, the founders' joint sign-off is the gate and the file carries a `PROVISIONAL` banner. See ADR-0005 (open) for the engagement timeline.

3. **Threat model canonical wins.** When a product repo's threat model contradicts `/docs/threat-model/canonical.md`, the canonical is authoritative; the product repo updates.

4. **Use the `adr-writer` skill** for any new shared-policy ADR.

5. **Use the `compliance-mapper` subagent** when adding or modifying any compliance mapping. (Skill not yet installed — see ADR-0004 in `zeroauth-dev/ZeroAuth`.)

6. **Use the `threat-model-update` skill** when modifying the canonical threat model.

7. **Release coordination matters.** Before any product repo cuts a release, check `/release-coordination/matrix.md` for compatibility. Cross-repo coordinated releases get their own changelog entry.

## Critical rules

NEVER:

- Reference a specific buyer by name in `/docs/shared/`
- Reference an in-flight unreleased feature as "current" in `/docs/compliance/`
- Edit `/docs/shared/security-policy.md` without two-reviewer sign-off
- Edit `/docs/shared/breach-notification.md` without (a) external DPO counsel review once engaged, OR (b) both founders' joint sign-off during the interim period

ALWAYS:

- Every entry in `/adr-index/ALL.md` is hyperlinked to the canonical ADR in its product repo
- Every shared-policy doc starts with a "Last reviewed by" block (date + reviewers)
- Every threat-model entry has a STRIDE classification and a residual-risk statement

## Build / test commands

```bash
npm run lint            # markdownlint
npm run check-links     # verify cross-references resolve
npm run check-checksums # verify evidence-pack-sources/CHECKSUMS.md matches actual files
```

Pre-commit:

```bash
npm run lint && npm run check-links
```

## When you (Claude) get stuck

- A product repo's CLAUDE.md and this repo's shared policy disagree → the shared policy wins; update the product repo's CLAUDE.md
- A new compliance clause needs mapping but nothing in any repo implements a control → either the product repos need an ADR for the control, OR the mapping says "Partial — see roadmap"; **never invent a control**
- Cross-repo release matrix is unclear → don't cut a release; resolve the matrix first

---

LAST_UPDATED: 2026-05-13
OWNER: Amit Dua (governance, policy) + Pulkit Pareek (technical correctness)
