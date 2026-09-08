# QuickBrief — Evals

## How Evals Were Set

Same agent. Precomputed answers. Three checks.

We do not ask another AI "does this look right?" We compare the agent's output to an answer key a person already computed.

1. **Ask the question** — The agent sees only the question, same as a real user. The yes/no confirmation pause is skipped so the exam runs autonomously.
2. **Look up the answer key** — A person wrote the trusted query and ran it on the warehouse. That table is the key. The agent never sees it.
3. **Compare, then pass or fail** — All three checks must pass. One miss = wrong.

### The Three Checks

| Check | What we ask |
|-------|-------------|
| Right kind of number | Did it treat this as sales, payments, customers, or a special count? |
| Followed the rules | Did shipping stay out of sales? Did the query actually run? |
| Spreadsheet matches | Do the rows and figures match the answer key? |

---

## The 12 Questions

One question for each counting mistake we cannot afford.

| # | Question | Why it's on the exam |
|---|----------|---------------------|
| 1 | Monthly product sales in 2018, excluding unreliable edge months | Sales = price, and only the year they asked for |
| 2 | Top 10 product categories by product sales | Rank what products sold for, not what cards charged |
| 3 | Product sales by customer state | Slice by the buyer's state |
| 4 | What is the repeat purchase rate? | Loyalty is people who came back, not extra orders |
| 5 | How do customers pay? Payment mix by value | This is payments, not product sales |
| 6 | On-time delivery rate and delay by state | Use our late / on-time flags |
| 7 | How many orders have zero order-item rows? | Do not hide the 775 empty orders |
| 8 | How many orders were placed in October 2018? | The answer is 4 — not a crash |
| 9 | How many unique customers ordered in São Paulo (SP)? | SP is the state, not the city |
| 10 | Total product sales, price only, reliable months | Price only — not shipping |
| 11 | Average customer latitude by state | Do not explode zip codes into extra rows |
| 12 | How much product sales is in uncategorized? | Missing category is a known bucket |

---

## Results — One Complete Run

Every query ran. Five answers were close but not exact.

| Check | Result |
|-------|--------|
| Fully correct | **7 / 12** |
| Right kind of number | 10 / 12 |
| Followed the rules | 12 / 12 |
| Spreadsheet matched | 9 / 12 |

### Passed (7)
Top categories · Sales by state · October 2018 (4 orders) · Unique customers in SP · Total sales without shipping ($13,541,712.78) · Latitude by state · Uncategorized sales

### Missed (5)

| Question | Expected | Returned |
|----------|----------|----------|
| Monthly sales 2018 | 8 months | 20 months (2017+2018) |
| Repeat rate | 3.12% | 6.38% |
| How people pay | 4 methods | 5 methods |
| Delivery by state | Our delivery metric | On-time rates matched; wrong label |
| Orders with no items | 775 as a data check | 775 as a normal count |

---

## Analysis

### What the 7/12 actually says

From one run only. We have not repeated the exam, so we do not know if 7/12 holds next time.

| Claim | Evidence |
|-------|----------|
| Nothing crashed, no forbidden shortcut got through | Rules check: 12/12 |
| Three answers were a different question, not a near miss | Monthly 20 vs 8 · Repeat 6.38% vs 3.12% · Pay 5 vs 4 |
| Two answers had the right sheet and the wrong heading | Delivery: on-time rates matched · Empty orders: 775 matched |
| The traps we already taught it, it passed | Sales $13,541,712.78 without shipping · Map did not explode zips |

### Key Insight — Modular Design Enables Precise Debugging

Modular, atomic LLM functions made it possible to trace exactly where each failure originated — the classifier, the query builder, or the synthesizer. Multiple functions crammed into a single LLM call would have made this far harder.

One thing the trace revealed: response synthesis accounted for ~55% of latency and ~50% of cost. Reducing reasoning effort on the synthesis step improved both without hurting accuracy.

### What we are not claiming

- Not that the warehouse is perfect — only that these 12 queries ran and followed the rules we check.
- Not a go-live score. Until we get 12/12 on more than one run, a person still checks the numbers before anyone acts.
