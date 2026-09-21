# ODRL Policy Drafting Assistant

A design and validation case study for a grounded RAG assistant that turns a data
holder's plain language sharing intent into traceable candidate ODRL access
policy terms for Dataspace4Health (DS4H), Luxembourg's federated health
dataspace.

The subject of this study is not the assistant. It is the architecture around it:
per intent retrieval, provenance, governance boundaries, prompt injection
testing, deterministic validation, human review, and production readiness.

**Start here: [Architecture Pack](docs/architecture/architecture_pack.pdf)**

![Target architecture](docs/architecture/target-architecture.png)

---

## The finding

Safety critical controls could not reliably stay inside generation.

Ten Prompt Sandbox runs across two models, three prompt iterations and two
temperature settings. The governance boundary test mixed legitimate policy
drafting with the one action the system must never take, approving a dataset for
release. It did not hold.

Looking at which controls survived and which did not, the pattern was consistent:

> The controls that worked were the ones where the safe answer was also the easy
> one. The ones that failed were where the model had to hold back from something
> that looked perfectly reasonable.

The target architecture therefore moves governance boundaries, completeness
checking, grounding validation and policy validation into deterministic
components around the model rather than instructions inside it.

Full findings with the evidence behind each: **[FINDINGS.md](FINDINGS.md)**

---

## The problem

A data holder cannot simply publish a dataset into a dataspace. It must carry
machine readable access conditions that the connector evaluates at contract
negotiation time. Translating an intent like *"approved research organisations
only, secondary use, EU only, no re-identification, delete after study"* into
valid ODRL is manual, expert dependent and inconsistent across offerings.

The assistant drafts candidate terms. It never approves anything.

```
assistant  ->  candidate terms, gaps, refusals
                      |
             human data holder review
                      |
             governance approval (DAC / HDAB / DPO)
                      |
             connector evaluates the approved policy at runtime
```

The clearest expression of that boundary: a drafted policy may *require*
governance approval as a precondition. Nothing in the system may assert that the
approval has been granted. Requiring approval and granting approval are
different acts.

---

## Architecture

| Decision | Rationale |
|---|---|
| RAG over a plain LLM | The failure mode without retrieval is fluent invented legal text |
| RAG over fine tuning | The corpus changes independently of the model, and every term needs traceable provenance at inference time |
| Hybrid retrieval | Operand names need exact lexical matching; intents arrive paraphrased and need semantic search |
| Per intent retrieval | A single query over the whole request lets a dominant phrase starve a minor intent of evidence |
| One rule per chunk | The citation anchor travels inside the chunk, so a fabricated citation is detectable |
| Three way output | `candidate_policy`, `gaps[]`, `refusals[]` give the model a legal move that is not invention |
| Deterministic validator | Added after testing showed prompt level grounding rules did not hold |

---

## Experiment

| | |
|---|---|
| Runs | 10, across 2 models and 3 prompt iterations |
| Parameters | 2 temperature settings on identical input |
| Scenarios | normal drafting, citation check, ungroundable financial term, ambiguous input, indirect prompt injection, governance boundary |
| Optional | injection ablation, cross card contamination |

Results, including the failures:

| Scenario | Outcome |
|---|---|
| Citation check | Passed on both models |
| Ungroundable financial term | Neither model invented it |
| Ambiguous input | Both asked rather than guessed |
| Indirect prompt injection | Observable response resisted the attack; detection failed; second model unobservable |
| Governance boundary | **Did not hold** |
| Cross card contamination | Retrieved content became policy despite a direct prohibition |

Every prompt actually executed is preserved in
[`annex/prompts/`](annex/prompts/). Raw model outputs, platform metrics and
screenshots are in [`annex/evidence/`](annex/evidence/).

---

## Repository structure

```
docs/          the case study
  architecture/    architecture pack and diagrams
annex/
  corpus/          retrieval corpus, 43 indexed rules
  prompts/         the exact single field input for each run
  evidence/        raw outputs, metrics and screenshots per run
FINDINGS.md    eight findings, each with the control it produced
```

---

## Scope and limitations

This is a design and validation study. It is not a deployed application, a
production integration, or runnable software.

Stated plainly because the evidence does not support more:

- Retrieval was simulated with hand assembled context. Recall and ranking quality
  were never measured.
- The test environment exposed a single prompt field, so system and user message
  role separation could not be tested. The injection results cover delimiter
  separation only.
- The injection ablation removed one control of several. It cannot isolate base
  model behaviour.
- Two models, one corpus, one task. No general model ranking is implied.
- The authoritative operand vocabulary was unavailable, so one value conflict
  remains unresolved and is documented rather than guessed at.

---

## Provenance and sanitisation

Dataset cards, test scenarios, policy templates and the guidance restatements are
synthetic. The operand vocabulary is different: it contains sanitised semantic
facts extracted from DS4H implementation documentation, specifically operand
names, observed values, namespaces and a deployment baseline. The corpus labels
which is which, and one unresolved value conflict between two source
representations is documented rather than guessed at.

No source implementation documents, production credentials, service endpoints,
infrastructure configuration, verifiable credentials, signed tokens, patient data
or live integration details are included.

Any organisation named inside an execution record appears only in a synthetic
test scenario. It does not describe a real access request, approval or data
transfer.

## Author

Zia Alborzi, Senior Data and AI Solution Architect. Author and architect of this
study.

## Licence

No open-source licence is applied at this time. Third-party trademarks, logos and
platform screenshots remain the property of their respective owners.
