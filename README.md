# FaithfulRAG-Agent
### A self-auditing multi-agent system that measures and corrects its own hallucinations

**The problem this solves:** every RAG/agent system can retrieve evidence and still generate
claims that evidence doesn't support. Most portfolio projects stop at "I built a RAG chatbot."
This project treats **faithfulness** (does every sentence the model outputs actually follow from
retrieved evidence?) as a first-class, *measured* quantity — and builds an agent loop that uses
that measurement to catch and repair its own hallucinations before returning an answer.

**Why this matters for production AI (and for an interview conversation):** in domains like
healthcare, finance, or law, an agent that's 90% accurate but silently overconfident on the other
10% is more dangerous than one that's 70% accurate and knows when it's unsure. This notebook builds
that self-awareness in explicitly, and *proves* it works with a before/after evaluation rather than
just claiming it.

---

## Architecture

```
                 ┌────────────┐
   claim ──────▶ │  PLANNER   │  decomposes claim into checkable sub-claims
                 └─────┬──────┘
                       ▼
                 ┌────────────┐
                 │ RETRIEVER  │  embeds sub-claims, FAISS search over Wikipedia corpus
                 └─────┬──────┘
                       ▼
                 ┌────────────┐
                 │ GENERATOR  │  drafts an answer, citing retrieved passages
                 └─────┬──────┘
                       ▼
                 ┌────────────┐
                 │  VERIFIER  │  NLI entailment check: does each sentence follow
                 └─────┬──────┘  from its cited evidence?  -> faithfulness score
                       │
             score < τ │  score >= τ
                       ▼            ▼
                 ┌────────────┐   ┌────────┐
                 │  REVISER   │   │ RETURN │  answer + citations + faithfulness score
                 │ (loop back │   └────────┘
                 │ to GENERATOR,
                 │ max 2x)   │
                 └────────────┘
```

We build this twice: once as the full agent above, and once as a **single-shot baseline**
(retrieve once, generate once, no verification) — then run both over the same benchmark so the
improvement from verification is a *measured number*, not a claim.

