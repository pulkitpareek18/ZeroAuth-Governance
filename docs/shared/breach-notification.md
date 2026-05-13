# DPDP §8(7) Breach Notification Procedure

> **Last reviewed by:** Pulkit Pareek (technical), Amit Dua (governance) on 2026-05-13
> **Status:** v1 — **PROVISIONAL** until external DPO counsel is engaged (ADR-0005 open; target ~2026-07-01).
>
> **PROVISIONAL means:** every step that reads "DPO counsel" below is filled, in the interim, by Pulkit + Amit acting jointly as the DPO function. The 72-hour DPDP clock still runs and the founders still execute the procedure. What is missing is: attorney-client privilege on incident communications, specialized DPDP advice on edge cases ("is this a §8(7) notifiable event"), pre-review of the DPBI submission form, and a named external escalation if the founders disagree on a call. These gaps are accepted by the founders for the interim period and are a hard blocker on first pilot SOW signing.
>
> The contacts and time windows below are extrapolated from the Digital Personal Data Protection Act 2023 + the draft DPDP Rules 2025; final clauses pending notification.

This procedure is invoked when a confirmed or strongly-suspected breach has exposed personal data of ZeroAuth tenants' end-users. For non-breach operational incidents, follow [`incident-response.md`](incident-response.md) instead.

## §0. Scope: what counts as a notifiable breach

Per DPDP §2(1)(p), a "personal data breach" is any unauthorized processing of personal data, or accidental disclosure, acquisition, sharing, use, alteration, destruction, or loss of access to personal data.

For ZeroAuth specifically, the data ZeroAuth holds is:

1. **Commitments** (Poseidon hashes of biometric secrets). These are not personal data in plaintext — the underlying secrets were never received. Disclosure of a commitment does NOT trigger this procedure UNLESS combined with a verifier soundness break.
2. **Audit events** referencing tenant + user external IDs. Disclosure of these IS personal data exposure if the external_id maps to a natural person.
3. **Console signups** (tenant operator email + password hash). Disclosure of these IS personal data exposure.
4. **Tenant API keys at rest** (SHA-256 hashed). Disclosure of the hash is low-risk. Disclosure of plaintext keys is treated as exposure of every downstream tenant's commitment data.

## §1. Confirmation gate

A breach moves from "suspected" to "confirmed" when ANY of the following is true:

- Incident commander (Pulkit, with Amit) signs off on the determination
- A third party (security researcher, customer, regulator) provides evidence
- An audit-log review (`incident-response.md` step 9) surfaces evidence of prior exploitation

**The 72-hour DPDP clock starts at the confirmation timestamp.** Capture this in the incident channel and pin it.

## §2. DPO function activation (within 2 hours of confirmation)

**Once external DPO counsel is engaged:**

1. Call DPO counsel. (Contact + backup populated post-engagement; see ADR-0005.)
2. Brief them on: nature of breach, scope (which tenants, how many records), confirmation timestamp, current containment state.
3. Counsel must approve the customer notification draft and the regulator notification draft before either is sent. No exceptions.

**Interim — until external DPO counsel is engaged:**

1. Both founders (Pulkit + Amit) join the incident channel within 1 hour of confirmation. The interim DPO function is filled by them jointly.
2. They brief each other in writing in the incident channel: nature of breach, scope, confirmation timestamp, current containment state. This becomes the contemporaneous record (it is **not privileged** — operate accordingly; assume it could be discoverable).
3. **Both founders must co-sign the customer notification draft and the regulator notification draft before either is sent.** No exceptions.
4. If the founders disagree on a call (e.g., "is this notifiable"), the conservative interpretation wins: treat it as notifiable. Document the disagreement for the postmortem.
5. The interim record is preserved for 7 years per [`security-policy.md` §6.3](security-policy.md), to allow for retroactive counsel review once engaged.

## §3. Affected-individual notification (DPDP §8(6))

The DPDP Act requires the Data Fiduciary (ZeroAuth's tenants are the Data Fiduciaries; ZeroAuth is the Data Processor) to notify affected Data Principals.

**ZeroAuth's role:** Notify our tenants (the Data Fiduciaries). The tenants then notify their end-users.

### Procedure

1. **Within 6 hours of confirmation:** Email every affected tenant's primary contact with:
   - Confirmation timestamp
   - Nature of the breach
   - Scope (their tenant's affected record count)
   - Containment status
   - Recommended action for their notification to their Data Principals
   - Our point of contact for follow-up (Pulkit / Amit)
2. Re-send + escalate to phone if no acknowledgement in 12 hours.
3. Keep a written record of every tenant notification (sent timestamp, ack timestamp, channel).

## §4. Regulator notification (DPDP §8(7) + DPDP Rules 2025)

The Data Protection Board of India (DPBI) must be notified within **72 hours of confirmation** per DPDP §8(7) [TODO: confirm window once Rules 2025 are finalized — the draft is 72h but final could differ].

### Procedure

1. **Within 24 hours of confirmation:** Draft the notification. Use the DPBI form at (TODO: URL once published). Required fields:
   - Nature of breach
   - Approximate number of Data Principals affected
   - Approximate number of records affected
   - Likely consequences
   - Measures taken or proposed
   - Contact details (DPO counsel)
2. **Within 48 hours of confirmation:** Reviewer sign-off. Once counsel is engaged: counsel reviews + approves the draft. Interim: both founders co-sign with explicit "best-effort, no counsel review" notation in the audit folder.
3. **Within 72 hours of confirmation:** Submit the notification. Capture the submission timestamp + reference number. Save to the incident folder.
4. Follow-up reports if new information emerges: per DPBI's request, typically every 7 days.

## §5. Adjacent regulator notifications

These run in parallel with §4, with shorter windows. Each requires sign-off on the draft — counsel once engaged, both founders during interim.

| Regulator | Window | Trigger condition | Counsel (post-engagement) |
|---|---|---|---|
| CERT-In | 6 hours | Any incident in the 2022 directive's list of categories | TBD post-engagement |
| RBI Cyber Security Framework | 2–6 hours | At least one affected tenant is an RBI-regulated entity | TBD post-engagement |
| IRDAI | 6 hours | At least one affected tenant is an IRDAI-regulated entity (insurer / intermediary) | TBD post-engagement |
| MeitY | per advisory | Per applicable Section 70B(6) directives | TBD post-engagement |

## §6. Public disclosure

- No public disclosure (LinkedIn, blog, marketing site, conference talk) before §3 + §4 are complete.
- Public disclosure draft must be approved by both founders (and counsel, once engaged). Pulkit drafts the technical sections; Amit drafts the customer-facing sections.
- Public disclosure timeline: typically within 30 days of containment, sooner if media coverage forces it.

## §7. Internal handling

- Incident channel notes are retained for 7 years (matches audit-log retention from [`security-policy.md` §6.3](security-policy.md)).
- No discussion of the breach on shared messaging (Slack, WhatsApp) until §3 + §4 are complete. Use the dedicated incident Signal group.
- No screenshots, no off-record briefings.
- **Privilege caveat during interim period:** before external counsel is engaged, the incident channel is **not protected by attorney-client privilege**. Assume every message could be discoverable. Stick to factual reporting; do not speculate about liability or fault in writing.

## §8. Recovery

After §3 + §4 are complete:

1. Postmortem (per `incident-response.md` step 8).
2. Threat model update — every breach maps to either a new `A-NN` entry or a strengthening of an existing one.
3. Customer pilot review — affected tenants get a free 30-day extension of any pilot, and a make-good call within 14 days.

## §9. Drill cadence

This procedure is drilled twice a year — once with external counsel (post-engagement), once internal-only. First drill: **2026-08 (Week 13 of build cycle)** — to be scheduled at the W05 Friday review.

## Pinned: DPDP timestamps cheat-sheet

| Event | Window from confirmation |
|---|---|
| DPO function activation (interim: both founders join channel) | 1–2 hours |
| Tenant notification sent | 6 hours |
| CERT-In / RBI / IRDAI (if applicable) | 2–6 hours |
| Notification draft ready | 24 hours |
| Sign-off complete (interim: both founders co-sign) | 48 hours |
| DPBI submission | **72 hours** |
| Tenant acknowledgement re-send if no ack | 12 hours after first send |

---

LAST_UPDATED: 2026-05-13
NEXT_REVIEW: external counsel review BEFORE first drill (target: counsel engaged by 2026-07-01; review by 2026-07-31)
DRILL_SCHEDULE: 2026-08, 2027-02, ongoing semi-annual
