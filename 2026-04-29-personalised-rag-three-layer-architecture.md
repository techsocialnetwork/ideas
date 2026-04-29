# Personalised RAG: Three-Layer Architecture

**Author:** Richard Lofthouse  
**Date:** 2026-04-29  
**Licence:** CC BY-NC-SA 4.0

---

## The Idea

A hybrid AI architecture that combines three distinct layers to produce AI systems that genuinely know the individual user — and improve with every use.

Current AI personalisation approaches are fragmented: RAG retrieves documents but has no user model; memory features store facts but have no structure; agents act but without personal grounding. This architecture unifies all three into a compounding system.

---

## The Three Layers

### Layer 1: Personal Knowledge Graph
**What you know** — your documents, notes, signals, research

Not a flat document store. A structured graph where nodes (concepts, decisions, signals, people) have typed relationships with each other. Query returns a relevant subgraph — not just matching chunks but the connected cluster of knowledge around a topic.

**Key distinction from standard RAG:** Graph traversal surfaces connected knowledge that pure vector similarity search misses. "HBM constraints" returns not just HBM documents but the connected path to energy supply, China dominance, and your macro allocation — because those relationships are stored as edges.

**Implementation:** Vector embeddings + graph structure (Chroma/Qdrant + Neo4j or equivalent)

---

### Layer 2: Personal Memory Schema
**Who you are** — your beliefs, decisions, frameworks, cognitive identity

A structured data model (see [Personal Memory Schema](2026-04-29-personal-memory-schema.md)) capturing:
- Working theories with confidence levels that evolve over time
- Committed decisions that don't need revisiting
- Rejected options with reasons and reopen conditions
- Active project state
- Cognitive style and communication preferences

**Key distinction from existing memory features:** Structured, not narrative. Queryable JSON, not prose summaries. Tracks belief evolution, not just current state.

---

### Layer 3: Agentic Execution with Personal Context Injection
**What gets done** — autonomous agents that act as an extension of the user

Every agent task is pre-loaded with:
- Relevant subgraph from Layer 1 (what the user knows about this topic)
- Relevant memory entries from Layer 2 (user's priors, decisions, style)

The agent acts *as an extension of the user's existing thinking* — not as a generic assistant that happens to have been given some context.

**Key distinction from standard agents:** Agents without personal grounding make locally reasonable but globally wrong decisions for specific users. Pre-injection grounds every action in the user's actual values and knowledge state.

---

## The Compounding Feedback Loop

The critical innovation is the feedback loop that makes all three layers improve over time:

```
User interaction / Agent output
         ↓
New knowledge extracted → added to Layer 1 (KG updated)
New decisions / belief changes → added to Layer 2 (memory updated)
         ↓
Next interaction starts with richer context
         ↓
Repeat — system improves monotonically with use
```

**This is what no existing system does.** OpenAI Memory updates passively but without structure. Mem.ai captures but doesn't feed back into agent execution. Standard RAG is entirely stateless.

The compound effect: after 6 months of active use, a user's personalised system has a detailed model of their domain expertise, working theories, decision history, and cognitive style. A generic AI user gets the same output on day 1 and day 180.

---

## Architectural Diagram

```
┌─────────────────────────────────────────┐
│           User Query / Task             │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│    Layer 2: Personal Memory Schema      │
│    Who is this? What are their priors?  │
│    What have they decided?              │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│    Layer 1: Personal Knowledge Graph    │
│    What do they know about this topic?  │
│    What's connected to it?              │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│    Layer 3: Agent Execution             │
│    Acts with both layers as context     │
│    Output → feeds back to L1 + L2       │
└─────────────────────────────────────────┘
```

---

## Market Gap

As of April 2026, no product ships this architecture as a unified system. Components exist in isolation:

| Component | Existing solutions | Gap |
|---|---|---|
| Personal KG | Obsidian, Roam | No vector search + graph traversal combined |
| Memory layer | OpenAI Memory, Claude Projects | No structure, cloud-owned, no confidence levels |
| Agent execution | LangChain, OpenClaw | No personal context injection pattern |
| Feedback loop | None | Entirely absent from market |

The integration layer — specifically the feedback loop connecting agent output back to both the KG and memory schema — is the core IP.

---

## Prior Art

This architecture was designed, implemented (MVP), and published on April 29, 2026 by Richard Lofthouse.

- Personal Memory Schema: [github.com/techsocialnetwork/personal-memory-schema](https://github.com/techsocialnetwork/personal-memory-schema)
- Working implementation: Chroma vector store + nomic-embed-text-v1, 628 chunks from 348-file knowledge base, semantic search validated
- Article: [tsnmedia.org/the-hybrid-ai-that-knows-you](https://tsnmedia.org/the-hybrid-ai-that-knows-you-why-personalised-rag-agents-will-outperform-everything/)

---

## Citation

```
Lofthouse, R. (2026). Personalised RAG: Three-Layer Architecture.
GitHub: https://github.com/techsocialnetwork/ideas
```
