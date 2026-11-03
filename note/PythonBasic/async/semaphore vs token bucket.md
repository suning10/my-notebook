

# Semaphore vs Token Bucket — They Are Different Patterns

## What I Used: Semaphore Pattern (Concurrency Limiter)

```
Semaphore(5) — controls HOW MANY run at the same time

    Slots: [ ■ | ■ | ■ | □ | □ ]
             SKU1 SKU2 SKU3  free free

    New task comes in → acquire (takes a slot)
    Task finishes     → release (frees a slot)

    Controls: CONCURRENCY COUNT (not rate)
    Question: "How many are running RIGHT NOW?"
```

---

## Token Bucket Pattern (Rate Limiter)

```
Token Bucket — controls HOW MANY start per unit of TIME

    Bucket capacity: 10 tokens
    Refill rate:     5 tokens/second

    t=0s  [ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ]  10 tokens
    t=0s  5 requests come in → consume 5 tokens
          [ ■ ■ ■ ■ ■ □ □ □ □ □ ]   5 tokens left

    t=1s  refill 5 tokens
          [ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ]  10 tokens again
    t=1s  8 requests come in → consume 8 tokens
          [ ■ ■ □ □ □ □ □ □ □ □ ]   2 tokens left

    Controls: RATE OVER TIME (not concurrency)
    Question: "How many started in the last N seconds?"
```

---

## Side by Side Comparison

```
                    Semaphore               Token Bucket
                    ─────────────────────────────────────────────
What it limits      Concurrent count        Requests per time unit

Core question       How many running now?   How many per second?

Refills?            ❌ No refill            ✅ Refills at fixed rate
                    Slot frees when         Tokens added every N ms
                    task completes

Burst behavior      Allows burst up         Can allow burst up to
                    to N concurrent         bucket capacity, then
                                            throttles to refill rate

Best for            LLM pipelines           External API calls
                    DB connections          (OpenAI, etc.)
                    Thread pools            rate limit = 60 RPM

Analogy             Parking lot             Water bucket with
                    (N spaces total)        a drip refill
```

---

## Code Comparison

```python
# ── SEMAPHORE ─────────────────────────────────────────────────────────────────
# "Only 5 running at the same time"

semaphore = asyncio.Semaphore(5)

async def run_with_semaphore(task):
    async with semaphore:          # acquire slot
        await task()               # run task
    # slot auto-released on exit   # release slot

# Timing: doesn't care WHEN tasks start
# just that no more than 5 overlap


# ── TOKEN BUCKET ──────────────────────────────────────────────────────────────
# "Only 10 tasks can START per second"

import asyncio
import time

class TokenBucket:
    def __init__(self, rate: float, capacity: int):
        self.rate     = rate        # tokens added per second
        self.capacity = capacity    # max tokens in bucket
        self.tokens   = capacity    # start full
        self.last_refill = time.monotonic()

    async def acquire(self):
        while True:
            self._refill()
            if self.tokens >= 1:
                self.tokens -= 1
                return             # got a token, proceed
            sleep_time = 1.0 / self.rate
            await asyncio.sleep(sleep_time)   # wait for refill

    def _refill(self):
        now     = time.monotonic()
        elapsed = now - self.last_refill
        new_tokens = elapsed * self.rate
        self.tokens = min(self.capacity, self.tokens + new_tokens)
        self.last_refill = now


bucket = TokenBucket(rate=5, capacity=10)  # 5 tokens/sec, max 10

async def run_with_token_bucket(task):
    await bucket.acquire()    # wait for a token
    await task()              # no release needed — token is consumed
```

---

## What You Actually Need for Inventory Agent

```python
# ── COMBINE BOTH for production ───────────────────────────────────────────────

_semaphore   = asyncio.Semaphore(5)         # max 5 concurrent pipelines
_token_bucket = TokenBucket(rate=10, capacity=20)  # max 10 LLM calls/sec

async def run_sku_pipeline(sku_gap: SKUGap) -> ResolutionResult:

    # Token bucket — controls rate of NEW pipeline starts
    await _token_bucket.acquire()

    # Semaphore — controls how many run concurrently
    async with _semaphore:
        timeline   = await timeline_agent.run(sku_gap)
        forensics  = await forensics_agent.run(timeline)
        resolution = await advisor_agent.run(forensics)
        return resolution
```

```
500 SKUs arrive simultaneously

Token Bucket gate:           Semaphore gate:
"max 10 start/sec"     →     "max 5 overlap"
                             
t=0s:  10 pass through  →    5 run,  5 queue
t=1s:  10 more pass     →    5 run, 10 queue
t=2s:  10 more pass     →    5 run, 15 queue
...
```

---

## Summary

| Pattern | You Used | What It Is |
|---|---|---|
| **Semaphore** | ✅ acquire/release | Concurrency limiter — slots |
| **Token Bucket** | ❌ not used | Rate limiter — tokens over time |
| **Leaky Bucket** | ❌ not used | Rate limiter — smooth output |
| **Use both** | ✅ Recommended | Concurrency + rate together |

> **Short answer**: The acquire/release I showed is a **Semaphore** (slot-based concurrency limit), not a Token Bucket (time-based rate limit). For hitting external LLM APIs, you ideally want **both** — semaphore to cap concurrency, token bucket to respect RPM limits.