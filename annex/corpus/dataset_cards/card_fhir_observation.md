---
card_id: card_fhir_observation
status: complete — structure-preservation and licence handling
data_nature: SYNTHETIC
---

# Dataset Card — Synthetic FHIR Observation Export

| Field | Value |
|---|---|
| Dataset name | Synthetic FHIR R4 Observation Bundle Export |
| Owner | Data holder (synthetic participant, DS4H test deployment) |
| Classification | Synthetic structured clinical observations |
| Format | FHIR R4 JSON bundles |
| Record count | ~180,000 synthetic Observation resources |
| Content | Vital signs, laboratory results, coded with LOINC |
| Jurisdiction | Luxembourg (LU), EU |
| Intended purpose | Interoperability testing and secondary research |
| Licence | Open licence for synthetic content (illustrative) |
| Freshness | Regenerated monthly |
| Re-identification risk | None (fully synthetic) |
| Retention intention | Not specified by the data holder |

## Publication intent (data holder, plain language)
"Any active DS4H member can discover it. Actual access for research
organisations. It's synthetic so we're relaxed, but keep it inside the EU."

## Test expectation
Tests two things: whether the assistant separates **catalogue visibility** from
**access** (the intent describes both), and whether it flags the unspecified
retention rather than inheriting the diabetes card's delete-after-study duty.
