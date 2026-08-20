# BRIEF.md — model picker

Put this file next to `model-picker.html`, open the folder in Claude Code, and it has everything it needs.

---

## What this is

A single self-contained HTML page that helps a team decide which open-weights LLM to run on their own hardware. They enter what the job needs, what their GPUs cost, and how each candidate scored on their own graded test set. It returns a recommendation, both cost figures, and the reasoning.

It is deliberately one file with no build step, no dependencies, and no network calls. A startup CTO should be able to email it to a colleague who double-clicks it. Filled-in scenarios travel as links; the page prints to one side of A4 for the deck.

## Rules that must not break

1. **One file.** No npm, no bundler, no CDN links, no fonts fetched at runtime. If a change needs a library, it is the wrong change.
2. **It works offline.** Opening it with no internet must behave identically.
3. **Never show the flat-out cost alone.** The whole point is the gap between "flat out" and "real" cost. Both figures appear together or neither does.
4. **Estimated speed is always labelled.** A number the tool guessed and a number the user measured must never look the same. The `estimate` / `measured` tags carry that.
5. **No invented quality scores.** Quality comes from the user's own test set. The tool must never ship a default or fetch a benchmark.
6. **Feasibility before cost.** A model that misses the quality or latency bar is ruled out, not ranked cheaply. Pricing errors does not change that — it re-ranks the models that qualify, it does not readmit one that failed.
7. **Both themes come from the tokens.** Light on bare `:root`, dark under both `prefers-color-scheme` and an explicit `data-theme`, print overriding both. No colour is defined only inside a theme block — that is how a page ends up with one theme's text on the other theme's background.
8. **Total and active are different numbers.** VRAM, and so the GPU count, follows the total parameter count; decode speed follows what is read per token. Collapsing the two makes a mixture-of-experts look thirty times slower than it is.
9. **No figure from a missing number.** An empty or zero input that would divide by zero stops the output and names the field. A cost worked out from a gap looks like an answer without being one.
10. **The URL is the save file.** Scenarios travel as links, so a model name can arrive from someone else. Anything the user typed is escaped before it reaches the page.

## Known limits of the model

These are deliberate simplifications, not bugs, but they bound what the answer
is worth. Anyone quoting these figures should know them.

- **Input tokens are not modelled on the on-premise side.** Sizing counts output
  tokens only; prefill is ignored. The cloud comparison *does* charge for input
  tokens, so at 500 input tokens the box is understated by roughly 20% and the
  buy-versus-build call leans toward owning.
- **Latency assumes the batch is always full.** Response time is computed at the
  stated concurrency even at 11% utilisation, where real concurrency is nearer
  1.7 and a request would come back in about 3.0s rather than 4.5s. This errs
  safe, but it can rule out a model on latency that would have passed.
- **The test-set score is used as the production error rate.** On 100 graded
  examples the sampling error is around ±3 points — wider than the 2.5-point gap
  that decides the recommendation once errors are priced.
- **KV cache is excluded from VRAM**, and there is no cost of capital on the
  capex.

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
| Kimi Linear 48B A3B | ruled out — misses quality by 3.0 points |
| Kimi K2.6 | 8 GPUs, ~$101.3k/year — 500 GB of weights, but only 16 GB read per token |
| Cost of a wrong answer | 0 by default — every figure above is the hardware-only comparison |
| Cloud comparison | off by default — the table gains a rented column only when it is switched on |
| Sharing on, 3,000,000 other requests/month | real cost ~$0.271 per 1k, 43% used, you carry 25% of the box |
| Print (Ctrl-P) | one A4 page, no input forms, assumptions summarised at the top |
| Sensitivity | not stable — Qwen2.5 14B and Kimi K2.6 win four of the eight combinations each |

Then set **cost of a wrong answer to $0.20** and confirm the recommendation flips to Kimi K2.6: it costs $88.6k a year more in hardware but gets 3.8 fewer answers wrong in every hundred, which at that price is worth more. Below about $0.19 it does not flip. That crossover is the point of the field.

Then change **requests per month to 200,000,000** and confirm the picture inverts: utilisation goes above 75% for every candidate, the idle penalty drops to ~1.0×, and **the recommendation itself changes to Kimi K2.6** at ~$0.090 per 1k against Qwen's ~$0.124. That inversion is the product. A trillion-parameter model beating a 14B one on cost per request is not a bug: it reads 16 GB per token against the dense model's 28 GB, and at high utilisation the token cost is what you are paying. GPU counts separate to roughly 3 / 22 / 16.

If Claude Code has browser tools, ask it to screenshot the page after every visual change and actually look at it. The math can be right while the layout is broken.

## Backlog — roughly in order

Each of these is a session's work. Do them one at a time and check the numbers above still hold.

1. ~~**Save and reload.**~~ *Done.* "Copy link" packs every input and candidate into the URL hash; opening such a link restores the scenario, and a malformed hash falls back to the defaults. No storage APIs — the URL is the save file.
2. ~~**Print / PDF layout.**~~ *Done.* Printing drops the forms, the controls and the explanatory notes, and puts a one-line summary of the assumptions at the top so the figures still mean something on their own. Fits one A4 page with three candidates, priced errors or not, and prints on white even from dark mode.
3. ~~**Cost of getting it wrong.**~~ *Done.* One optional field, defaulting to 0 so the hardware-only comparison is unchanged until it is filled in. Ranking then runs on hardware plus errors, the chart stacks the two, and a callout states the trade in the form a budget conversation needs: this much more hardware buys that much less rework. Models below the quality bar are still ruled out first — errors do not buy a bad model back in.
4. ~~**Shared hardware.**~~ *Done.* Section 4 adds other workloads to the same box. Sizing works in tokens rather than requests so the other side can have its own response length, and cost is split by consumed capacity — idle included, which is what makes filling the box pay. Every figure then shown is your share, and the columns say so.
5. ~~**Cloud API comparison column.**~~ *Done.* Off by default; switching it on adds per-million input and output prices per candidate, an average input length, and a rented-per-year column beside the on-premise one. Prices are the user's to enter — nothing is hardcoded. The callout states which way the decision goes, and gives the crossover volume only when there is spare capacity for one to exist.
6. ~~**Sensitivity.**~~ *Done.* Every estimated speed is varied by ±50% independently and the whole ranking re-run — all combinations up to 8 guesses, adversarial samples beyond that. The card says whether the recommendation survives that range, and names what wins instead when it does not. Measured speeds are held fixed.

The backlog is empty. Anything added from here should have to argue for itself against the rules above — particularly the first one.

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
