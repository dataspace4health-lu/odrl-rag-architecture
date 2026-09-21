# Corpus layout

The folder structure expresses the trust model. In production the separation is
enforced by access control, index isolation and change approval, not by
directory naming.

| Folder | Zone | Indexed |
|---|---|---|
| `guidance/` | Approved retrieval corpus | yes |
| `system/` | Refusal rules, always in context | **never** |
| `dataset_cards/` | Request inputs, not policy authority | no |
| `tests/` | Adversarial fixture for the injection test | test index only |

A refusal rule that fires only when retrieval happens to surface it fails
exactly when the system is under attack. That is why `system/` is never indexed.

`README.md` in this folder is the corpus manifest.
`erratum-001-operand-conflict.md` documents an unresolved source conflict that
blocks one operand from being drafted at all.
