---
document_id: HEALTHDCAT-ACCESS-V1
source_status: draft-guidance
authority_note: HealthDCAT-AP is a DRAFT application profile (Release 5, extends DCAT-AP v3). Namespace and several vocabularies are not ratified.
retrieval_note: DOWN-RANK relative to stable sources; draft status must travel into the citation.
last_reviewed: 2026-08-10
---

# HealthDCAT-AP Access-Condition Notes

Terms cited here must be surfaced to the reviewer as drawn from **draft**
guidance. Where a stable source supports the same term, prefer it.

## [HDCAT-1] Metadata and policy are different concerns
Metadata describes the dataset offering; the ODRL policy expresses candidate
access conditions attached to it. Do not fabricate metadata because a policy
would need it.

## [HDCAT-2] Access-condition information
Relevant fields may include stated access rights, licence, data-use conditions,
and applicable governance references. Missing metadata is reported, not inferred.

## [HDCAT-3] Access rights vs policy
Access-rights metadata and the ODRL policy must agree. If the card says public
and the intent describes tight restrictions, flag the inconsistency.

## [HDCAT-4] Licence
If licence information is absent, do not invent one. Return it as unknown where
relevant to the requested draft. A licence reference does not substitute for a
drafted constraint.

## [HDCAT-5] Governance references
A reference to a health data access body indicates governance may be required.
The assistant may surface the dependency; it may not impersonate the authority.

## [HDCAT-6] Draft-status handling
Draft guidance is never represented as a final legal requirement.
