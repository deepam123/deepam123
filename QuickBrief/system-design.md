# QuickBrief — Iterative System Design

This document details the design decisions, constraints, optimizations, guardrails, and eval metrics for each iteration of QuickBrief.

---

## Business Constraints (All Iterations)

| Constraint | Requirement |
|------------|-------------|
| Cost & Latency | ≤ $XX per question; answered within YY minutes. Scales by iteration. |
| Trust | Numbers and recommendations must be verifiable before they ship. |
| Data Scope | Iter. 1–2: database only. Iter. 3: RAG expands to ingested knowledge. |

---

## Iteration 1 — Workflow Agent

**Question answered:** *What happened?*

**Architecture:** Fixed workflow — deterministic pipeline, no model-directed branching.

### Design Decisions
- The Classifier LLM routes the question and extracts intent; it does not touch data.
- A Python function compiled via a YAML semantic layer translates intent into SQL — the LLM never writes SQL directly, eliminating hallucination in query construction.
- The Response Composer LLM synthesizes the result set into a plain-language answer.
- A human analyst reviews and approves before anything ships.

### Cost / Latency Profile
| Factor | Value |
|--------|-------|
| LLM calls | 2 (fixed) |
| Data queries | 1 |
| Total cost | Fixed per question |

### Optimizations
- Relevant context only (no full schema dump)
- Semantic cache for repeated question patterns
- RBAC on data access

### Guardrails
- Human review of all results before delivery
- Read-only data access
- Grounded by semantic layer — prevents SQL hallucination
- Guard against large result sets from SQL execution

### Eval Metrics
| Metric | Type | Description |
|--------|------|-------------|
| Answer Accuracy | North Star | Exact match — is the end output correct? |
| Classification Accuracy | Diagnostic | Did it route the question correctly? |
| Query Groundedness | Diagnostic | Code check — right data, right reporting? |

---

## Iteration 2 — ReAct-Style Bounded Agent

**Question answered:** *Why did it happen?*

**Architecture:** Agent decides which diagnostic queries to run; bounded by a hard code cap.

### Design Decisions
- Built on top of Iteration 1's validated retrieval — Iter. 1 runs first to confirm *what* changed before diagnosing *why*.
- A Think LLM plans the investigation and picks dimensions to test one at a time.
- The stop decision uses a **deterministic threshold check**, not an LLM call — prevents the model from running forever.
- Hard cap of ≤5 diagnostic queries is **enforced by code**, not by instructing the model to stop.

### Cost / Latency Profile
| Factor | Value |
|--------|-------|
| LLM calls | 3–7 |
| Data queries | Up to 5 |
| Optimization option | Async/batching → 50% cost reduction, ~3–4 hours latency |

### Optimizations
- Async / batching (user-selectable): 50% cost reduction at the expense of latency
- Prompt caching for repeated question patterns
- Bounded investigation — stops as soon as a driver is found

### Guardrails
- Hard cap ≤5 tests, enforced by code
- Stop decision is a deterministic threshold check (not an LLM judgment)
- Human review before delivery

### Eval Metrics
| Metric | Type | Description |
|--------|------|-------------|
| RCA Driver Correctness | North Star | Correct drivers with accurate numbers |
| Contribution Calculation Correctness | Diagnostic | Did the math check out? |
| Exploration Failure Rate | Diagnostic | Right data available, but agent never looked |

---

## Iteration 3 — Multi-Agent + RAG

**Question answered:** *Can I trust the recommendation?*

**Architecture:** Adds a Critique LLM and RAG grounding to catch confounders before anything ships.

### Design Decisions
- **Critique LLM** reviews the collected evidence before synthesis — catches confounders that a data-only system cannot see.
- **RAG on past diagnoses** grounds the Think LLM's planning step with historical context.
- **RAG on incident logs** grounds the Critique LLM's review step with known events.
- Critique is capped at ≤2 rounds and shares the same 5-test investigation budget.
- Unresolved disagreement after 2 critique rounds always escalates to a human — the system never loops indefinitely.

### Cost / Latency Profile
| Factor | Value |
|--------|-------|
| LLM calls | 4–9 |
| RAG lookups | 2 |
| Data queries | Up to 5 |

### Optimizations
- Structured pre-filter before RAG (narrow by date/service, then semantic search within that set)
- Parallel investigation where dimensions are independent
- RAG scope widened to ingested knowledge base

### Guardrails
- Critique capped at ≤2 rounds
- Unresolved disagreement escalates to human review — never loops indefinitely
- RAG retrieval inherits source-document permissions (users never see content they couldn't already access)
- Role-Based Access Control throughout

### Eval Metrics
| Metric | Type | Description |
|--------|------|-------------|
| Approved-Answer Correctness | North Star | Same as Iteration 2 |
| RAG Retrieval Relevance | Diagnostic | Did RAG return relevant context? |
| RAG Generation Quality | Diagnostic | Did the LLM actually use RAG correctly? |

---

## Side-by-Side Comparison

| | Iter. 1 | Iter. 2 | Iter. 3 |
|---|---------|---------|---------|
| **LLM Calls** | 2 (fixed) | 3–7 | 4–9 |
| **Queries** | 1 | ≤5 | ≤5 |
| **RAG** | — | — | 2 lookups |
| **Agent Style** | Workflow | ReAct bounded | Multi-agent |
| **Answers** | What happened | Why it happened | Trust the answer |
| **Key Guardrail** | Human review | Hard code cap | Critique + escalation |
| **Key Optimization** | Semantic cache | Async batching | Pre-filter RAG |
