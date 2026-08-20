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

Each of the seven findings from the equation review has been addressed. What
remains is bounded and stated.

- **Latency is service time, not response time.** The tool now reports how full
  the busiest hour is, and warns above 80% that waiting for a slot is excluded.
  It does not model the queue itself: continuous batching is not an M/M/1
  server, and putting a precise number on the wait would be inventing one.
- **Prefill is modelled at 40% of peak arithmetic**, decode at 70% of peak
  bandwidth, tensor parallelism at 80%. Real kernels vary; these are the
  constants at the top of the script, exposed rather than buried.
- **Mixture-of-experts decode is credited with every GPU's bandwidth.** At batch
  this is close to right; for a single stream it is optimistic, because a token
  touches a few experts and all-to-all routing is not modelled. Sparse models
  are the least trustworthy latency figures on the page.
- **The KV cache is only counted if you enter it.** Left blank, the GPU count is
  a floor rather than an answer.
- **The test-set score is used as the production error rate.** On 100 graded
  examples, sampling error is around ±3 points; the sensitivity card now says so
  whenever a candidate sits within 1.5 points of the bar.
- **Shared hardware splits by consumed GPU-seconds**, so a workload that causes
  the peak pays only for its average share of it.

## How to verify a change## How to verify a change

There are no unit tests. There is something better and faster: open the file in a browser and check these numbers, which come from the defaults it ships with.

With the shipped defaults — 1,000,000 requests/month, 500 input and 250 output tokens, 92% quality bar, 8s limit, flat traffic, H100 SXM at $30,000 over 3 years, $0.12/kWh, no cost of capital:

| Check | Expected |
|---|---|
| Recommendation | Qwen2.5 14B |
| Its estimated decode speed | ~893 tok/s |
| Its response time | ~5.0s — service time, prefill included |
| Real cost per 1k | ~$1.06 |
| Flat-out cost per 1k | ~$0.135 |
| Utilisation | ~12% average, ~12% at peak |
| Kimi Linear 48B A3B | ruled out — misses quality by 3.0 points |
| Kimi K2.6 | 8 GPUs, ~$101.3k/year — 500 GB held, 16 GB read per token |
| Sensitivity | not stable — Qwen2.5 14B and Kimi K2.6 split the eight combinations |

Then exercise each correction in turn and confirm it bites:

| Change | Expected |
|---|---|
| **Busiest hour → 3×** | Qwen's peak load goes 12% → 36%; nothing else moves at this volume |
| **Cost of capital → 15%** | Qwen $12.8k → **$16.7k** a year, $1.06 → $1.39 per 1k |
| **KV MB/token → 0.197 on Qwen, concurrency → 128** | 19 GB of KV appears beside the weights, and Qwen fails the latency bar at 19.5s |
| **Requests → 200,000,000 with a 3× peak** | Qwen needs **72 GPUs** and ~$932k a year, against 22 GPUs and $297k when peaks were ignored — the correction is a factor of three |
| **Same, then read the warning** | "At the busiest hour this runs at 95% of capacity" — the queueing caveat |
| **Cached input → 60% at 10% of list** | the rented figure drops by roughly a third |

The 200M case is the one that matters: it is where the old model was most wrong,
and the new one is three times more expensive because hardware is bought for the
peak and paid for all year.

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
