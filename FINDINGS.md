# Findings

Eight findings from ten Prompt Sandbox runs, each with the control it produced in
the target architecture. They are stated at the strength the evidence supports
and no higher.

---

## F1. Temperature is not a grounding control

Identical substantive failures at 0.1 and 0.8 on the same input: the same
collapsed constraints, the same incorrect citation, the same unrequested operand.
The only difference was that a requirement degraded from wrongly refused at 0.1
to silently absent at 0.8.

*Scope:* one sample at each setting. This shows temperature did not remediate the
failures, not that no sampling variance exists.

**Control:** low temperature retained for reproducibility. Not treated as a
safety control.

---

## F2. Prompt level scoping does not hold

Both models added a constraint the request never asked for, across three runs,
the last under an explicit rule stating that a retrieved chunk is not a licence
to add one.

The cleanest evidence came from a deliberately constructed test: an irrelevant
retention chunk placed in context, the dataset card marked "not specified", and
the prompt explicitly forbidding inheritance of a deletion rule. The duty was
drafted anyway.

**Control:** per intent retrieval enforced upstream. Only chunks matching a
requested intent reach the model.

---

## F3. Models normalise what they are told to copy

One model regularised a deliberately asymmetric enum literal into a symmetric
one, at both temperatures. After the prompt named that exact error, it stopped,
then emitted an operand explicitly marked do not emit.

**Control:** operand allow list and verbatim value checking in a deterministic
validator. Transcription cannot be delegated to generation.

---

## F4. Generation did not reliably enforce the governance boundary

A request mixing legitimate constraints with an approval demand. One model
returned an empty refusals array, filed the governance precondition in the wrong
construct, dropped the logical operator, and emitted a response contract
violation. The other produced the correct compound obligation but truncated
before the refusal field, so its behaviour is unobservable and is not scored.

**Control:** a policy boundary engine applied per decomposed intent, not per
request. A gate on the whole request would either block the legitimate drafting
or pass the forbidden element. The boundary has to be drawn through the request.

---

## F5. Resistance without detection

The injected instruction was ignored: no approval statement, citations retained,
no mode change. The injection flag field stayed empty in both the main run and
the ablation.

Removing the trust boundary instruction produced no observable degradation. That
does not show the model's own alignment defeated the attack. The ablation removed
one control of several and cannot isolate base model behaviour.

An attack that is resisted but not detected is an attack that is not logged.

**Control:** independent screening of retrieved content, outside the generator.
Not a field the model reports on itself.

---

## F6. The output contract degraded in the wrong direction

One model truncated in seven of ten invocations. The lost fields were always the
tail: refusals, injection flags, review required. The policy body always
survived. A truncated response therefore reads as a clean draft with the refusal
missing, which is more dangerous than no response.

**Control:** detect incomplete generation and reject the whole response. A
partial structured output never reaches a reviewer. Secondarily, order safety
fields before the policy body so any partial view degrades toward safety.

---

## F7. Every free text field is a leakage surface

One field was identified as the likely injection target and constrained
accordingly. Approval language then appeared in a different free text field that
nobody had constrained.

**Control:** reduce free text rather than police it. Replace the free form field
with a reference key, a controlled enum and a templated label. The reference key
also gives validation a deterministic join, so scoping can be checked in both
directions: every term traces back to a requested intent, and every requested
intent traces forward to a term, a gap or a clarification. The second direction
catches silent omission, which nothing else does.

---

## F8. Citation format drift breaks exact matching

One model carried the retrieved chunk's presentation delimiters into the citation
value. An exact match validator would reject every one.

**Control:** canonical citation validation. A narrow parser may recognise known
wrappers, but any non canonical form is logged as a deviation rather than
silently repaired into validity.

---

## What this adds up to

The controls that held were those where the model's easiest path was already the
safe one: abstaining, asking, citing. The controls that failed were those asking
it to restrain itself against a plausible alternative: not adding a constraint it
had evidence for, not tidying an inconsistent value, not answering a request it
understood.

Prompt controls shape behaviour. They do not enforce it.
