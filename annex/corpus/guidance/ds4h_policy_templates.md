---
document_id: DS4H-TEMPLATES-V1
source_status: ds4h-observed-and-synthetic
authority_note: House conventions of the current deployment, not ODRL requirements.
last_reviewed: 2026-08-10
---

# Synthetic DS4H Policy Templates

Candidate drafting patterns. Not production policy objects; no guarantee of
schema-valid ODRL.

---

## [DS4H-T1] Baseline research-organisation permission
Intent: "Active research organisations in Luxembourg."
```
membershipStatus    = active
AND organizationType    = research-organization
AND jurisdictionCountry = LU
```
If a drafted access policy omits any of these three, flag for reviewer
confirmation. Starting point, not a ceiling — and not a default for a silent card.

## [DS4H-T2] Catalogue-visibility policy
Observed as a separate, minimal access policy:
```
membershipStatus = active
```
**Catalogue visibility and data access are different policies.** A dataset may
be discoverable to all active members while access stays tightly constrained.
If the intent mixes the two, ask which is meant.

## [DS4H-T3] EU/EEA restriction
Positive: `euEeaStatus = eu-member-state`
Exclusion: prohibition on `euEeaStatus = third-country`
Do not emit both automatically. Pair with `jurisdictionCountry` only when a
specific member state is genuinely required — "EU-only" does not imply `LU`.

## [DS4H-T4] Public-sector restriction
Positive: `organizationOwnershipType = public`
Exclusion: prohibition on `organizationOwnershipType = private`
Draft only when the intent names public-sector or publicly owned bodies. Not
from "non-commercial", which this operand does not express.

## [DS4H-T5] Organisation-role restriction — BLOCKED
Intent shape: "only participants holding role X".

**This template is not available for drafting.** It depends on
`organizationRoles`, whose values are unresolved between two source
representations (DS4H-EDC-VOCAB-V1#DS4H-C6, ERRATUM 001).

Correct handling: record the requirement in `gaps[]` with a reason and route it
to platform governance. Do not substitute a role value, and do not assume any
role means "approved researcher". The template is restored once the operand is
verified against the deployed Control Plane.

## [DS4H-T6] DAC approval / permit precondition
```
obligation: dacApproval = true OR permitIssued = true
```
Correct way to express "governance must have cleared this" as a policy condition.
**Boundary:** drafting the condition is in scope. Determining, asserting or
simulating the value is not.
Allowed: "Candidate policy should require dacApproval=true OR permitIssued=true."
Forbidden: "DAC approval granted." / "Permit issued." / "The requester qualifies."

## [DS4H-T7] Secondary-use purpose
Construct: purpose constraint. Sources: EHDS-SECONDARY-V1#EHDS-1, ODRL-PATTERNS-V1#ODRL-P2
Runtime support: not_verified_in_current_source

## [DS4H-T8] No re-identification
Construct: prohibition. Source: ODRL-PATTERNS-V1#ODRL-P4
Runtime support: not_verified_in_current_source

## [DS4H-T9] Delete after study
Construct: duty. Source: ODRL-PATTERNS-V1#ODRL-P5
Runtime support: not_verified_in_current_source
