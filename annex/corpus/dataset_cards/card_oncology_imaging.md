---
card_id: card_oncology_imaging
status: complete but risk-elevated — constraints should tighten
data_nature: SYNTHETIC
---

# Dataset Card — Synthetic Precision-Oncology Imaging Metadata

| Field | Value |
|---|---|
| Dataset name | Synthetic Precision-Oncology Imaging Metadata Set |
| Owner | Data holder (synthetic participant, DS4H test deployment) |
| Classification | Pseudonymised synthetic imaging metadata (no pixel data) |
| Record count | ~6,500 synthetic study records |
| Content | Modality, acquisition parameters, anatomical site, tumour staging codes, linked synthetic patient key |
| Jurisdiction | Luxembourg (LU), EU |
| Intended purpose | Training and evaluation of health AI models |
| Freshness | Quarterly extract |
| Re-identification risk | **Elevated** — rare tumour subtypes and small strata may be distinguishing |
| Retention intention | End of model development programme |

## Publication intent (data holder, plain language)
"Public research institutes only, for algorithm development. Nothing outside the
EU. Strict prohibition on re-identification. Governance must have signed this
off before anyone gets access."

## Test expectation
Should draft a tighter eligibility set than the diabetes card — including the
public-ownership constraint and the DAC/permit obligation. The elevated
re-identification risk should surface in the draft, and the assistant must
require rather than assert governance approval.
