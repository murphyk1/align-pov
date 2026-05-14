# PoV Running Notes — Acme Corp

Maintained by: [SE name]
Last updated: [DATE]

This file captures running context from follow-up calls, internal decisions,
and clarifications that have come up since the original RFP / requirements
walkthrough. The customer's RFP doc is preserved as-received in this folder;
updates and refinements live here.

When you re-run `/align-pov`, pass this file as the third (optional) argument
along with the template and the RFP, and the plugin will layer this context
on top of the customer's stated requirements.

---

## [DATE] — Stakeholder alignment call

- **Executive sponsor confirmed**: [Title — e.g., VP, Information Security].
  Will attend the executive alignment meeting and the readout.
- **Project owner / champion**: [Title — e.g., Director, Endpoint Security].
  Day-to-day point of contact.
- **Security Operations lead**: [Title — e.g., SOC Manager].
- **IT / Infrastructure lead**: [Title — e.g., Sr. Manager, Endpoint Engineering].
- **Identity / AD lead**: [Title — e.g., Identity & Access Engineering Lead].
- **Note**: [Customer CISO] is now joining the executive alignment meeting only.
  Heads-up for the AE.

## [DATE] — Timeline locked

Customer prefers a 30-day PoV. Tentative dates (verbally agreed; final
sign-off at kickoff):

- [DATE] — Commercial discussion & terms alignment
- [DATE] — PoV workshop & technical deep-dive
- [DATE] — Executive alignment meeting
- [DATE] — PoV kickoff & environment deployment begins
- [DATE] — Check-in #1
- [DATE] — Check-in #2
- [DATE] — Threat scenario demonstrations (multi-day block)
- [DATE] — Readout & success criteria review
- [DATE] — Commercial decision / next steps

## [DATE] — Scope confirmed

- Endpoint counts (full estate): ~[N] Windows, ~[N] macOS, ~[N] Linux,
  [N] Kubernetes clusters.
- PoV cohort (representative slice): [N] Windows, [N] macOS, [N] Linux,
  [N] cluster. Mid-PoV expansion possible if Phase 1 deployment is on-track.
- Legacy OS: [N] [LEGACY OS — e.g., Server 2008 R2] hosts running a legacy
  application — in scope but tested in an isolated network segment; agent
  install + basic prevention only (not full feature parity).

## [DATE] — Deployment model & retention

- **SaaS console only**. No on-prem / OVA deployment requirement —
  reclassify on-prem from "Confirm" to "Out of Scope".
- **Retention**: [N] days in-console, with extended retention via export to
  customer-owned storage ([e.g., S3 bucket, Snowflake, etc.]).

## [DATE] — XDR target platforms confirmed

For cross-tool actioning demos, validate:

- [Identity provider — e.g., disable user, force re-auth]
- [Secure web gateway — e.g., move user to restrictive group]
- [Email security gateway — e.g., disable email send for compromised user]
  (this is **net-new vs. the RFP** — please add to scope)

Out of scope for this PoV: [Other integration tools]. Deferred to phase 2
post-purchase.

## [DATE] — Mobile decision

Mobile is deferred to a follow-on PoV. Out of scope for this engagement.
Customer wants to revisit once their MDM rollout is further along.

## [DATE] — Operational SLAs

- Policy propagation: <[N] minutes across the full fleet.
- Critical / Severity-1 alert handling: [N]-hour SLA.
- Non-critical alert handling: [N]-hour SLA.
- Routine management tasks (policy edits, exclusions): same-day.
- New integrations / platform configuration changes: [N] hours.

## [DATE] — Browser standardization

Customer is standardized on [BROWSER 1] and [BROWSER 2] only. Don't validate
[BROWSER 3] / [BROWSER 4]; reclassify those as Out of Scope.

## [DATE] — New threat scenario request

Customer asked us to specifically test against recent ransomware variants
relevant to their industry sector. Add a scenario covering: [variant 1],
[variant 2] behaviors, validated against the PoV cohort.

## [DATE] — Priority change

[Capability — e.g., container security] was originally one of several
priority categories. Post-walkthrough, customer leadership has elevated
**[capability] to a top-[N] priority** — driven by [reason]. Reflect in PoV
scenario emphasis and success criteria priority.

---

## Open items still requiring confirmation

- [Open question 1 — context, owner, target date]
- [Open question 2 — context, owner, target date]
- [Open question 3 — context, owner, target date]
