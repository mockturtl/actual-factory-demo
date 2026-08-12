# Cost estimate: completing the 26 letter beads

Estimated on 2026-08-07, based on the two letters that had actually run at that
point (`a` and `b`). All figures are list-price estimates for decision support,
not authoritative charges.

> **Revised 2026-08-07.** The first version of this document overstated every
> absolute cost by a factor of two. Each API call appears twice in the Claude
> Code transcripts — same `requestId` and same message id, different entry uuid —
> and the original pass summed both copies. Figures below are deduplicated by
> `requestId`. The *proportions* in the original were correct; only the absolute
> dollars were wrong.

## Measured baseline

Token usage was read from the Claude Code session transcripts for the factory
agents, not from `gc costs` — the `.gc/usage.jsonl` sink only recorded the mayor
session's model usage, so `gc costs` reports 16 unpriced invocations and a $0.00
total.

Letter `b`, a clean run with no retries:

| Agent | calls | output | cache read | cache write |
| --- | --- | --- | --- | --- |
| planner | 11 | 5,483 | 412,862 | 40,663 |
| builder | 20 | 9,268 | 829,747 | 41,760 |
| architect | 13 | 6,111 | 481,808 | 39,673 |
| reviewer | 15 | 7,754 | 569,866 | 38,845 |
| **total** | **59** | **28,616** | **2,294,283** | **160,941** |

Letter `a`, which needed two extra builder passes, totals 65 calls, 29,110
output tokens, 2,498,556 cache read and 187,144 cache write across five
sessions.

Uncached input is negligible — 114 tokens for the whole of letter `b`.
Effectively everything rides the prompt cache.

## Rates per million tokens

Claude Sonnet 5 introductory pricing applies through 2026-08-31. Cache write is
2× base input at the 1-hour TTL and 1.25× at 5 minutes; cache read is 0.1× base
input at either TTL.

| Model | Input | Output | Cache write (1h) | Cache write (5m) | Cache read |
| --- | --- | --- | --- | --- | --- |
| Claude Opus 5 | $5 | $25 | $10 | $6.25 | $0.50 |
| Claude Sonnet 5 (intro) | $2 | $10 | $4 | $2.50 | $0.20 |
| Claude Sonnet 5 (standard) | $3 | $15 | $6 | $3.75 | $0.30 |

All observed cache writes were at the 1-hour TTL — confirmed from the
`cache_creation.ephemeral_1h_input_tokens` field, which accounts for 100% of the
349,753 raw (pre-dedup) write tokens, with zero at 5 minutes.

## Per-letter cost

| Run shape | Opus 5 | Sonnet 5 intro | Sonnet 5 standard |
| --- | --- | --- | --- |
| clean (letter `b`) | $3.47 | $1.39 | $2.08 |
| two retries (letter `a`) | $3.85 | $1.54 | $2.31 |

## 26 letters

| Scenario | Cost |
| --- | --- |
| Sonnet 5, intro pricing | $36–$40 |
| Sonnet 5, after 2026-08-31 | $54–$60 |
| Opus 5, same work | $90–$100 |

The manager session is always awake. Its cost is roughly $0.58 per hour on
Sonnet 5 intro pricing; at about ten minutes per letter, 26 letters add four to
five hours, or about $3.

**Total Sonnet 5 estimate: $39–$43 at introductory pricing.**

## Where the money goes

For letter `b` on Sonnet 5 intro pricing:

| Component | Cost | Share |
| --- | --- | --- |
| cache write | $0.64 | 46% |
| cache read | $0.46 | 33% |
| output | $0.29 | 21% |

Cache write is the largest single line, and 58% of it (93,662 of 160,941 tokens)
is the first call of each session re-writing a prefix that is nearly identical
across all four agents. See `aa-qfe` for the investigation into why that prefix
is not shared, and what is and is not reducible.

## Caveats

1. **Token counts assumed to hold across models.** Claude Opus 5 and Claude
   Sonnet 5 share the Opus-4.7-family tokenizer, so this is approximately true,
   but it has not been verified against a `count_tokens` baseline.
2. **Retry rate is the largest unknown.** Letter `a` needed two builder retries
   on Opus 5. Sonnet 5 may fail the architect and reviewer gates more often;
   each extra builder pass adds about $0.14 at Sonnet 5 intro pricing. Three
   retries per letter instead of one would push the 26-letter total to roughly
   $45.
3. **Measured on Opus 5 runs.** Both sample letters ran on `claude-opus-5`. The
   Sonnet 5 columns reprice those same token counts and assume behaviour is
   otherwise identical.
