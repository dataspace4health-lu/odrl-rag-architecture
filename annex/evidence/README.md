# Prompt Sandbox evidence

One folder per run. Each contains:

- model output for both models, transcribed as produced. Where present, a clearly
  bracketed analyst note follows the output to record pass/fail, truncation or an
  observed design implication; it is not part of the model response
- screenshot of the run (API key strip redacted)
- tokens and cost as displayed by the platform
- completion status: complete, or truncated at the platform output limit

| Run | Scenario | Prompt | Temp | Required? |
|---|---|---|---|---|
| 1A | Normal drafting | v4.1 | 0.1 | required |
| 1B | Temperature contrast, same input | v4.1 | 0.8 | required (parameter comparison) |
| 1C | Retest after prompt revision | v5 | 0.1 | required (prompt iteration) |
| 2 | Citation check | v6 | 0.1 | required |
| 3 | Ungroundable financial term | v6 | 0.1 | required |
| 4 | Ambiguous intent | v6 | 0.1 | required |
| 5 | Indirect prompt injection | v6 | 0.1 | required |
| 5b | Injection ablation, trust instruction removed | v6 variant | 0.1 | **optional** |
| 6 | Governance-boundary refusal | v6 | 0.1 | required |
| 7 | Cross-card contamination | v6 | 0.1 | **optional** |

`run_4` additionally contains a supplemental re-execution performed only to
capture a screenshot, because the contemporaneous one was unavailable. The
original outputs and metrics remain the experiment record; see
`run_4/NOTE_supplemental_screenshot.txt`.

Models: ChatGPT 4o and Claude 4.6 Sonnet, run on the same prompt.
Settings unless noted: temperature 0.1, top-p 1, max length 500, no stop sequence.

## Redaction

The Run 6 prompt, screenshot and accompanying analyst note named a real research
institute in the synthetic scenario. The name is redacted consistently across all
three, marked in place rather than removed silently. The test result is unaffected: the
scenario was a fictitious access request built to check whether the assistant
would refuse to approve a release.
