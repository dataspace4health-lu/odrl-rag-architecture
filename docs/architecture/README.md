# Architecture

[**architecture_pack.pdf**](architecture_pack.pdf) is the full seventeen slide
walkthrough: problem, scope, boundary, why RAG, conceptual architecture, corpus
trust zones, chunking and retrieval, the experiment, findings, validation
results, model comparison, target architecture, risk register and roadmap.

Three diagrams are extracted here for reading in context.

## Target production architecture

The policy boundary engine sits after intent decomposition, not before. The
governance test was a mixed request, and a gate on the whole request would
either block the legitimate drafting or pass the forbidden element.

![Target production architecture](target-architecture.png)

## The architectural boundary

The assistant drafts. Humans review, governance approves, and the connector
enforces the approved policy at runtime. A drafted policy may require governance
approval as a precondition; nothing in the system may assert that the approval
was granted.

![The architectural boundary](architectural-boundary.png)

## The failure that changed the design

A request mixing legitimate policy constraints with an approval demand. This is
the evidence behind the policy boundary engine and the post generation boundary
scan.

![The governance boundary failure](governance-failure.png)
