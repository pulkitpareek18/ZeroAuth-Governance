# DPDP §8(7) Breach Notification Procedure

> **Last reviewed by:** Pulkit Pareek (technical), Amit Dua (governance) on 2026-05-13
> **Status:** v1 — DRAFT pending counsel sign-off. **Do NOT execute this procedure without DPO counsel on the call.** The contacts and time windows below are extrapolated from the Digital Personal Data Protection Act 2023 + the draft DPDP Rules 2025; final clauses pending notification.

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

## §2. Counsel call (within 2 hours of confirmation)

1. **Call DPO counsel.** Contact: (TODO: name + phone — DPO counsel TBD). If unreachable in 30 min, escalate to (TODO: backup counsel).
2. Brief them on: nature of breach, scope (which tenants, how many records), confirmation timestamp, current containment state.
3. **Counsel must approve the customer notification draft and the regulator notification draft before either is sent.** No exceptions.

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
2. **Within 48 hours of confirmation:** Counsel reviews + approves the draft.
3. **Within 72 hours of confirmation:** Submit the notification. Capture the submission timestamp + reference number. Save to the incident folder.
4. Follow-up reports if new information emerges: per DPBI's request, typically every 7 days.

## §5. Adjacent regulator notifications

These run in parallel with §4, with shorter windows. Each requires its own counsel sign-off on the draft.

| Regulator | Window | Trigger condition | Counsel |
|---|---|---|---|
| CERT-In | 6 hours | Any incident in the 2022 directive's list of categories | (TODO) |
| RBI Cyber Security Framework | 2–6 hours | At least one affected tenant is an RBI-regulated entity | (TODO) |
| IRDAI | 6 hours | At least one affected tenant is an IRDAI-regulated entity (insurer / intermediary) | (TODO) |
| MeitY | per advisory | Per applicable Section 70B(6) directives | (TODO) |

## §6. Public disclosure

- No public disclosure (LinkedIn, blog, marketing site, conference talk) before §3 + §4 are complete.
- Public disclosure draft must be approved by counsel + Amit. Pulkit drafts the technical sections; Amit drafts the customer-facing sections.
- Public disclosure timeline: typically within 30 days of containment, sooner if media coverage forces it.

## §7. Internal handling

- Incident channel notes are retained for 7 years (matches audit-log retention from [`security-policy.md` §6.3](security-policy.md)).
- No discussion of the breach on shared messaging (Slack, WhatsApp) until counsel approves. Use the dedicated incident Signal group.
- No screenshots, no off-record briefings.

## §8. Recovery

After §3 + §4 are complete:

1. Postmortem (per `incident-response.md` step 8).
2. Threat model update — every breach maps to either a new `A-NN` entry or a strengthening of an existing one.
3. Customer pilot review — affected tenants get a free 30-day extension of any pilot, and a make-good call within 14 days.

## §9. Drill cadence

This procedure is drilled twice a year — once with counsel, once internal-only. First drill: **2026-08 (Week 13 of build cycle).** [TODO: schedule.]

## Pinned: DPDP timestamps cheat-sheet

| Event | Window from confirmation |
|---|---|
| Counsel call | 2 hours |
| Tenant notification | 6 hours |
| CERT-In / RBI / IRDAI (if applicable) | 2–6 hours |
| Notification draft | 24 hours |
| Counsel sign-off | 48 hours |
| DPBI submission | **72 hours** |
| Tenant acknowledgement re-send | 12 hours after first send |

---

LAST_UPDATED: 2026-05-13
NEXT_REVIEW: counsel review BEFORE first drill (target: 2026-06-30)
DRILL_SCHEDULE: 2026-08, 2027-02, ongoing semi-annual
