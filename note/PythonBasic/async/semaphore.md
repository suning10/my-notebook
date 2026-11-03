# Why use async.semaphore

- to limit the # of cucurrency
- What else need to be consider 


# how to use it 



### things need to be consider

# ── 1. SEMAPHORE — prevent API rate limit explosions ───────────────────────
semaphore = asyncio.Semaphore(5)  # max 5 SKUs running at once
# Rule of thumb: (API rate limit / avg tokens per pipeline) = max_concurrent


# ── 2. TIMEOUT — don't let one stuck SKU block forever ────────────────────
async def run_with_timeout(sku_gap, semaphore, timeout_seconds=120):
    try:
        return await asyncio.wait_for(
            run_sku_pipeline(sku_gap, semaphore),
            timeout=timeout_seconds
        )
    except asyncio.TimeoutError:
        return {"sku": sku_gap.sku, "error": "Timed out", "status": "failed"}


# ── 3. RETRY — transient API failures ──────────────────────────────────────
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))
async def run_sku_pipeline_with_retry(sku_gap, semaphore):
    return await run_sku_pipeline(sku_gap, semaphore)


# ── 4. PROGRESS TRACKING ───────────────────────────────────────────────────
from tqdm.asyncio import tqdm

async def orchestrate_with_progress(sku_gaps, max_concurrent=5):
    semaphore = asyncio.Semaphore(max_concurrent)
    tasks = [run_sku_pipeline(sku, semaphore) for sku in sku_gaps]

    results = []
    for coro in tqdm.as_completed(tasks, total=len(tasks), desc="Processing SKUs"):
        result = await coro
        results.append(result)

    return results