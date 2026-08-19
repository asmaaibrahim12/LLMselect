# BRIEF.md — model picker

Put this file next to `model-picker.html`, open the folder in Claude Code, and it has everything it needs.

---

## What this is

A single self-contained HTML page that helps a team decide which open-weights LLM to run on their own hardware. They enter what the job needs, what their GPUs cost, and how each candidate scored on their own graded test set. It returns a recommendation, both cost figures, and the reasoning.

It is deliberately one file with no build step, no dependencies, and no network calls. A startup CTO should be able to email it to a colleague who double-clicks it.

## Rules that must not break

1. **One file.** No npm, no bundler, no CDN links, no fonts fetched at runtime. If a change needs a library, it is the wrong change.
2. **It works offline.** Opening it with no internet must behave identically.
3. **Never show the flat-out cost alone.** The whole point is the gap between "flat out" and "real" cost. Both figures appear together or neither does.
4. **Estimated speed is always labelled.** A number the tool guessed and a number the user measured must never look the same. The `estimate` / `measured` tags carry that.
5. **No invented quality scores.** Quality comes from the user's own test set. The tool must never ship a default or fetch a benchmark.
6. **Feasibility before cost.** A model that misses the quality or latency bar is ruled out, not ranked cheaply.

## How to verify a change

There are no unit tests. There is something better and faster: open the file in a browser and check these numbers, which come from the defaults it ships with.

With the shipped defaults — 1,000,000 requests/month, 250 output tokens, 92% quality bar, 8s limit, H100 SXM at $30,000 over 3 years, $0.12/kWh:

| Check | Expected |
|---|---|
| Recommendation | Qwen2.5 14B |
| Its estimated speed | ~893 tok/s |
| Its response time | ~4.5s |
| Real cost per 1k | ~$1.06 |
| Flat-out cost per 1k | ~$0.120 |
| Idle penalty | ~8.8× |
| Utilisation | ~11% |
| Llama 3.1 8B | ruled out — misses quality by 3.5 points |
| Llama 3.3 70B | 2 GPUs, ~$25.6k/year |

Then change **requests per month to 200,000,000** and confirm the picture inverts: utilisation goes above 90%, the idle penalty drops to ~1.0×, and GPU counts separate sharply (roughly 13 / 22 / 68). If that inversion stops happening, the cost model is broken — that behaviour *is* the product.

If Claude Code has browser tools, ask it to screenshot the page after every visual change and actually look at it. The math can be right while the layout is broken.

## Backlog — roughly in order

Each of these is a session's work. Do them one at a time and check the numbers above still hold.

1. ~~**Save and reload.**~~ *Done.* "Copy link" packs every input and candidate into the URL hash; opening such a link restores the scenario, and a malformed hash falls back to the defaults. No storage APIs — the URL is the save file.
2. **Print / PDF layout.** These numbers end up in board decks. A `@media print` stylesheet that fits one page and drops the inputs.
3. **Cost of getting it wrong.** A field for what a bad answer costs (a re-do, a refund, a complaint) and a column showing expected cost including errors at each model's accuracy. Often flips the recommendation toward the better model — and it is the argument a CTO actually needs for the budget conversation.
4. **Shared hardware.** Right now each model is priced on its own box. Let several use cases share one GPU and split the cost between them by how much capacity each consumes. This is where the real savings are at low utilisation.
5. **Cloud API comparison column.** Per-million-token pricing entered by the user, next to the on-prem figure, so the buy-versus-build conversation happens in one table. Do not hardcode provider prices — they change and would go stale.
6. **Sensitivity.** Show how the answer moves if the estimated speed is out by ±50%. If the recommendation is stable across that range, they can act on estimates and skip benchmarking; if it flips, they must measure first. That single indicator is worth more than a more precise estimator.

Item 6 is the one worth doing early if the tool is being used to make an actual purchase decision.

## What not to build

No accounts, no backend, no database, no model catalogue fetched from an API, no automatic benchmarking. Every one of those turns a file you can email into a service someone has to operate.

---

## The first prompt to paste into Claude Code

> Read BRIEF.md, then open model-picker.html and read it end to end before changing anything.
>
> Then verify the tool works as described: open it in a browser, confirm the numbers in the "How to verify a change" table, and tell me anything that does not match.
>
> Once that checks out, implement backlog item 1 (save and reload via the URL hash). Keep it to the one file, keep it dependency-free, and re-check the verification numbers when you are done. Show me a screenshot before and after.

After that, each session is the same shape: *"Read BRIEF.md. Implement backlog item N. Re-check the verification numbers. Screenshot it."*

Keep the brief updated as the rules change. It is short on purpose — when it stops being short, the tool has stopped being simple.
