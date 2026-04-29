# Personal Memory Schema (PMS)

**Author:** Richard Lofthouse  
**Date:** 2026-04-29  
**Status:** Published — [Full spec and implementation](https://github.com/techsocialnetwork/personal-memory-schema)  
**Licence:** CC BY-NC-SA 4.0

---

## The Idea

A structured JSON data model that captures a user's cognitive identity for use as persistent context in AI systems.

Distinct from simple fact storage ("user's name is Richard") or document retrieval (RAG). PMS captures the user's *mental model*: what they currently believe, how confident they are, what they've decided, what they've rejected, and how their thinking has evolved over time.

---

## Core Innovation

**Beliefs with confidence levels that change over time.**

No existing AI memory system tracks the evolution of beliefs. PMS stores:

- `working_theories` — active hypotheses with confidence scores (0-100)
- `confidence_history` — log of every confidence change with trigger and date
- `committed_decisions` — things decided; don't re-litigate
- `rejected_options` — things considered and ruled out, with reasons and reopen conditions
- `belief_history` — full audit log of how the user's thinking evolved

---

## Why It's Different

| Feature | OpenAI Memory | Mem.ai | PMS |
|---|---|---|---|
| Structured belief model | ❌ | ❌ | ✅ |
| Confidence levels per belief | ❌ | ❌ | ✅ |
| Belief revision history | ❌ | ❌ | ✅ |
| Committed decisions vs open questions | ❌ | ❌ | ✅ |
| Local-first / user-owned | ❌ | ❌ | ✅ |
| Compounding feedback loop | Partial | ❌ | ✅ |

---

## The Compound Effect

Every interaction enriches the model. After sustained use, the system has a detailed map of how the user thinks, what they've concluded, and what evidence moved their beliefs. Generic AI gives the same output on day 1 and day 180. PMS-grounded AI improves monotonically.

---

## Full Implementation

See: [github.com/techsocialnetwork/personal-memory-schema](https://github.com/techsocialnetwork/personal-memory-schema)

Includes: full JSON schema, example data, management CLI, and technical specification.
