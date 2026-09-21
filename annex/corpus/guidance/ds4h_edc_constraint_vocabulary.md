---
document_id: DS4H-EDC-VOCAB-V1
source_status: ds4h-observed
authority_note: Sanitised extract from supplied DS4H implementation material. NOT the complete catalogue.
namespace_odrl: http://www.w3.org/ns/odrl/2/
namespace_ds4h: https://w3id.org/edc/v0.0.1/ns/
last_reviewed: 2026-08-10
---

# DS4H / EDC Controlled Constraint Vocabulary

DS4H policies use **two namespaces**: W3C ODRL for policy structure
(`action`, `constraint`, `leftOperand`, `operator`, `rightOperand`), and the EDC
namespace for DS4H left operands. A policy using ODRL structure with an operand
outside this list may be syntactically valid and still unevaluable by DS4H.

**Completeness warning.** Derived from observed policy examples. The
authoritative list lives in `edc-controlplane/ARCHITECTURE.md`, which is not in
this corpus. Absence here means *unverified*, never *unsupported*.

**Transcription rule.** Values are recorded verbatim. Observed DS4H values are
not internally consistent in formatting; do not normalise them.

---

## [DS4H-C1] membershipStatus
Operator: `eq` · Observed value: `active`
Meaning: participant membership status in DS4H.
Observed use: permission constraint, including the catalog access policy.
Runtime support: verified_in_current_ds4h_source

## [DS4H-C2] euEeaStatus
Operator: `eq` · Permission value: `eu-member-state` · Prohibition value: `third-country`
Meaning: whether the participant is in the EU/EEA area.
Runtime support: verified_in_current_ds4h_source

## [DS4H-C3] organizationType
Operator: `eq` · Observed value: `research-organization`
Meaning: organisation classification used within DS4H.
Runtime support: verified_in_current_ds4h_source
**Caution:** does NOT encode commercial vs non-commercial. Do not draft
"non-commercial only" from this operand.

## [DS4H-C4] jurisdictionCountry
Operator: `eq` · Observed value: `LU`
Meaning: country of legal jurisdiction.
Runtime support: verified_in_current_ds4h_source

## [DS4H-C5] organizationOwnershipType
Operator: `eq` · Permission value: `public` · Prohibition value: `private`
Meaning: ownership classification.
Runtime support: verified_in_current_ds4h_source
**Caution:** ownership classification is not a commercial-purpose restriction.

## [DS4H-C6] organizationRoles
Operator: `eq`
Observed values:
```
health-data-applicant    <- agreed across both source representations
<user role>              <- DISPUTED: the machine-readable policy JSON and the
                            human-readable attribute table disagree on the exact
                            spelling of the second role value
```
Meaning: roles assigned to the participant. Multiple role values are combined.
Runtime support: **value_unverified**

**Do not draft this operand.** The conflict is unresolved (see ERRATUM 001) and
cannot be settled from the current source pack. A requirement that needs
`organizationRoles` is routed to `gaps[]` as a vocabulary gap for platform
governance, pending verification against the deployed Control Plane policy
extension. Choosing either spelling would produce a syntactically valid,
silently unevaluable policy — the failure this rule exists to prevent.

## [DS4H-C7] dataProtectionRole
Operator: `eq` · Observed value: `controller`
Runtime support: verified_in_current_ds4h_source

## [DS4H-C8] supportedProtocols
Operator: `eq` · Observed value: `dataspace-protocol-http:2025-1`
Technical interoperability constraint, not access governance. Rarely
appropriate to draft from a publication intent.
Runtime support: verified_in_current_ds4h_source

## [DS4H-C9] dacApproval
Operator: `eq` · Observed value: `true`
Observed pattern: `dacApproval = true OR permitIssued = true` (obligation policy)
Runtime support: verified_in_current_ds4h_source
**Governance rule:** this value is PRODUCED by the Data Access Committee
process. A policy may require it. Nothing in this system may set, assert or
simulate it.

## [DS4H-C10] permitIssued
Operator: `eq` · Observed value: `true`
Observed pattern: as [DS4H-C9].
Runtime support: verified_in_current_ds4h_source
Produced externally by the permit process.

## [DS4H-C11] Deployment baseline
The DS4H deployment test checklist identifies this minimum constraint set:
```
membershipStatus    = active
organizationType    = research-organization
jurisdictionCountry = LU
```
Treat as the **current DS4H deployment/test baseline**, not a universal ODRL
requirement, and not a default to apply when inputs are missing.

## [DS4H-C12] Operands not present
No operand for research purpose, re-identification, or retention/deletion
appears in the observed examples. Two distinct explanations, not to be
conflated:
1. **Vocabulary gap** — the operand may exist in the full catalogue this corpus
   lacks. Resolvable by syncing with the control plane.
2. **Structural limit** — constraints evaluate attributes presented at
   negotiation; no presented attribute attests to *future conduct*. Not
   resolvable by a better vocabulary file.
Label such terms `runtime_support: not_verified_in_current_source`. Do not
assert they are unenforceable.
