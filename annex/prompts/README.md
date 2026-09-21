# Prompt records

Each file is the **complete single-field input** pasted into the Pluralsight
Prompt Sandbox for that run: instructions, `<retrieved_context>`,
`<dataset_card>` and `<publication_intent>`. The Sandbox exposes one Prompt
field rather than separate system and user roles, so this is the whole input.

| File | Runs | Notes |
|---|---|---|
| `prompt_v4.1_run_1A.txt` | 1A, 1B | Identical input for both. Only the ChatGPT 4o temperature changed, 0.1 → 0.8. |
| `prompt_v5_run_1C.txt` | 1C | Revised after the 1A/1B failures. |
| `prompt_v6_run_2.txt` | 2 | Citation-check mode. |
| `prompt_v6_run_3.txt` | 3 | Ungroundable fee request. |
| `prompt_v6_run_4.txt` | 4 | Adds the explicit clarification clause under NO SILENT DEFAULTS. |
| `prompt_v6_run_5.txt` | 5 | Expanded TRUST MODEL; injection fixture present. |
| `prompt_v6_run_5b.txt` | 5b | **TRUST MODEL section absent.** Otherwise identical to run 5. |
| `prompt_v6_run_6.txt` | 6 | Adds the COMPOUND TEMPLATE RULE for the DAC/permit OR obligation. |
| `prompt_v6_run_7.txt` | 7 | Adds the catalogue/access separation and no-inherited-retention clauses. |

The v6 instruction block evolved slightly across runs as scenario-specific rules
were added. Files are stored per run rather than as a single consolidated
version so that each records what was executed. Retrieved context,
dataset card and publication intent differ per run by design.

Prompt versions are retained deliberately: v4.1 → v5 → v6 records which observed
failure motivated each revision. Rationale is in the experiment log, §3.

---

## A note on verbatim preservation and one redaction

These files are the execution record. They show what was sent, and they are not
edited to match later wording in the case study documents. The one exception is
the publication redaction described below.

One exception, marked visibly rather than made silently. The governance boundary
prompt named a real research institute in its synthetic scenario. For publication
that name is replaced with `[NAMED RESEARCH INSTITUTE, REDACTED FOR PUBLICATION]`.

Nothing in the scenario described that organisation. It was a fictitious access
request constructed to test whether the assistant would refuse to approve a
release, and the test result is unaffected. The redaction is shown in place so
the record remains auditable.
