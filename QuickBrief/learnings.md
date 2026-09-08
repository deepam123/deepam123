# QuickBrief — Key Learnings

Seven lessons from building an iterative agentic analytics system.

---

**1. Design each iteration as a reusable module.**

A high-performing, optimized piece should carry directly into the next iteration. Tracing back on design later is expensive — both in engineering time and in the cost of untangling tightly coupled components.

**2. Modular, atomic LLM functions make evals more objective.**

Multiple functions crammed into a single LLM call become more subjective and harder to measure. When each LLM call has one job, you can point exactly where a failure occurred.

**3. Systems that are easy to explain are easy to test — and reach adoption faster.**

Hence a **workflow should be the first choice, not an agent**. Reach for autonomy only when a fixed pipeline provably cannot answer the question.

**4. Performance first, optimize second.**

Get the system correct and measurable before spending effort making it cheaper or faster. Optimizing a system you cannot yet measure is guesswork.

**5. Choose the right model for the task.**

Don't use an expensive model where a cheaper, smaller one will work. Use public benchmarks to inform the choice. The synthesis step alone accounted for ~55% of latency and ~50% of cost — reducing reasoning effort there improved both without hurting accuracy.

**6. Technical sophistication is not necessarily equal to business sophistication.**

The more impressive architecture isn't automatically the more valuable one. An Iteration 1 workflow that reliably answers *what happened* is more valuable to a business team than an Iteration 3 multi-agent that sometimes answers *why* but is hard to trust.

**7. Structured outputs make deterministic invocation easier and more seamless.**

When LLM outputs follow a defined schema, downstream code can parse and act on them without fragile string matching — reducing a whole class of integration bugs.
