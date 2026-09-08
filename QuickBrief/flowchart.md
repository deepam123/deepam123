# QuickBrief — Architecture Flowcharts

Mermaid diagrams for all three iterations. Render in GitHub, Notion, or any Mermaid-compatible viewer.

---

## Iteration 1 — Workflow Agent (What Happened?)

Fixed, deterministic pipeline. 2 LLM calls, same cost every time.

```mermaid
flowchart TD
    A([Business Question]) --> B[Classifier LLM\nRoute & extract intent]
    B --> C[Python Function\nCompiled via YAML Semantic Layer]
    C --> D[Build SQL + Execute Query]
    D --> E[Response Composer LLM\nSynthesize answer]
    E --> F{Human Analyst Review}
    F -->|Approved| G([Answer Delivered])
    F -->|Rejected| A
```

**Key properties:**
- Deterministic by design — same question, same number
- Read-only data access with RBAC
- Semantic layer grounds every query
- Guard against large SQL result sets

---

## Iteration 2 — ReAct-Style Bounded Agent (Why Did It Happen?)

Adds root-cause diagnosis. Autonomous bounded loop capped at ≤5 tests.

```mermaid
flowchart TD
    A([Business Question]) --> B[Classifier LLM\nRoute & extract intent]
    B --> C[Think LLM\nPlan diagnostic investigation]
    C --> D{Pick Next Dimension\nor STOP?}
    D -->|Pick Next| E[Python Function\nCompiled via YAML Semantic Layer]
    E --> F[Build SQL + Execute Query]
    F --> G[Evaluate Result\nDoes this explain the change?]
    G -->|No — continue| D
    G -->|Yes — found driver| H[Response Composer LLM\nSynthesize root-cause report]
    D -->|STOP — cap reached| H
    H --> I{Human Analyst Review}
    I -->|Approved| J([Root Cause Delivered])
    I -->|Rejected| A
```

**Key properties:**
- Bounded loop — hard cap of 5 diagnostic queries (code-enforced, not model-directed)
- Stop decision uses a deterministic threshold check, not an LLM call
- Async/batching option for 50% cost reduction at higher latency
- Builds on Iteration 1's validated SQL foundation

---

## Iteration 3 — Multi-Agent + RAG (Trust the Recommendation?)

Adds critique gate and RAG grounding. Bounded at ≤2 critique rounds.

```mermaid
flowchart TD
    A([Business Question]) --> B[Classifier LLM\nRoute & extract intent]
    B --> RAG1[(RAG\nPast Diagnoses)]
    B --> C[Think LLM\nPlan investigation\ngrounded by RAG]
    RAG1 --> C
    C --> D{Pick Next Dimension\nor STOP?}
    D -->|Pick Next| E[Python Function\nCompiled via YAML Semantic Layer]
    E --> F[Build SQL + Execute Query]
    F --> G[Evaluate Result]
    G -->|No — continue| D
    G -->|Yes| H[Collect Evidence]
    D -->|STOP| H
    H --> RAG2[(RAG\nIncident Logs)]
    H --> I[Critique LLM\nCheck evidence & flag confounders]
    RAG2 --> I
    I --> J{Evidence holds?}
    J -->|Yes| K[Response Composer LLM\nSynthesize final report]
    J -->|No — send back\nmax 2 rounds| C
    J -->|Unresolved after 2 rounds| L{Human Analyst Review}
    K --> L
    L -->|Approved| M([Trusted Answer Delivered])
    L -->|Rejected| A
```

**Key properties:**
- Critique LLM catches confounders data-only systems miss
- RAG inherits source-document permissions (RBAC-safe)
- Critique bounded at ≤2 rounds; shares the same 5-test investigation budget
- Unresolved disagreement always escalates to human — never loops indefinitely
