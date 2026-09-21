# RAG Concept Walkthrough

Conceptual and pseudo-workflow level. No working retriever, application or UI is
claimed or implied.

Two decisions in this document — model choice and temperature — were settled by
Prompt Sandbox evidence rather than argued from a desk. Both are recorded below
with the observation that produced them.

---

## 1. Why RAG, and where it is the wrong tool

The guide's first learning outcome asks when RAG suits a structured-drafting
task and when a deterministic validator or a human expert is better. The honest
answer is that this system needs all three, in sequence.

| Approach | Verdict | Reasoning |
|---|---|---|
| Plain LLM, no retrieval | Rejected | The failure mode is fluent invented legal text. Nothing in the model's weights distinguishes a real DS4H operand from a plausible one. |
| Fine-tuning | Rejected as the primary solution | Fine-tuning specialises behaviour, and can be combined with retrieval (e.g. RAFT). What it does not provide on its own is current source retrieval and verifiable per-term provenance for the evidence actually used at inference. The corpus is small and changes independently of the model, so reindexing beats retraining when guidance or DS4H vocabulary shifts. Could complement RAG later if evaluation exposes a specific weakness. |
| **RAG** | **Chosen** | The knowledge is small, curated, changes on its own schedule, and every output must be traceable to a source. That is the RAG-shaped problem. |
| Deterministic validator | Complementary, out of scope | Cannot draft from natural-language intent, but *should* check the draft's structure. Correctly a production-readiness recommendation, not a runtime component here. |
| Human expert | Mandatory, not replaced | Approval, legal judgement and governance decisions are outside the system by design. |

The architecture is therefore: **RAG drafts → human reviews → governance
approves → EDC enforces.** Each stage does what it is actually good at.

---

## 2. Conceptual flow

```
  Dataset card              Publication intent
       |                            |
       +-------------+--------------+
                     |
                     v
          [1] Intent decomposition
              splits into discrete drafting intents
                     |
                     v
          [2] Per-intent retrieval  ------> approved corpus
              (one query per intent)        (docs/, status-aware ranking)
                     |
                     v
          [3] Controlled prompt assembly
              system rules OUTSIDE  |  retrieved chunks INSIDE delimiters
                     |
                     v
          [4] Grounded generation (low temperature, structured output)
                     |
                     v
     +---------------+----------------+
     |               |                |
 candidate_policy  gaps[]        refusals[]
 (cited terms)   (ungroundable)  (forbidden)
     |               |                |
     +---------------+----------------+
                     |
                     v
          [5] Output validation + logging
              citation present? operand in vocabulary? shape valid?
                     |
                     v
           HUMAN DATA HOLDER REVIEW
                     |
                     v
        governance approval -> EDC policy configuration
```

The boundary is after review, not after generation. Nothing this system produces
reaches a connector without a human decision in between.

---

## 3. The seven stages

Answering the architecture questions in the guide's §6 directly.

### Stage 1 — Document selection
*What is authoritative? What is excluded? How is draft vs stable flagged?*

Six documents, four trust zones. The folder structure **is** the trust model:

```
/system         always-on context, never indexed   (ODRL-DONTS-V1)
/guidance       the retrieval corpus                (5 documents)
/dataset_cards  request inputs, not authority       (4 cards)
/tests          adversarial fixtures, test index only (TEST-INJ-001)
```

Authority is declared per document in front matter, not inferred:

| Document | source_status | Treatment |
|---|---|---|
| ODRL-PATTERNS-V1 | stable-pattern | normal rank |
| DS4H-EDC-VOCAB-V1 | ds4h-observed | normal rank; operand allow-list |
| DS4H-TEMPLATES-V1 | ds4h-observed-and-synthetic | normal rank |
| EHDS-SECONDARY-V1 | synthetic-training | normal rank; never cited for compliance |
| HEALTHDCAT-ACCESS-V1 | **draft-guidance** | **down-ranked; status shown in citation** |
| ODRL-DONTS-V1 | binding-system-rule | **not indexed** |

Excluded: the 115-page internal DS4H document (credential structures, JWTs, DID
documents, cluster configuration). Only sanitised semantic facts were extracted —
operand names, observed values, namespaces, the deployment baseline.

**Why the refusal rules are not indexed.** A refusal rule that only fires when
retrieval happens to surface it fails precisely when the system is under attack.
ODRL-DONTS-V1 is always-on system context. This is a deliberate choice, not a
gap in the retrieval design.

### Stage 2 — Ingestion
*How are Markdown, tables and code blocks handled?*

Documents are authored in Markdown with YAML front matter, so ingestion is
parsing rather than extraction — no PDF or OCR stage, and no layout to lose.

- **Front matter** becomes chunk metadata, not chunk text.
- **Code blocks** (ODRL fragments, operand values) are preserved byte-for-byte.
  This matters more than it sounds: see ERRATUM 001, where two representations
  of one policy in the source pack disagree on an operand value. Values are
  never normalised, reflowed or spell-corrected.
- **Tables** are flattened to key–value lines, since a table row split across
  chunks loses its header and becomes meaningless.

**Production note.** Chunks must be *generated* from the source documents, never
hand-transcribed. Two citation-ID drift incidents occurred during this project's
own design reviews when chunks were retyped by hand — a small but real
demonstration of why the ingestion step has to be mechanical.

### Stage 3 — Chunking
*What chunk size preserves a whole pattern? How is a pattern kept with its
citation anchor?*

**One rule per chunk.** The boundary is semantic, not a token count. Each
bracketed rule (`[DS4H-C2]`, `[ODRL-P4]`, `[EHDS-5]`) is one self-contained
chunk: intent shape, construct, skeleton, guidance, runtime-support label.

Measured across the corpus, rules run from roughly 14 to 113 tokens with
a median near 45 (43 indexed chunks; word counts scaled by 1.4). The spread is
the point: a fixed-size splitter would have cut the long rules mid-pattern and
merged several short ones, producing fragments that retrieve well and ground
badly — an operand without its namespace warning, or a pattern without its
"not verified" label.

**The anchor travels inside the chunk.** The citation ID is part of the chunk
text, so a term cites `DS4H-EDC-VOCAB-V1#DS4H-C2` — a rule, never a file or a
page. Two consequences: citations survive document reordering, and a fabricated
citation is detectable because the ID either appeared in the retrieved context
or it did not.

Chunk record:

```json
{
  "document_id": "DS4H-EDC-VOCAB-V1",
  "section_id": "DS4H-C2",
  "citation": "DS4H-EDC-VOCAB-V1#DS4H-C2",
  "source_status": "ds4h-observed",
  "trust_zone": "corpus",
  "runtime_support": "verified_in_current_ds4h_source",
  "last_reviewed": "2026-08-10",
  "text": "euEeaStatus. Operator eq. Permission value eu-member-state..."
}
```

### Stage 4 — Indexing and retrieval
*Keyword or embedding? How is stale or draft guidance down-ranked?*

**Hybrid, and the reason is specific to this corpus.**

- Operand names are exact lexical tokens. `organizationOwnershipType` must match
  exactly; a near-neighbour is a different constraint with different semantics.
  Lexical search (BM25) is right for these.
- Intents arrive as natural language. "Keep it inside the EU", "EEA only", "no
  third countries" must all reach `[DS4H-C2]`. Embeddings handle that.

Either alone fails: pure embedding blurs operand names that are lexically close
and semantically distinct; pure keyword misses paraphrased intent.

**Retrieval runs per intent, not per request.** Intent decomposition splits the
publication intent into discrete drafting intents first, then issues one query
each. A single query over the whole paragraph lets a dominant phrase crowd the
top-k and silently starve a minor intent of evidence — which then surfaces as a
missing term rather than a retrieval failure, and is very hard to diagnose from
the output.

**Status-aware ranking.** `source_status` is a ranking signal. `draft-guidance`
chunks (HealthDCAT-AP) are down-ranked and, when retrieved, carry their status
into the citation shown to the reviewer. Stale-guidance handling is a mechanism
here, not an intention.

The indexed corpus is 43 chunks, so retrieval cost is not the binding
constraint. The real constraint is keeping the assembled evidence legible:
enough context to ground every intent, few enough chunks that the trust boundary
in stage 5 remains a clear separation rather than a wall of text.

### Stage 5 — Prompt assembly
*How are instructions separated from retrieved content to resist injection?*

Three separations, layered:

1. **Role separation — target design, not validated.** System rules belong in
   the system message; retrieved evidence belongs in the user/context layer.
   The Prompt Sandbox exposed a single prompt field, so this layer could not be
   exercised experimentally.
2. **Delimiter separation.** Retrieved chunks sit inside a named, closed
   `<retrieved_context>` block that the system prompt has already declared
   untrusted, alongside `<dataset_card>` and `<publication_intent>`.
3. **Declared trust semantics.** The system prompt states that content inside
   the block is evidence and never instruction, and that anything
   instruction-shaped is ignored and recorded in `injection_flags`.

**Layers 2 and 3 are tested; layer 1 is not.** Run 5 injects a malicious
instruction inside a retrieved chunk, and Run 5b repeats it with the trust-model
block removed. Without the ablation a pass would say nothing about which layer
produced the result — and with it, the honest finding is that no marginal
benefit from the trust block was measurable under this attack. The ablation
removed one control of several and cannot isolate base-model behaviour.

### Stage 6 — Answer generation
*How are citations forced? How is invention of legal terms prevented?*

Four mechanisms, deliberately overlapping:

1. **Structured output contract.** A term without a `citation` field is
   malformed. Grounding is a schema property, not a request.
2. **Operand allow-list.** Left operands come only from DS4H-EDC-VOCAB-V1,
   emitted verbatim. An unknown or disputed operand is a gap.
3. **Three-way outcome.** `candidate_policy` for grounded terms, `gaps[]` for
   ungroundable ones, `refusals[]` for forbidden ones. The model always has a
   legal move that is not invention.
4. **Low temperature.** Determinism matters more than fluency: the same intent
   must yield the same operands and the same citations. **Setting: temperature 0.1,
   top-p 1.** The contrast run at 0.8 produced the same substantive failures —
   the same collapsed constraints, the same incorrect citation, the same
   unrequested operand — with the approved-user requirement degrading from an
   incorrect refusal to silent omission. Low temperature is therefore retained
   for determinism, but temperature is not treated as a grounding or safety
   control.

**Grounded ≠ machine-enforceable.** Every term carries `runtime_support`.
Eligibility constraints map to verified DS4H operands. Purpose limitation,
re-identification prohibition and deletion duties are grounded in ODRL/EHDS
patterns but have no verified DS4H operand — those get null operand fields and a
`not_verified_in_current_source` label. Conflating the two would be the most
consequential silent error the system could make.

**Model choice: Claude 4.6 Sonnet as the preferred candidate for the next
evaluation stage** — on grounding and structured-policy fidelity in the
observable portions of its outputs. In the observable portions of its output it
maintained one citation per term, correct operand-to-citation alignment, verbatim
enum literals and the compound OR obligation. This is not yet a production model
selection: 7 of its 10 responses hit the platform output cap, leaving the
injection and governance-boundary tails unobservable, and the comparison covers
one task on one corpus. The evaluation must be repeated under the intended output
budget and true system/user role separation before a model is selected.

### Stage 7 — Evaluation and logging
*What is logged? What must not be logged? How is drift monitored?*

**Logged**

| Item | Why |
|---|---|
| Intent decomposition output | Distinguishes retrieval failure from generation failure |
| Retrieved chunk IDs + scores | Reconstructs why a term appeared |
| Prompt version hash | Ties behaviour to a specific design |
| Model + parameters | Reproducibility |
| Full structured output | The audit record |
| `injection_flags` events | Security signal; alert on rate change |
| Gap and refusal counts by type | Governance signal |
| Reviewer accept / edit / reject | The only real ground truth available |

**Not logged**

| Item | Why |
|---|---|
| Raw dataset-card content beyond an ID | Cards may carry sensitive metadata in production |
| Injection payload text in the main log | Quarantine separately; do not replay poisoned strings into an operational store |
| Any credential, VC, JWT or key | Never in scope, never in a log |
| Reviewer identity in the **analytics** store | See the two-store split below |

**Two stores, different purposes.** Production governance must record who
approved a candidate policy; monitoring must not become staff performance
profiling. Separate them:

```
AUDIT STORE (governance record, access-controlled, retained)
    candidate policy as approved
    corpus version + chunk IDs used
    model + prompt version
    approval decision, authorised reviewer identity, timestamp

ANALYTICS STORE (operational monitoring, aggregate only)
    accept / edit / reject rates
    gap and refusal counts by type
    retrieval and grounding metrics
    injection_flags rate
    no reviewer-level performance profiling
```

**Drift monitoring.** Three signals: retrieval drift (chunks that stop being
retrieved after a corpus update), grounding drift (rising rate of terms with
citations absent from the retrieved set), and reviewer-edit rate. The last is
the leading indicator — when stewards start editing the same term repeatedly,
the corpus is wrong before the metrics say so.

**Evaluation approach.** Retrieval and generation are scored separately, since
they fail differently: a missing term is usually retrieval, a wrong citation is
usually generation. Formal metric tooling is a production recommendation, not a
requirement of this study; the six validation scenarios plus the ablation are the
evidence here.

---

## 4. Pseudo-workflow

```
draft_policy(dataset_card, publication_intent):

    intents = decompose(publication_intent)          # per-intent, not per-request

    context = []
    for intent in intents:
        hits = hybrid_search(intent, k=4)            # BM25 + embedding
        hits = downrank(hits, where source_status == "draft-guidance")
        context += hits                               # chunk text carries its own anchor

    prompt = system_rules(ODRL_DONTS_V1)             # always-on, never retrieved
           + "<retrieved_context>"  + dedupe(context) + "</retrieved_context>"
           + "<dataset_card>"       + dataset_card    + "</dataset_card>"
           + "<publication_intent>" + publication_intent + "</publication_intent>"

    draft = generate(prompt, temperature=LOW, structured=True)

    for term in flatten(draft.candidate_policy):

        assert term.citation in {c.citation for c in context}
            # a term may cite only an anchor that was actually retrieved

        for condition in term.conditions:
            if condition.left_operand is not null:
                assert condition.left_operand in DS4H_VOCABULARY      # no invented operand
                assert condition.right_operand matches vocabulary verbatim  # no normalisation

        if term.logical_operator is not null:
            assert term.logical_operator in {"AND", "OR"}
            assert len(term.conditions) > 1                            # OR of one is malformed

    log(intents, context_ids, prompt_hash, draft, draft.injection_flags)
    return draft                                     # review_required is always true
```

These checks are the concept-level equivalent of a future deterministic
output-validation layer. They address fabricated citations (RK1), invented DS4H
operands (RK8), silently normalised values (ERRATUM 001), and malformed compound
conditions. Structural ODRL schema validation remains a separate
production-readiness recommendation, not something claimed here.

---

## 5. What this walkthrough deliberately does not include

| Excluded | Rationale |
|---|---|
| Working retriever or vector store | The guide accepts a pseudo-retrieval walkthrough; an embedding experiment is explicitly optional |
| ODRL schema validator in the runtime path | Structural validity is a production-readiness recommendation, not a requirement of this study |
| Agentic orchestration, MCP, multi-agent | Considered and rejected: added surface area, no material benefit for this scoped use case, and human review already gates the output |
| Live DS4H / EDC / IdentityHub integration | Out of scope for a design-and-validation study; unnecessary to demonstrate the pattern |
| Fine-tuning | Not required. Does not solve current retrieval or per-term provenance; may complement RAG later if evaluation identifies a model or retriever weakness |

---

## 6. Resolved decisions and remaining production follow-ups

| Item | Outcome |
|---|---|
| Model and temperature | **Resolved** — temperature 0.1; Claude 4.6 Sonnet as next-stage candidate |
| Whether citations need a post-generation check | **Resolved** — the prompt contract alone was insufficient; a deterministic check is required |
| Whether the trust-model block is load-bearing | **Resolved as inconclusive** — no measurable marginal benefit under this attack |
| `organizationRoles` value conflict | ERRATUM 001 → production readiness |
| Chunks must be generated, not transcribed | Improvement backlog |
| Vocabulary version-pinning to the control-plane extension | Production readiness |
