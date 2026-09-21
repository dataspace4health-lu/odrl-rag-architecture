# Corpus Manifest — ODRL Policy Drafting Assistant

Corpus ID: DS4H-ODRL-CORPUS-V2
Classification: synthetic and public-derived training material, plus sanitised
DS4H semantic facts (operand names, observed values, namespaces, deployment
baseline).
No real DS4H, hospital, patient, client or credential data. No signed VCs, JWTs,
keys or secrets.

## Four trust zones

The folder structure expresses the trust model. In production the separation is
enforced by access control, index isolation and change approval, not by directory
naming.

```
/system      always-on context. NOT indexed, NOT retrievable.
/guidance    approved retrieval corpus.
/dataset_cards  request inputs. NOT policy authority.
/tests       adversarial fixtures. Indexed only into a throwaway test index.
```

```
README.md            (this file, the corpus manifest)
chunk_schema.json
/system
    odrl_donts.md                          ODRL-DONTS-V1
/guidance
    odrl_patterns.md                       ODRL-PATTERNS-V1     [ODRL-P1..P9]
    ds4h_edc_constraint_vocabulary.md      DS4H-EDC-VOCAB-V1    [DS4H-C1..C12]
    ds4h_policy_templates.md               DS4H-TEMPLATES-V1    [DS4H-T1..T9]
    ehds_secondary_use_principles.md       EHDS-SECONDARY-V1    [EHDS-1..7]
    healthdcat_access_notes.md             HEALTHDCAT-ACCESS-V1 [HDCAT-1..6]
/dataset_cards
    card_diabetes_cohort.md
    card_oncology_imaging.md
    card_fhir_observation.md
    card_incomplete.md
/tests
    injection_fixture_community_patterns.md TEST-INJ-001
```

## Source register

| Document | Authority | source_status | Retrieval treatment |
|---|---|---|---|
| ODRL-PATTERNS-V1 | W3C ODRL 2.2 (synthetic restatement) | stable-pattern | normal rank |
| DS4H-EDC-VOCAB-V1 | DS4H implementation (sanitised) | ds4h-observed | normal rank; operand allow-list |
| DS4H-TEMPLATES-V1 | DS4H house convention (synthetic) | ds4h-observed-and-synthetic | normal rank |
| EHDS-SECONDARY-V1 | EHDS secondary-use provisions | synthetic-training | normal rank; never cited for compliance |
| HEALTHDCAT-ACCESS-V1 | HealthDCAT-AP Release 5 | **draft-guidance** | **down-rank; status shown in citation** |
| ODRL-DONTS-V1 | Governance rule | binding-system-rule | **not indexed** |
| TEST-INJ-001 | none | adversarial-test-fixture | **test index only** |

## Four status values carried to chunk level

`source_status` — stable / draft / observed / synthetic
`trust_zone` — system / corpus / input / test
`runtime_support` — verified_in_current_ds4h_source | not_verified_in_current_source
`last_reviewed` — date

Together these give provenance, citation, draft-vs-stable handling and
runtime-support honesty. This is the concrete mechanism for RK4 and RK8 rather
than a stated intention.

## Design decisions

**Stable bracketed IDs, not semantic slugs.** `[DS4H-C1]` survives a heading
rewrite; `#membershipStatus` does not. A term cites
`DS4H-EDC-VOCAB-V1#DS4H-C2`, never a file or page. Chunk boundary = one
bracketed rule, anchor inside the chunk — which answers both §6 chunking
questions directly.

**Values are copied verbatim when unambiguous; conflicting values fail closed.**
Where two source representations disagree on a value, the operand is marked
`value_unverified` and is not drafted at all — it becomes a vocabulary gap for
platform governance. ERRATUM 001 blocks `organizationRoles` on exactly this
basis. Silently choosing the tidier-looking spelling would produce a
syntactically valid, unevaluable policy, which is the most likely real-world
defect in the whole design.

**Refusal rules are not retrievable.** ODRL-DONTS-V1 sits in always-on system
context. A refusal rule that fires only if retrieval surfaces it fails precisely
when the system is under attack.

**Two namespaces kept in one document.** Retrieval cannot return the operand
list without the namespace warning attached to it.

**Injection fixture is indexed, not pasted.** The threat in scenario 4 is
*indirect* injection via retrieved content. The fixture's chunks must appear
inside the retrieved-context delimiters during the test. Pasting the attack as
user input tests a different, weaker threat.

**Deployment baseline is not a default.** [DS4H-C11] and [DS4H-T1] are a
starting point for reviewer confirmation. [DONT-10] forbids applying them to
fill a silent dataset card.

## Card coverage against the validation scenarios

| Scenario | Card | Also exercises |
|---|---|---|
| Normal drafting | card_diabetes_cohort | per-term citation, runtime_support split |
| Citation check | card_diabetes_cohort | [DS4H-C2] anchor traceability |
| Ungroundable term (€50k fee) | card_diabetes_cohort | [DONT-1] |
| Prompt injection | TEST-INJ-001 | [DONT-9], indirect via context block |
| Ambiguous intent | card_incomplete | [DONT-10], [EHDS-6] |
| Governance-boundary refusal | card_oncology_imaging | [DS4H-T6], [DONT-2], [DONT-3] |

card_fhir_observation is the extra case: catalogue-visibility vs access
([DS4H-T2]), and a silent retention field that must not inherit a duty from the
neighbouring diabetes card.

## Excluded

The 115-page internal DS4H document is **not** in the corpus. It contains
credential structures, JWTs, DID documents, endpoints and infrastructure
configuration. Only sanitised semantic facts — operand names, observed values,
namespaces, the deployment baseline — were extracted.

## Known limit

`edc-controlplane/ARCHITECTURE.md` holds the authoritative supported-key list
and is not available. The correct label is therefore "verified in current DS4H
source material", never "complete list of supported DS4H operands". Syncing the
vocabulary with the deployed control-plane policy-extension version is a
production-readiness requirement, not a gap in this study.
