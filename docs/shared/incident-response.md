# ZeroAuth Incident Response Runbook

> **Last reviewed by:** Pulkit Pareek (technical), Amit Dua (governance) on 2026-05-13
> **Status:** v1 — runbook is operational but **PROVISIONAL** at steps 6 (customer notification), 7 (regulator notification), 8 (postmortem disclosure) until external DPO counsel is engaged. Engagement target: before first pilot SOW signing (~2026-07-01). See ADR-0005 (open) in `pulkitpareek18/ZeroAuth`.
>
> **Operating without counsel for an actual SEV-1 today is feasible but risky.** Specifically: no attorney-client privilege on the incident channel, no specialist who has run a DPBI submission before, no relationship to call at 2am. The founders accept this risk during the interim and treat counsel engagement as a hard blocker on first pilot SOW.

This is the on-call runbook for any operational incident affecting ZeroAuth. For incidents involving customer data breach specifically, **also** follow [`breach-notification.md`](breach-notification.md) — that procedure has regulatory deadlines this one does not.

## What counts as an incident

| Severity | Definition | Example |
|---|---|---|
| **SEV-1** | Production unavailable, OR active exploitation of a vulnerability, OR customer data exposure suspected/confirmed | `zeroauth.dev` is down; a tenant can read another tenant's audit events; a verifier soundness break is suspected |
| **SEV-2** | Production degraded, OR a non-exploited vulnerability needs patching within 24 hours, OR a partial customer-impacting bug | API latency >2s p99 for >15 min; signup flow broken; a dependency CVE rated high |
| **SEV-3** | Internal-only impact, OR a customer-facing issue affecting <5% of requests | Dev environment down; one customer's webhook delivery failing |

## Contact tree

| Role | Name | Channel | Backup |
|---|---|---|---|
| Incident commander (engineering) | Pulkit Pareek | (TODO: confirm Signal/phone for after-hours) | Amit Dua |
| Incident commander (business / customer comms) | Amit Dua | (TODO: confirm phone) | Pulkit Pareek |
| DPO function (interim) | Pulkit Pareek + Amit Dua jointly | same channels as incident commanders | — |
| External DPO counsel | **NOT YET ENGAGED** — see ADR-0005 (open). Engagement target ~2026-07-01. | — | — |
| IP counsel | **NOT YET ENGAGED** — see ADR-0005 (open). | — | — |
| Hosting / infra emergency | Pulkit Pareek | VPS root SSH (`104.207.143.14`) | — |

## Steps

### Step 1 — Detect

Detection sources:

- Cowork DW08 — CI failure escalator (when CI fails twice in a row)
- `https://status.zeroauth.dev` red status *(planned — status page not yet wired up)*
- Customer email to `security@zeroauth.dev` *(TODO: confirm mailbox routes)*
- Manual observation by Pulkit / Amit

### Step 2 — Triage (within 15 minutes of detection)

1. Confirm the incident is real (not a noisy alert). Hit `https://zeroauth.dev/api/health`. Hit `/v1/audit` with a test key.
2. Assign severity (table above).
3. Open an incident channel (TBD — Signal group / GitHub issue with `incident` label).
4. Name the incident commander. Default: Pulkit for SEV-1/SEV-2; Amit for non-technical SEV-1.

### Step 3 — Contain (SEV-1 only, within 30 minutes)

1. If exploitation suspected: rotate the affected API keys immediately. Force every tenant's `Default Live Key` to be reissued.
2. If verifier soundness suspected: take `/v1/verifications` offline (return `503` from the route) until rotation is complete. Per [`security-policy.md` §3.7](security-policy.md), this must happen within 48 hours of suspicion.
3. If data exposure suspected: **escalate to `breach-notification.md` immediately** — that runbook has the 72-hour DPDP clock.

### Step 4 — Investigate

1. Capture state: VPS `docker compose logs` since the suspected onset time. Save to `/zeroauth/incidents/<YYYY-MM-DD>/logs/` *(internal tree, outside this repo)*.
2. Capture database snapshot: `pg_dump` to the same incident directory. **Encrypt at rest** before storing.
3. Identify root cause hypothesis. Write it down in the incident channel.
4. Test the hypothesis. Disprove or confirm.

### Step 5 — Remediate

1. Apply the fix. PR must include:
   - The fix itself
   - A regression test that would have caught the incident
   - An update to [`/docs/threat-model/canonical.md`](../threat-model/canonical.md) referencing the attack scenario by `A-NN`
2. Deploy via the standard CI + deploy pipeline. **No hotfix-via-SSH unless SEV-1 + the CI pipeline itself is broken.**
3. Verify the fix worked. Run the regression test against production.

### Step 6 — Notify customers (SEV-1 only)

- If customer data was exposed: follow [`breach-notification.md`](breach-notification.md). Do not deviate.
- If customer-impacting downtime > 30 minutes: email all affected tenant contacts within 24 hours with: incident summary, impact, fix deployed at (timestamp), what we're doing to prevent recurrence. **Once external counsel is engaged:** CC counsel for approval before send. **Interim:** both founders co-sign the draft before send.
- Update `status.zeroauth.dev` *(planned)*.

### Step 7 — Regulator notification

- See [`breach-notification.md`](breach-notification.md) §§3–5 for the DPDP §8(7) procedure (72-hour clock from confirmation).
- IRDAI cybersecurity incident reporting (for BFSI tenants): within 6 hours of confirmation. *(TODO: confirm 6h is current — the IRDAI guidelines were updated in 2024.)*
- RBI Cyber Security Framework: within 2–6 hours depending on tenant classification. *(TODO: confirm.)*
- CERT-In: incidents matching the listed categories within 6 hours per the 2022 directives.

### Step 8 — Postmortem (within 7 days of remediation)

Write a postmortem. Template + skill: `02_claude_code/skills/(postmortem-drafter)` in the prompt suite. Required sections:

1. Timeline (UTC, minute-precision)
2. Impact (which tenants, how many records, what controls failed)
3. Root cause (technical + organizational)
4. What went well
5. What went poorly
6. Action items, each with an owner and a deadline
7. Threat model update reference (which `A-NN` was strengthened, or which new one was added)

Postmortems for SEV-1 go to all tenant contacts (sanitized). Postmortems for SEV-2/SEV-3 stay internal.

### Step 9 — Audit-log review

After every SEV-1, run an audit-log review for the 30 days preceding the incident, looking for prior-art evidence the same attack was attempted earlier. If found, the incident severity is upgraded retroactively for regulator-notification purposes.

## What we DO NOT do

- Restart the production stack to "see if that fixes it" before capturing logs
- Hotfix via SSH while the CI is up
- Email customers without legal counsel sign-off when the incident is SEV-1
- Discuss the incident on public channels before customer notification + regulator notification are complete

---

LAST_UPDATED: 2026-05-13
NEXT_REVIEW: 2026-08-13 (quarterly) or after any SEV-1
