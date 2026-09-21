---
document_id: ODRL-PATTERNS-V1
source_status: stable-pattern
authority_note: Synthetic training document. NOT the W3C ODRL specification.
scope: generic policy language — not DS4H implementation vocabulary
last_reviewed: 2026-08-10
---

# Candidate ODRL Drafting Patterns

Each section is a self-contained retrievable unit. Cite the bracketed ID.

---

## [ODRL-P1] Permission to use
A permission grants an action (typically `use`) to an assignee, optionally
narrowed by constraints. A permission with no constraint grants broadly and
must be flagged for reviewer confirmation.

## [ODRL-P2] Purpose limitation
Intent shape: "Only for approved secondary research."
Construct: constraint on a permission.
Purpose values must come from an agreed vocabulary. A free-text purpose string
is a comment, not a constraint. If the intent says only "for research", ask
which purpose rather than inventing a specific one.
Runtime support: not verified in current DS4H source pack.

## [ODRL-P3] Territorial / EU-only restriction
Intent shape: "EU/EEA participants only."
Two equivalent expressions: constrain the permission to the permitted
territory, or prohibit the excluded territory. DS4H uses both forms. Prefer the
form matching the house template so offerings stay comparable.
For machine-evaluable forms use only operands listed in DS4H-EDC-VOCAB-V1.

## [ODRL-P4] No re-identification
Intent shape: "Do not attempt to re-identify data subjects."
Construct: prohibition.
This binds *future conduct*, unlike an eligibility constraint checked from an
attribute presented at negotiation. Do not claim the connector verifies it.
Runtime support: not verified in current DS4H source pack.

## [ODRL-P5] Delete after use / study completion
Intent shape: "Delete when the approved study finishes."
Construct: duty.
The triggering event must be determinate. "After study completion" is usable;
"delete later" is not — ask.
Runtime support: not verified in current DS4H source pack.

## [ODRL-P6] Combining constraints
`and` = all conditions must hold. `or` = any is sufficient.
Never convert `and` to `or` to make a request easier to satisfy. Logical
composition materially changes policy meaning.

## [ODRL-P7] Obligation-style precondition
Distinct from [ODRL-P5]: a condition on an externally produced status that must
already hold, not an action the assignee performs afterwards.
See DS4H-TEMPLATES-V1#DS4H-T6.

## [ODRL-P8] Assignee-attribute constraint
Constrains eligibility using attributes the counterparty presents at
negotiation. This is the family a connector can evaluate directly.

## [ODRL-P9] Citation requirement
Every emitted permission, constraint, prohibition or duty cites the pattern
supporting it. No citation means the term is not emitted as grounded; it is
returned as `cannot_ground` with a reason.
