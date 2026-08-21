# BRIEF.md — model picker

Put this file next to `model-picker.html`, open the folder in Claude Code, and it has everything it needs.

---

## What this is

A single self-contained HTML page that helps a team decide which open-weights LLM to run on their own hardware. They enter what the job needs, what their GPUs cost, and how each candidate scored on their own graded test set. It returns a recommendation, both cost figures, and the reasoning.

It is deliberately one file with no build step, no dependencies, and no network calls. A startup CTO should be able to email it to a colleague who double-clicks it. Filled-in scenarios travel as links; the page prints to one side of A4 for the deck.

## What this measures

Everything on the page is the cost of **serving** a model: hardware, power,
utilisation, peaks, errors in production, renting the same model instead. The one
exception is the fine-tuning column, which is the cost of **getting a model
ready** — the training run, the labelling, the eval harness, someone's week. The
two are kept apart because they behave differently: serving recurs whether or not
anyone touches the model, and tuning recurs only because models and data drift.

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
10. **One colour means one thing.** The accent is money on a cost scale; texture separates the parts of it; muted grey means ruled out and always carries the word too. Nothing is ever coloured by its rank — a bar that changes colour when the winner changes teaches the reader something untrue. A second hue was tried for error costs and failed the readability check in dark mode, which is why it is texture.
11. **The answer comes after the inputs, and says whose numbers it is.** A recommendation printed above the form is a recommendation made from figures nobody entered — and the scores this file ships with are placeholders. So the inputs come first, the answer builds underneath, and until something is touched the verdict is labelled an example rather than a recommendation. A sticky line carries the current answer while the form is on screen, so nothing has to be scrolled to see it change.
12. **The URL is the save file, and old links stay good.** A link written before a field existed is padded from the shipped defaults, so adding one never orphans a scenario already sent to a colleague. Test it by lopping the last value off a link's field list and reopening it — the rest must still restore. Scenarios travel as links, so a model name can arrive from someone else. Anything the user typed is escaped before it reaches the page.

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
- **Mixture-of-experts routing is charged at a flat 65%** of aggregate bandwidth
  whenever a sparse model spans more than one GPU. That is a constant at the top
  of the script, not a measurement: the real cost of all-to-all depends on the
  interconnect. Sparse models remain the least trustworthy figures on the page,
  and the one most worth replacing with a measured number.
- **The task presets are shapes, not measurements.** They set input and output
  length, latency bar and concurrency to typical values for that kind of work.
  Your own traffic is the number that counts, and the quality score must come
  from a test set for the same task.
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
| Kimi K2.6 | 8 GPUs, ~$101.4k/year, ~6.5k tok/s, ~0.8s — 500 GB held, 16 GB read per token |
| Sensitivity | not stable — Qwen2.5 14B and Kimi K2.6 split the eight combinations |
| Sharing on, 3,000,000 other requests/month | real cost ~$0.272 per 1k, you carry about a quarter of the box |
| At 200,000,000 on flat traffic | Qwen2.5 14B wins on cost — $324.7k on 24 GPUs against Kimi K2.6's $432.2k on 32 — because routing a token between eight GPUs is not free |
| Fine-tuning off by default | the table shows serving only, and says so |
| Fine-tuning on, $60,000 once on the small model at 200M requests | it wins — $40.5k a year to serve plus $60.0k to keep tuned, against $324.7k for the best model served untouched |
| Same at 1M requests | it loses — the tuning would have to buy $59.9k a year of quality to be worth doing |
| Task presets | picking Code completion (2s bar) rules Qwen2.5 14B out at 6.9s and moves the recommendation to Kimi K2.6; picking Chat assistant restores every figure above |

Then exercise each correction in turn and confirm it bites:

| Change | Expected |
|---|---|
| **Busiest hour → 3×** | Qwen's peak load goes 12% → 36%; nothing else moves at this volume |
| **Cost of capital → 15%** | Qwen $12.8k → **$16.7k** a year, $1.06 → $1.39 per 1k |
| **KV MB/token → 0.197 on Qwen, concurrency → 128** | 19 GB of KV appears beside the weights, and Qwen fails the latency bar at 19.5s |
| **Requests → 200,000,000 with a 3× peak** | Qwen needs **72 GPUs** and ~$932.1k a year, against 24 GPUs and ~$324.7k on flat traffic — the correction is a factor of three |
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
