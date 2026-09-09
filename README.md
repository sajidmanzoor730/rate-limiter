

![Tests](https://github.com/sajidmanzoor730/rater-limiter/actions/workflows/test.yml/badge.svg)













# ratelimiter

Thread-safe rate limiting for Python, with three interchangeable
algorithms and **zero runtime dependencies** — no framework, no
Redis, just the standard library. Built as a personal portfolio
project to demonstrate core software engineering practices (tests,
CI, dependency injection for testability, documented tradeoffs), not
as a claim of production usage anywhere.

## Why three algorithms, not one

Each has a real, different tradeoff — implementing all three (instead
of picking one and hiding the others' downsides) is the point:

| Algorithm | Memory per key | Accuracy | Known weakness |
|---|---|---|---|
| `TokenBucketLimiter` | O(1) | Allows controlled bursts up to capacity | None significant — this is what most production systems (AWS, Stripe) use |
| `SlidingWindowLimiter` | O(requests in window) | Exact — no boundary effects | Memory grows with request volume in-window |
| `FixedWindowLimiter` | O(1) | Cheapest | Can allow ~2x the intended rate in a burst straddling a window boundary — proven by `test_documents_the_known_boundary_burst_flaw` in the test suite, not just asserted in this table |

## Installation

No PyPI package published — clone and use directly, or `pip install -e .`:

```bash
git clone <this-repo>
cd rate-limiter
pip install -e .
```

## Usage

```python
from ratelimiter import TokenBucketLimiter, rate_limited, RateLimitExceeded

limiter = TokenBucketLimiter(capacity=3, refill_rate=1)  # 3 burst, 1/sec sustained

if limiter.allow("user-42"):
    ...  # handle the request
else:
    ...  # return 429

# Or as a decorator, with a per-caller key:
api_limiter = TokenBucketLimiter(capacity=100, refill_rate=10)

@rate_limited(api_limiter, key_func=lambda user_id, *a, **kw: user_id)
def handle_request(user_id: str, payload: dict):
    ...

try:
    handle_request("alice", {})
except RateLimitExceeded as e:
    print(f"denied: {e}")  # e.retry_after available for sliding-window limiters
```

Run `examples/basic_usage.py` for working, executable output:

```bash
python3 examples/basic_usage.py
```

## Testing

20 tests, all passing — run them yourself:

```bash
cd tests
python -m unittest discover -v
```

Tests use an injected `FakeClock` (see `tests/helpers.py`) instead of
real `time.sleep()` calls, so refill/expiry/boundary behavior is
asserted exactly rather than approximately — no flaky timing-based
tests. The one exception is `test_thread_safety.py`, which
deliberately uses real threads and real time, because it's testing
actual concurrent execution, which a fake clock can't simulate.

CI (`.github/workflows/test.yml`) runs the full suite on Python
3.10–3.12 on every push.

## Design notes / honest limitations

- **In-process only.** State lives in memory in each limiter
  instance. This is fine for a single process, but a real
  multi-server production deployment would need a shared backend
  (Redis is the standard choice) so all servers see the same
  counters — that's a genuinely different, larger project and isn't
  implemented here.
- **Single lock per limiter, not per key.** Documented in
  `base.py` — correctness is guaranteed (see
  `test_thread_safety.py`), but all keys serialize through one lock.
  A sharded-lock or lock-free approach would scale further under very
  high concurrency; not needed to prove the concept here.
- No load testing has been done — the concurrency test proves
  correctness (no race condition lets the limit be exceeded), not
  throughput under real load.

## Project structure

```
src/ratelimiter/    - the library
tests/               - unittest-based test suite (pytest-compatible)
examples/            - runnable usage example
.github/workflows/   - CI
```
