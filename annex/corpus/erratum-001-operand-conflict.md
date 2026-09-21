# Corpus Erratum 001 — organizationRoles value conflict

**Status:** unresolved. Blocks drafting of the `organizationRoles` operand.

## The conflict
Two representations of the same DS4H policy, in the same source pack, disagree:

| Representation | Value |
|---|---|
| Machine-readable policy JSON (implementation source) | `healthdata-user` |
| Human-readable attribute table (deployment documentation) | `health-data-user` |

Both agree on `health-data-applicant`.

## Which is likely authoritative
Evidence favours the JSON:
- The same attribute table renders `supportedProtocols = 2025-1`, whereas the
  JSON has `dataspace-protocol-http:2025-1`. The table demonstrably truncates
  values — it is a lossy restatement, not a transcription.
- The JSON preserves artefacts faithfully: `permitIssued` carries `"true "`
  with a trailing space, which a cleaned-up table would silently drop.

Counter-consideration: PDF text extraction can corrupt strings at line breaks.
This cannot be excluded from the available material.

## Resolution required
Verify against `edc-controlplane/ARCHITECTURE.md` or a live policy definition in
the deployed control plane. Not available in the current source pack.

## Interim handling
`DS4H-EDC-VOCAB-V1#DS4H-C6` is marked `value_unverified`. The assistant does not
draft the `organizationRoles` operand; the requirement is routed to gaps[] as a
vocabulary gap for platform governance.

## Why this matters
A concrete, documented instance of RK4 (stale/uncertain guidance) and RK8
(syntactically valid, semantically unevaluable policy). The conflict is
invisible unless both representations are read. It is the strongest available
argument for the production-readiness requirement: **the RAG operand vocabulary
must be generated from, and version-pinned to, the deployed control-plane policy
extension — never hand-transcribed from documentation.**

## Related observation
`permitIssued` right operand appears as `"true "` (trailing space) in the JSON.
Flag; do not silently clean. If real, it is an operand-matching hazard.
