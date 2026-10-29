# Resilience patterns: circuit breaker, backoff and rate limiting

**Covers:** Circuit breaker, Backoff, Rate limiting

## Failures spread unless you stop them {#overview}
In a system made of many parts, something is always failing. The danger is not the failure itself but how it spreads. A slow service makes its callers slow, retries make the slow service slower, and a flood of requests can knock over a service that would have coped with a normal load. Three patterns limit the damage: stop calling a service that is failing, retry more politely, and cap the demand.

## Circuit breaker {#circuit-breaker}
A circuit breaker stops calls to a service after it has failed repeatedly, instead of letting failed calls pile up. The name comes from the electrical device that cuts the current when it is overloaded.

It has three states:
- **Closed:** normal. Calls go through, and failures are counted.
- **Open:** after too many failures in a row, calls are refused at once, with no waiting. This gives the failing service time to recover and spares the caller from long timeouts.
- **Half-open:** after a pause, a single trial call is allowed. If it succeeds, the breaker closes. If it fails, the breaker opens again.

The caller needs a plan for the open state: show a fallback such as cached data, a short message or a reduced feature.

## Backoff {#backoff}
When a call fails, retrying is natural. But how soon? With backoff, the delay between repeated attempts keeps growing, for example 1 second, then 2, then 4, then 8, doubling each time. If many clients retry immediately and constantly, they hammer a service that is already struggling and keep it down. Growing delays spread the load out and give it time to recover.

Good practice adds jitter, a small random amount to each delay, so that thousands of clients do not all retry at the same instant. It also sets a cap on the delay and a maximum number of attempts, after which the failure is reported.

## Rate limiting {#rate-limiting}
Rate limiting caps how many requests one client may make within a time window, to protect a service from being overloaded by any single source, whether a bug, an attacker or an overenthusiastic customer. When the limit is reached, the service replies with the status 429, "too many requests", and often says how long to wait.

Typical limits are something like 100 requests per minute per user. It also helps share capacity fairly. The clients must handle the 429 reply, which is where backoff comes in.

## A worked example {#example}
Here are the three patterns in short programs.

```python
def delays(base, attempts, cap):
    return [min(base * 2**i, cap) for i in range(attempts)]


print(delays(1, 6, 20))
# [1, 2, 4, 8, 16, 20]


class Breaker:
    def __init__(self, limit):
        self.failures = 0
        self.limit = limit

    def call(self, fn):
        if self.failures >= self.limit:
            return "open: call skipped"
        try:
            result = fn()
            self.failures = 0
            return result
        except Exception:
            self.failures += 1
            return "failed"


def broken():
    raise RuntimeError("down")


b = Breaker(3)
print([b.call(broken) for _ in range(5)])
# ['failed', 'failed', 'failed', 'open: call skipped', 'open: call skipped']


def allow(timestamps, now, limit, window):
    recent = [t for t in timestamps if now - t < window]
    if len(recent) < limit:
        recent.append(now)
        return True, recent
    return False, recent


log = []
results = []
for second in (0, 1, 2, 3, 12):
    ok, log = allow(log, second, 3, 10)
    results.append(ok)
print(results)
# [True, True, True, False, True]
```

Let's read it:

1. The delays double from 1 to 16 seconds and are then held at the cap of 20. Six attempts wait a total of 1 + 2 + 4 + 8 + 16 + 20 = 51 seconds.
2. The breaker lets three calls fail, then stops calling at all. The fourth and fifth calls cost nothing, instead of waiting for a timeout each.
3. The limiter allows 3 requests in any 10-second window. The fourth, at second 3, is refused. At second 12, the earlier requests have left the window, so the request is allowed again.

Now count the saving. Without the breaker, 5 calls with a 3-second timeout each would take 15 seconds. With it, 3 failed calls take 9 seconds, and the last two are instant, which saves 6 seconds for each batch and lets the failing service rest.

## A common mistake {#mistake}
A common slip is retrying immediately and without limit. If 1,000 clients each retry 5 times at once, the struggling service receives 5,000 extra requests, which keeps it down. Use backoff with jitter and a maximum.

A second slip is forgetting the fallback. An open breaker without a plan shows an error faster, and nothing more.

## Key points {#key-points}
- A circuit breaker stops calling a failing service, then tests it again after a pause.
- Backoff makes retries wait longer each time, and jitter spreads them out.
- Rate limiting caps requests per client and answers with 429 when the cap is reached.
- Combine them: limit demand, retry with backoff and break the circuit when failures persist.
