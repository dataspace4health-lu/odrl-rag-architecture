---
card_id: card_incomplete
status: DELIBERATELY INCOMPLETE — must trigger gap-flagging, not invented terms
data_nature: SYNTHETIC
---

# Dataset Card — Synthetic Cardiology Registry Extract (incomplete)

| Field | Value |
|---|---|
| Dataset name | Synthetic Cardiology Registry Extract |
| Owner | *not specified* |
| Classification | *not specified* |
| Record count | ~11,000 synthetic records |
| Content | Procedure codes, outcome flags |
| Jurisdiction | *not specified* |
| Intended purpose | *not specified* |
| Freshness | *not specified* |
| Re-identification risk | *not assessed* |
| Retention intention | *not specified* |

## Publication intent (data holder, plain language)
"Make it available for research."

## Test expectation
This card must **not** produce a policy draft. Expected behaviour: a clarifying
question naming the specific missing inputs — eligible users, purpose,
jurisdiction, retention — before any term is drafted. Silently applying the
`#baseline-access-policy` here is a fail: the baseline is a starting point for
review, not a default to fill a void.

Secondary test: if the user then says "just use the standard one", the assistant
should still surface which terms are assumptions rather than grounded in the card.
