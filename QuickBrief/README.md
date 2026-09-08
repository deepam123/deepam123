# QuickBrief — Your "Always-On" Analyst

> ⚠️ **MVP** — Early-stage proof of concept. Numbers should be verified by a person before acting on them.

Business teams wait days for an analyst to turn a raw question into data and insights. QuickBrief does it end-to-end, in minutes, using an agentic AI pipeline grounded in a semantic data layer.

**Team:** Akash J, Abhishek A, Deepam A, Lauren S, Manswani R, Nish P, Vijay K

---

## What It Does

QuickBrief takes a plain-language business question and returns a grounded, verifiable data insight — no analyst handoff required.

It is built in three progressive iterations, each answering a harder question while staying within strict cost, latency, and trust constraints.

| Iteration | Style | Answers |
|-----------|-------|---------|
| 1 — Workflow Agent | Deterministic pipeline, 2 LLM calls | *What happened?* |
| 2 — ReAct Bounded Agent | Autonomous loop, hard cap ≤5 tests | *Why did it happen?* |
| 3 — Multi-Agent + RAG | Critique gate + RAG grounding | *Can I trust the recommendation?* |

---

## How We Think About Each Iteration

### Iteration 1 — Workflow Agent

The foundation. A fixed, deterministic pipeline: a Classifier LLM routes the question, a Python function compiled via a YAML semantic layer builds the SQL (the LLM never writes SQL directly), and a Response Composer synthesizes the answer. Same question always returns the same number. A human analyst reviews before anything ships.

The trade-off: it tells you *what* changed, not *why*.

### Iteration 2 — ReAct Bounded Agent

Adds root-cause diagnosis on top of Iteration 1. A Think LLM plans an investigation and tests candidate dimensions one at a time, stopping the moment a dimension explains the change — or when a hard cap of 5 tests is hit. The cap is enforced by code, not by asking the model to stop. An async/batching option cuts cost by ~50% at the expense of latency.

The trade-off: more powerful, harder to predict exactly how many queries it will run.

### Iteration 3 — Multi-Agent + RAG

Adds a Critique LLM that reviews collected evidence before synthesis, catching confounders (seasonality, known incidents) that a data-only system cannot see. RAG grounds both the planning step (past diagnoses) and the critique step (incident logs). Critique is capped at ≤2 rounds; unresolved disagreement escalates to a human rather than looping indefinitely.

The trade-off: highest coverage, most complex to explain and test.

---

## Tech Stack

| Layer | Tool | Role |
|-------|------|------|
| **Agent Framework** | [LangGraph](https://github.com/langchain-ai/langgraph) | Orchestrates the agentic workflows — stateful graphs for the bounded ReAct loop (Iter. 2) and multi-agent critique flow (Iter. 3) |
| **LLM** | OpenAI GPT models | Classifier, Think, Critique, and Response Composer nodes |
| **Embeddings** | OpenAI Embeddings API | Converts past diagnoses and incident logs into vectors for RAG retrieval |
| **Vector Store** | [ChromaDB](https://www.trychroma.com/) | Stores and retrieves embedded documents for RAG in Iter. 3 (past diagnoses + incident logs) |
| **Semantic Layer** | YAML + Python | Defines the data model and compiles intent into SQL — the LLM never writes SQL directly |
| **Observability** | [Arize AI](https://arize.com/) | Traces every LLM call and data query; captures latency, token usage, and model inputs/outputs for debugging and eval |
| **Dataset** | [Brazilian E-Commerce (Olist)](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) | ~100K orders from a Brazilian marketplace; used to evaluate the agent against real business questions |

---

## Eval Strategy

We do not ask another AI "does this look right?" We compare the agent's output to an answer key a human computed in advance. Three checks must all pass for a question to count as correct.

| Check | What We Ask |
|-------|-------------|
| Right kind of number | Sales? Payments? Customers? Special count? |
| Followed the rules | No forbidden shortcuts (e.g., shipping included in sales)? Query ran? |
| Spreadsheet matched | Do rows and figures match the answer key? |

The exam has 12 questions — one for each counting mistake we cannot afford.

**Current results (one complete run):**

| Check | Result |
|-------|--------|
| Fully correct | **7 / 12** |
| Right kind of number | 10 / 12 |
| Followed the rules | **12 / 12** — no forbidden shortcuts |
| Spreadsheet matched | 9 / 12 |

Every query ran. Five answers were close but not exact — three answered a different question entirely, two had the right data with the wrong label. See [evals.md](./evals.md) for the full breakdown.

This is a single run. We are not claiming go-live readiness. Until we hit 12/12 on more than one run, a person checks the numbers before anyone acts on them.

---

## Documentation

| File | What's Inside |
|------|---------------|
| [system-design.md](./system-design.md) | Architecture decisions, cost/latency profiles, guardrails, and eval metrics for each iteration |
| [flowchart.md](./flowchart.md) | Mermaid diagrams showing the data flow for each iteration |
| [evals.md](./evals.md) | Full eval methodology, the 12 questions, and detailed results |
| [learnings.md](./learnings.md) | Seven lessons from building an iterative agentic analytics system |

---

## Code

The implementation lives in a separate repository:

**→ [github.com/deepam123/quickbrief](https://github.com/deepam123/quickbrief)**

The codebase covers Iteration 1 (the workflow agent). Iterations 2 and 3 are designed and documented here but not yet implemented — this is an MVP.

---

## Business Constraints

All iterations operate under these non-negotiables:

- **Cost & Latency** — scales by iteration: 2 fixed LLM calls (Iter. 1) up to 4–9 calls + 2 RAG lookups (Iter. 3)
- **Trust** — every answer is verifiable: semantic layer grounding in Iter. 1–2, plus a critique gate in Iter. 3
- **Data Scope** — Iter. 1–2 stay database-only; Iter. 3 adds RAG over an ingested knowledge base

---

## Status: MVP

Iteration 1 is built and evaluated. Iterations 2 and 3 are fully designed (see [system-design.md](./system-design.md)) but not yet implemented. Next step is getting to 12/12 on evals before expanding scope.
