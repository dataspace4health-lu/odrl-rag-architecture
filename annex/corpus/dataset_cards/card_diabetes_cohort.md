---
card_id: card_diabetes_cohort
status: complete — expect a well-grounded draft
data_nature: SYNTHETIC
---

# Dataset Card — Synthetic Retrospective Diabetes Cohort

| Field | Value |
|---|---|
| Dataset name | Synthetic Retrospective Type 2 Diabetes Cohort 2018–2024 |
| Owner | Data holder (synthetic participant, DS4H test deployment) |
| Classification | Anonymised synthetic health data |
| Record count | ~42,000 synthetic patient records |
| Content | Demographics, HbA1c series, prescribed medication class, comorbidity flags |
| Jurisdiction | Luxembourg (LU), EU |
| Intended purpose | Secondary scientific research |
| Freshness | Static extract, no refresh |
| Re-identification risk | Low (synthetic, no direct identifiers) |
| Retention intention | Duration of the approved study |

## Publication intent (data holder, plain language)
"Make this available to approved research organisations only, for secondary
research use. EU/EEA participants only. No attempts at re-identification.
Delete the data once the approved study finishes."

## Test expectation
A complete, cited draft. Eligibility constraints should ground to the DS4H
vocabulary; the purpose, re-identification and retention terms should carry
`runtime_support: not_verified_in_current_source`.
