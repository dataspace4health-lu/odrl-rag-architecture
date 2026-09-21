# Production-Readiness Recommendations

What it would take to run this as an enterprise service. Ordered by what the
validation evidence says is load-bearing, not by build effort.

**Framing:** the concept design is not a prototype to harden. Testing showed
that the controls the design depends on most cannot be enforced by generation.
Production is therefore not "the same system, deployed" — it is the same system
**wrapped in deterministic components that the concept deliberately excluded.**

---

## 1. Target architecture

```
Data steward
     │  SSO / Entra ID  ── role: data steward, scoped to own participant org
     ▼
┌──────────────────────────────────────────────────────────┐
│ RAG orchestration service                                 │
│                                                           │
│  1. Input / data classification ── rejects real data,     │
│                                    PHI, credentials       │
│  2. Intent decomposition        ── assigns intent_ref      │
│  3. POLICY BOUNDARY ENGINE      ── classifies EACH intent  │
│       ├─ allowed_intents[]      → continue                │
│       ├─ refused_intents[]      → refusals[], never gen'd  │
│       └─ clarification_intents[]→ clarifications_needed[]  │
│  4. Per-intent retrieval        ── allowed intents only,   │
│                                    hybrid BM25 + vector    │
│  5. Retrieval-content screening ── provenance, patterns,   │
│                                    classifier, quarantine  │
│  6. Prompt assembly             ── system/user roles,      │
│                                    delimited context       │
│  7. LLM                         ── sees allowed_intents[]  │
│                                    only                    │
│  8. Completeness check          ── truncated ⇒ REJECT      │
│  9. Schema + grounding validator── blocking                │
│ 10. Free-text / boundary scan   ── every field             │
│ 11. Compose result              ── merge generated terms   │
│                                    with pre-set refusals   │
└──────────────────────────────────────────────────────────┘
     ▼
Human data holder review  ── mandatory gate
     ▼
Governance approval (DAC / HDAB / DPO)
     ▼
EDC Control Plane ── policy definition → contract definition → negotiation
```

**Why the boundary engine sits at step 3, after decomposition.** Run 6 was a
mixed request: legitimate constraints *plus* an approval demand. A gate applied
to the whole request before decomposition either blocks everything (losing the
legitimate drafting) or passes everything (reproducing the Run 6 failure). The
boundary has to be drawn **through** the request, intent by intent — which is
precisely what neither model did unaided.

**The taxonomy is semantic, not lexical.** Blocking the word "approval" would
break the system's most valuable output:

| Intent | Decision |
|---|---|
| `require_external_approval`, `require_permit` | **allowed** — draft the obligation |
| `assert_requester_is_approved`, `grant_approval`, `authorise_release`, `issue_permit`, `certify_compliance` | forbidden — deterministic refusal |

**The provenance chain the whole design rests on:**

```
publication intent
   → intent_ref (decomposed, controlled intent_code)
   → per-intent retrieval
   → retrieved chunk ID
   → candidate term  { intent_ref, citation }
   → deterministic validation (both directions)
   → human review → governance → EDC-compatible approved policy
```

Checked forward, it catches over-generation. Checked backward — every requested
intent resolving to a term, a gap or a clarification — it catches **silent
omission**, the failure that made Run 1B worse than Run 1A.

Steps 1, 3, 5, 8, 9 and 10 are the additions. The remainder is the concept design.

**Platform mapping (Azure — the DS4H deployment is Kubernetes-based, so managed
services would sit alongside the existing cluster):**

| Component | Service | Note |
|---|---|---|
| Approved corpus | Blob Storage, versioned container | immutable versions, not overwrite-in-place |
| Retrieval index | AI Search, hybrid profile | separate index per corpus version |
| Model access | Azure OpenAI, or a Bedrock/Vertex equivalent | model choice is not architecturally load-bearing |
| Orchestration + validator | Container Apps or Functions | validator is a separate deployable, see §3 |
| Identity | Entra ID | steward role, scoped per participant |
| Audit store | append-only store with retention policy | governance record |
| Analytics | Azure Monitor / Log Analytics | aggregate only |
| Secrets | Key Vault | no credentials in prompts or logs, ever |

AWS and GCP equivalents map cleanly (S3 + Bedrock + OpenSearch + ECS;
Cloud Storage + Vertex + Cloud Run). The architecture does not depend on the
provider; it depends on the validator existing.

---

## 2. Priority 1 — the two controls testing proved are missing

### 2.1 Deterministic output validator (blocking)

Runs on every response before a human sees it. A failed check blocks the draft
and raises a defect, rather than annotating it.

| Check | Evidence that it is needed |
|---|---|
| Schema valid (not merely parseable JSON) | Run 6: GPT-4o emitted `"left_operand": {object}` where a string belongs — valid JSON, contract violation |
| `citation` matches a retrieved chunk ID canonically; known presentation wrappers parsed, anything else logged as a deviation rather than silently repaired | Run 5/6: citations wrapped in `[...]`, unmatchable |
| Every operand in the term supported by *that term's* citation | Runs 1A/1B/1C: DS4H operands under an EHDS citation |
| Every operand in the vocabulary allow-list | v6 required this; a disputed operand was still emitted |
| Operand values byte-identical to the vocabulary | Run 1A/1B: enum normalised to a non-existent literal |
| `runtime_support` **derived** from operands, not accepted as stated | Run 1C: verified operand labelled not-verified |
| Compound logic well-formed (`OR` implies ≥2 conditions) | Run 6: `OR` dropped, `permitIssued` omitted |
| Every term's `intent_ref` is in `allowed_intents[]` | Runs 1A/1B/1C/7: unrequested constraints |
| Every allowed intent resolves to a term, gap or clarification | Run 1B: approved-user requirement silently dropped |
| Response is complete, not truncated | Claude truncated in 7 of 10 invocations |

Deterministic, testable, and independent of model choice — which matters, because
model behaviour changed between versions during a single evening of testing.

### 2.2 Policy Boundary Engine

Per intent, not per request — see §1. Two-sided, because the model failed on both
sides in Run 6.

- **Before generation:** each decomposed intent is classified against the
  taxonomy. Forbidden intents populate `refusals[]` deterministically and never
  reach the model. Allowed intents proceed to retrieval.
- **After generation:** an independent scan of every field — including
  `display_label` and any remaining free text — for approval, eligibility or
  compliance assertions. Block on match.

This is the control for RK3, the highest residual risk in the register.

### 2.3 Completeness gate

A truncated structured response is invalid and is rejected in full. It is retried
or escalated; it is never shown to a reviewer as a partial result. Secondarily,
safety fields precede the policy body so any streaming or diagnostic view
degrades toward safety. Truncation rate is a monitored signal.

---

## 3. ODRL validation service

This study treats structural validity as a recommendation rather than
a runtime guarantee. In production it becomes a real component, and it is
**separate from §2.1**: the output validator checks the *contract*; this checks
the *policy*.

- Serialise the approved candidate into EDC-namespace ODRL JSON-LD.
- Validate against an ODRL profile / SHACL shapes.
- Dry-run against a non-production Control Plane policy-definition endpoint.

Deploy as an independent service so it can be versioned against the connector
rather than against the assistant.

---

## 4. Corpus refresh and vocabulary synchronisation

**The single most important operational requirement**, and the one with the most
concrete evidence behind it.

Two authorities, and they are not the same thing. New policy constraints are
registered in the Control Plane policy extension — **runtime registration is the
executable authority** — and `edc-controlplane/ARCHITECTURE.md` documents them
afterwards, making it the *documented* authority. Neither was available during
this study, which is why the vocabulary is labelled "verified in current DS4H
source material" rather than complete. ERRATUM
001 documents a live consequence: two representations of the same policy in one
source pack disagree on an operand value, and the disagreement is invisible unless
both are read.

Requirements:

- **Generate** the vocabulary from the deployed Control Plane policy registration
  for a pinned release, and cross-check it against `ARCHITECTURE.md`.
  Documentation alone is never treated as runtime truth. Never hand-transcribe —
  two citation-ID drift incidents occurred during this project's own design
  reviews from retyping.
- **Version-pin** the corpus to a control-plane release. A connector upgrade
  triggers a vocabulary regeneration and a diff review before the index is rebuilt.
- **Fail closed** on unknown operands: route to a gap, never improvise.
- **Carry `source_status` into citations** so draft-status guidance
  (HealthDCAT-AP) is visible to the reviewer at the point of decision.
- **Named owner** with a scheduled review cadence, plus an event-driven trigger on
  connector upgrade.

Resolve ERRATUM 001 as part of the first production vocabulary generation.

---

## 5. Monitoring

**Leading indicator: reviewer edit rate.** When stewards begin editing the same
term repeatedly, the corpus is wrong before any automated metric says so. This is
the only signal grounded in ground truth, because it is the only one produced by
someone who knows the right answer.

| Signal | Alert on |
|---|---|
| Validator rejection rate by check | step change after a model, prompt or corpus version change |
| Reviewer edit / reject rate by term type | sustained rise on a specific term |
| Retrieval drift | chunks that stop being retrieved after a corpus update |
| Grounding drift | citations not present in the retrieved set (should be zero post-validator) |
| Injection detection rate | any change; repeated probing of one corpus document |
| Gap and refusal counts by type | shift in distribution |
| Truncation rate | any — see §7 |
| Latency and cost per draft | budget tracking only |

Model, prompt and corpus versions are all logged, so a behaviour change can be
attributed to one of them. This matters: during testing, prompt v4.1→v5→v6
changed failure modes substantially with no model change at all.

---

## 6. Cost

Measured across the full ten-run experiment: **$0.0725 total.** GPT-4o averaged
1,989 displayed tokens per run at ~$0.0016/1k; Claude 4.6 Sonnet averaged 1,123 at
~$0.0036/1k — 44% fewer tokens, 29% higher cost.

**These are Sandbox-displayed metrics at experiment scale, and they do not
establish production TCO.** Unknown: production input/output token accounting,
provider pricing, average draft size, retrieval and indexing cost, screening
classifier cost, and reviewer handling time.

What can be said: inference cost was negligible at this scale, and for the
intended low-volume steward workflow human review is *likely* to dominate
variable cost — **to be confirmed by pilot measurement, not assumed.** Track cost
per accepted draft, reviewer handling time, retrieval/index cost and inference
cost separately.

Cost controls that do matter: cache retrieval results per corpus version; do not
re-run generation on validator failure without a diagnosis (a retry loop on a
systematic fault multiplies cost and produces nothing).

---

## 7. Output contract and truncation

**Primary:** detect truncation and reject the whole response. Retry or escalate;
never present a partial result. Field ordering should not matter because nothing
partial is shown.

**Secondary:** emit `status`, `refusals`, `injection_flags` and `review_required`
before `candidate_policy`, so any streaming view, diagnostic dump or partial log
degrades toward safety rather than toward an apparently complete draft.

**Operational:** set a generous output budget and monitor the truncation rate.
Claude truncated in 7 of 10 invocations under the 500-token cap; a production cap
set carelessly reproduces that silently.

---

## 8. CI/CD, versioning and rollback

Four independently versioned artefacts: **model**, **prompt**, **corpus**, and
**validator rules**. Any of them can change behaviour, so each needs its own
version and its own rollback path.

Pipeline:

```
change (prompt / corpus / validator / model version)
    → core regression:        S1–S7  (S7 included — it caught the cleanest
                                       instance of retrieved data becoming policy)
    → adversarial regression: multiple S5 injection variants
    → diff against the previous run's pass/fail matrix
    → staging deployment
    → canary on a subset of stewards
    → full rollout
```

The scenarios become the regression suite. That is their long-term value — they
were designed to catch specific failure modes, and those failure modes will recur
on the next model version.

The 5b ablation is a **diagnostic experiment, not a release gate.** Run it when
the trust-control or prompt architecture changes, to re-measure which layer is
load-bearing; do not run it on every deployment.

Rollback: corpus versions are immutable and indexed separately, so rollback is an
index pointer change. Prompt versions are stored with their hash. Model version is
pinned explicitly, never "latest".

---

## 9. Ownership and governance checkpoints

| Artefact | Owner | Approval |
|---|---|---|
| Refusal rules (`/system`) | platform security | two-person |
| Guidance corpus (`/guidance`) | corpus owner | governance sign-off |
| Operand vocabulary | control-plane owner | generated, diff-reviewed |
| Prompt versions | solution owner | change record + regression pass |
| Validator rules | platform security | two-person |
| Model version | solution owner | regression pass |
| A drafted policy | data steward | governance (DAC/HDAB) |

Separation of duties: the person who edits the corpus does not approve policies
drafted from it.

**Governance checkpoints:** corpus change review; connector-upgrade vocabulary
diff; quarterly review of reviewer edit rates and the gap/refusal distribution;
annual review of the escalation matrix against DS4H governance structure.

---

## 10. Phased roadmap

| Phase | Scope | Exit criterion |
|---|---|---|
| **1. Boundary engine + validator** | Build §2.1, §2.2 and §2.3 against the recorded Sandbox outputs as fixtures. No new model work. | Every known-bad fixture is rejected by the *appropriate* control — boundary engine for governance intent, schema validator for the nested operand, grounding validator for misaligned citations, vocabulary validator for unsupported operands, completeness gate for truncation — and every known-good fixture is accepted |
| **2. Corpus, retrieval, vocabulary** | Real corpus, hybrid index, per-intent scoping with `intent_ref`, retrieval-content screening, generated vocabulary pinned to a Control Plane release | Unrequested-constraint rate → 0 on the regression suite; both provenance directions verifiable |
| **3. ODRL serialisation + connector validation** | EDC-namespace JSON-LD, profile/SHACL validation, dry-run against a non-production Control Plane | Drafts load into a test connector without manual correction |
| **4. Synthetic steward pilot** | 2–3 stewards, synthetic dataset cards, full audit trail | Reviewer edit rate stable; no approval language escapes the gate; RK7 measured |
| **5. Limited production** | Real dataset cards and metadata, one participant organisation | Governance sign-off; time-to-publish measurably reduced |

**Phase 1 is deliberately first, and deliberately model-free.** The validation
evidence shows the deterministic control layer is the immediate prerequisite
before further model optimisation. Building it against the ten recorded outputs as fixtures means it can be
proven correct before any new generation happens.

**Connector validation precedes the steward pilot.** Asking stewards to judge
draft quality before drafts can survive deterministic structural and connector
compatibility checks wastes the scarcest resource in the programme — expert
attention — on defects a machine can find.

Note that Phase 5 says *real dataset cards and metadata*, not real datasets. The
assistant never needs patient-level data at any phase.

---

## 11. What would change the recommendation

- **Role separation available.** The injection findings are provisional; the
  Sandbox could not test message-role isolation. Re-run the suite before relying
  on any conclusion about injection resistance.
- **Control Plane source or documentation obtained.** Would let the supported DS4H
  operand vocabulary be completed and version-pinned, and would resolve ERRATUM
  001. It would **not** by itself convert the "not verified in current source"
  labels into a machine-enforceable / contractual-only split: generic ODRL duties
  and prohibitions would still need their runtime enforcement semantics verified
  separately before being labelled either way.
- **Reviewer behaviour observed.** RK7 (automation bias) is untested. If stewards
  accept drafts without reading them, every control upstream is decorative, and
  the design would need a forcing function rather than a presentation choice.
