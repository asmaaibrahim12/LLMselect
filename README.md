# Which model should we run?

A single HTML file that works out which open-weights LLM a team should run on
its own hardware — and what it will actually cost them.

You enter what the job needs, what your GPUs cost, and how each candidate scored
on **your own** graded test set. It rules out anything that misses your quality
or latency bar, ranks what is left by real cost, and shows the reasoning.

**[Open the hosted copy](https://claude.ai/code/artifact/dd24be70-ac72-4072-8c5c-ce3d2212551d)** ·
or download [`model-picker.html`](model-picker.html) and double-click it.

No build step, no dependencies, no network calls, no accounts. It works offline
and it works from a file:// URL, because the point is that a CTO can email it to
a colleague who opens it.

---

## What it tells you that a cost calculator won't

**The real cost, not the flat-out cost.** Every online calculator quotes the
price of a GPU running at 100%. Yours will not. At 1,000,000 requests a month on
one H100, the shipped example runs at 11% utilisation — so the true cost is
$1.06 per thousand requests against a flat-out figure of $0.120. That 8.8× gap
is the largest line in most on-premise comparisons and it is the one nobody
quotes. Both numbers always appear together.

**What sparsity does to the bill.** VRAM follows a model's total parameters;
speed follows how many are active per token. Enter both and a mixture-of-experts
stops looking absurd: the shipped example needs eight GPUs to hold Kimi K2.6's
500 GB of weights, yet it reads only 16 GB per token — so at 200M requests a
month it is *cheaper per request* than a dense 14B model, and the recommendation
changes to it. Collapse the two numbers into one, as most calculators do, and
you would price it thirty times too slow.

**What the peak costs you.** Hardware is bought for the busiest hour and paid
for all year. Tell it how much busier your peak is than your average and the
GPU count follows the peak: at 200M requests a month, a normal 3× office-hours
peak takes the shipped example from 22 GPUs to 72, and the bill from $297k to
$932k. Every calculator that sizes on the annual average is wrong by that
factor. The tool also says how full the busiest hour runs, and warns when a
response-time promise stops being credible at that load.

**Whether cost is even your deciding factor.** Below about 25% utilisation you
are paying for the box, not the tokens, and most models on the list cost the
same. The tool says so plainly and tells you to decide on quality instead.

**Whether the answer survives being wrong.** Speeds you have not measured are
estimates from memory bandwidth and can be out by half. Every estimate is varied
by ±50% independently and the whole ranking re-run, so you learn whether the
recommendation is stable enough to act on or whether you need to benchmark
first. On the shipped defaults it is *not* stable — which is the honest answer.

**What the mistakes cost.** Give it the price of one bad answer — a re-do, a
refund, a complaint — and it ranks on hardware plus errors at each model's own
accuracy. On the defaults, twenty cents a mistake is enough to flip the
recommendation from Qwen2.5 14B to the far larger Kimi K2.6, which gets 3.8
fewer answers wrong in every hundred. This is usually the argument that wins the
budget conversation.

**Buy versus build, without a thumb on the scale.** Enter the per-million-token
prices for renting the same model and the two sit in one table. No provider
prices are baked in; they change monthly and would be stale before you opened
the file. Both sides are costed honestly: the on-premise figure includes the
prefill work of reading the prompt, and the rented figure accounts for cached
input at whatever share and discount your provider gives you — the two
corrections that used to push the answer toward owning.

**What sharing the box saves.** Put other workloads on the same hardware and the
cost splits by consumed capacity. On the defaults, adding 3M requests a month of
other work takes this workload from $1.06 to $0.271 per thousand. At low
utilisation this is the biggest saving available anywhere in the tool.

## Things it will not do

It will not invent a quality score. Quality comes from your own graded examples
and nothing else — no bundled benchmark, no default, no download. The quality
bar is the one number nobody can supply for you: on 100 real examples a domain
expert has graded, what score is good enough to ship?

It will not hide a guess. An estimated speed and a measured one never look the
same; the `estimate` and `measured` tags carry that distinction everywhere.

It will not rank an unusable model cheaply. Anything that misses your quality or
latency bar is ruled out before cost is discussed, and shown greyed out so you
can see what you turned down.

## Sharing a scenario

**Copy link** packs every input into the URL — the URL *is* the save file, so a
filled-in scenario mails to a colleague or reopens next week with nothing
stored anywhere. Old links keep working as the tool gains fields.

**Ctrl-P** prints the recommendation, the figures and a summary of the
assumptions to one side of A4, with the input forms dropped. These numbers end
up in board decks.

## Files

| | |
|---|---|
| `model-picker.html` | the tool — the whole thing, 54 KB |
| `BRIEF.md` | the rules it must not break, and how to verify a change |

`BRIEF.md` is the file to read before changing anything. It carries the
verification table: a set of figures the shipped defaults must produce, which
takes about a minute to check in a browser and catches almost any mistake in the
cost model.

## Contributing

Open `model-picker.html` in a browser, make the change, re-check the numbers in
`BRIEF.md`. There are no unit tests and there should not be — the verification
table is faster and catches more.

The first rule is that it stays one file. If a change needs a library, it is the
wrong change.

## Licence

MIT — see [LICENSE](LICENSE). Copy it, change it, ship it.
