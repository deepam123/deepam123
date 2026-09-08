# QuickBrief — Your "Always-On" Analyst

> Business teams wait days for an analyst to turn a raw question into data and insights. QuickBrief does it end-to-end, in minutes.

**Team:** Akash J, Abhishek A, Deepam A, Lauren S, Manswani R, Nish P, Vijay K

---

## What It Does

QuickBrief is an AI-powered analytics system that takes a plain-language business question and returns a grounded, verifiable data insight — no analyst handoff required.

It is built in three progressive iterations, each adding more reasoning capability while staying within strict cost, latency, and trust constraints.

| Iteration | Agent Style | Answers |
|-----------|-------------|---------|
| 1 | Workflow (deterministic pipeline) | What happened? |
| 2 | ReAct-style bounded agent | Why did it happen? |
| 3 | Multi-agent + RAG | Can I trust the recommendation? |

---

## Repository Structure

```
QuickBrief/
├── README.md               ← You are here
├── system-design.md        ← Iteration-by-iteration architecture details
├── flowchart.md            ← Mermaid flowcharts for all 3 iterations
├── evals.md                ← Evaluation methodology and results (7/12)
└── learnings.md            ← Key learnings from the capstone
```

---

## Business Constraints

- **Cost & Latency** — scales by iteration (Iter. 1: 2 LLM calls fixed; Iter. 2: 3–9 calls + ≤5 queries; Iter. 3: adds 2 RAG lookups)
- **Trust** — verifiable before shipping via grounding checks (Iter. 1–2) and a critique gate (Iter. 3)
- **Data Scope** — Iterations 1–2 stay database-only; Iter. 3's RAG widens scope to ingested knowledge

---

## Quick Summary of Each Iteration

### Iteration 1 — Workflow Agent (What Happened?)
Fixed, deterministic pipeline. 2 LLM calls every time. A Python function compiled via a YAML semantic layer builds the SQL — the LLM does not write SQL. Same question always returns the same number. Trade-off: tells you *what* changed, not *why*.

### Iteration 2 — ReAct-Style Bounded Agent (Why Did It Happen?)
Adds root-cause diagnosis. Tests candidate dimensions one at a time instead of guessing one explanation. Stops the moment a dimension explains the change (or hits the hard cap of 5 tests). Cost stays bounded despite autonomy — the cap is enforced by code, not by asking the model to behave.

### Iteration 3 — Multi-Agent + RAG (Trust the Recommendation?)
Adds a critique pass before anything ships, catching confounders a data-only system cannot see. RAG grounds both planning and critique in past diagnoses and known incident logs. Critique is capped at ≤2 rounds; unresolved doubt escalates to a human.

---

## Eval Snapshot

One complete run on 12 questions. Every query ran. 5 answers were close but not exact.

| Check | Result |
|-------|--------|
| Fully correct | 7 / 12 |
| Right kind of number | 10 / 12 |
| Followed the rules | 12 / 12 |
| Spreadsheet matched | 9 / 12 |

See [`evals.md`](./evals.md) for the full breakdown.
