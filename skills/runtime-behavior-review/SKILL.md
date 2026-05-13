---
name: runtime-behavior-review
description: Runtime behavior analysis layer for every infrastructure, observability, monitoring, middleware, rate-limiting, health-check, readiness-probe, metrics-collection, caching, queueing, or system-lifecycle PR. Catches what static code review misses — self-measurement contamination, cumulative-vs-windowed metric confusion, boundary math at small sample sizes, shutdown/drain race conditions, config flags that are semantic no-ops, middleware ordering that changes what gets measured, probe interference with business metrics, and metric exposure without auth. Composes with generator-reviewer-loop and triple-pass-review-loop as an additional mandatory checklist that runs AFTER Stage 2 (Reviewer) and BEFORE Stage 3 (Reviewer-of-Reviewer). Operates at the "2 AM prod is down" level — the question is never "does this compile" but "what happens to this code when 400 instances are draining, GC is spiking, the load balancer is probing every 5 seconds, and the on-call engineer is reading the dashboard this code powers."
trigger: Any PR touching middleware registration, rate limiting, health checks, readiness/liveness probes, metrics collection, monitoring utilities, event loop measurement, GC tracking, latency histograms, active request counters, graceful shutdown, circuit breakers, caching layers, queue consumers/producers, retry logic, timeout configuration, connection pooling, or any code whose correctness depends on runtime conditions rather than static logic.
enforcement: mandatory
---

# Runtime Behavior Review

Static code review catches syntax bugs, logic errors, missing guards, and style violations. It does not catch what happens when the code runs under real conditions — when the monitoring system measures itself, when a "disable" flag doesn't actually disable, when a cumulative metric reports lifetime averages as if they're current values, when a readiness probe returns 200 during shutdown because nobody checked the drain flag.

This skill exists because the gap between "code that is correct" and "code that behaves correctly at runtime" is where production incidents live. Every finding in this skill was learned from a real miss — code that passed a thorough static review and still caused problems in production.

This skill composes with `generator-reviewer-loop` and `triple-pass-review-loop`. It runs as a mandatory checklist after Stage 2 (Reviewer) and before Stage 3 (Reviewer-of-Reviewer). If any item on the checklist produces a finding, it gets the same severity treatment as any other Stage 2 finding.

---

## The Core Question

For every piece of infrastructure code, the reviewer must ask:

> "What happens to this code when 400 instances are draining, GC is spiking at 15%, the load balancer is probing every 5 seconds, event loop lag is at 300ms, the deployment is mid-rollout, and the on-call engineer is reading the dashboard this code powers at 2 AM?"

If the answer is "it works fine" — prove it by walking through each checklist item. If the answer is "I don't know" — that's a finding.

---

## The Seven Failure Domains

Infrastructure code fails in seven domains that static review systematically misses. Each domain has a checklist, a rationale, and examples of real failures.

---

### Domain 1 — Self-Measurement Contamination

**The failure:** The monitoring system measures itself. A readiness probe runs through the same middleware that tracks active requests and latency. The probe's own fast response time (sub-millisecond) floods the latency sample set. On low-traffic services, the probe dominates the metrics, masking slow business requests. The readiness score looks healthy because the probe keeps telling itself it's healthy.

**Checklist:**

- [ ] Does any health/readiness/liveness probe pass through monitoring middleware?
- [ ] If yes, is the probe registered *before* the monitoring middleware in the middleware stack, so it bypasses measurement?
- [ ] Are health-check paths excluded from latency recording?
- [ ] Are health-check paths excluded from active request counting?
- [ ] Does the probe's own response time contaminate the P95/P99 calculation?
- [ ] On a service with 1 req/min business traffic and 12 probes/min (K8s default), which traffic dominates the sample set?

**The fix pattern:** Register probe routes before monitoring middleware. Or filter probe paths inside the monitoring middleware. Never let the system that determines health include its own health checks in the health calculation.

```javascript
// WRONG — probe measured by the middleware it feeds
app.use(monitoringMiddleware);
app.get('/readiness', readinessHandler);

// RIGHT — probe bypasses measurement
app.get('/readiness', readinessHandler);
app.use(monitoringMiddleware);
```

---

### Domain 2 — Cumulative vs. Windowed Metrics

**The failure:** A metric reports a lifetime aggregate as if it's a current value. `histogram.mean` returns the mean event loop delay since the histogram was created, not the mean over the last 10 seconds. After a 5-second spike at startup, the mean stays elevated for hours — long after the system has recovered. The readiness score stays degraded. The on-call engineer sees a permanently yellow dashboard and ignores it. Then a real spike happens and nobody notices because the dashboard was already yellow.

**Checklist:**

- [ ] For every metric read (mean, p95, count, rate): is the value windowed or cumulative?
- [ ] If cumulative: is the accumulator reset after reading? If not, the metric drifts toward lifetime average and loses sensitivity to current conditions.
- [ ] If windowed: what is the window size? Is it appropriate for the probe interval?
- [ ] Does `histogram.mean` represent "right now" or "since process start"?
- [ ] Does a GC pressure percentage represent "last 60 seconds" or "since boot"?
- [ ] After a spike, how long does the metric take to recover to baseline? Is that acceptable?

**The fix pattern:** Reset cumulative accumulators after reading. Or use explicitly windowed data structures (ring buffers, sliding windows, EWMAs). Document the window size.

```javascript
// WRONG — cumulative, never recovers from spikes
getEventLoopLag() {
  const { mean } = this.histogram;
  return mean / 1e6;
}

// RIGHT — reset after read, each sample reflects the window since last read
getEventLoopLag() {
  const { mean } = this.histogram;
  this.histogram.reset();
  return Number.isNaN(mean) ? 0 : mean / 1e6;
}
```

---

### Domain 3 — Boundary Math at Small Sample Sizes

**The failure:** A P95 calculation uses `Math.floor(length * 0.95)`. With 20 samples, that's index 19 — the maximum value, not the 95th percentile. The readiness score becomes artificially pessimistic because the P95 is actually the P100. With 1 sample, `Math.floor(1 * 0.95)` = 0, which accidentally works. With 2 samples, `Math.floor(2 * 0.95)` = 1, which returns the max. The math only produces correct percentiles at scale — exactly where you need it least, because at scale the difference between P95 and P100 is small.

**Checklist:**

- [ ] For every percentile calculation: what does the math produce at n=1, n=2, n=5, n=10, n=20, n=100?
- [ ] Does `Math.floor(n * 0.95)` ever equal `n - 1` (returning the max instead of P95)?
- [ ] Is the index clamped to `[0, length - 1]`?
- [ ] Does the formula match a standard implementation (e.g., nearest-rank method)?
- [ ] What happens when the sample buffer is empty? Does it return 0, null, or throw?
- [ ] What is the sample buffer size? Is it large enough for the percentile to be meaningful?

**The fix pattern:**

```javascript
// WRONG — returns max for small sample sizes
const index = Math.floor(sorted.length * 0.95);
return sorted[index]; // out of bounds when index === length

// RIGHT — nearest-rank, clamped
const index = Math.min(sorted.length - 1, Math.ceil(sorted.length * 0.95) - 1);
return sorted[index];
```

**Bonus — array access syntax:**

```javascript
// WRONG — accidental array coercion
return sorted[[index]]; // creates [index], coerced to string via .toString()

// RIGHT
return sorted[index];
```

`[[index]]` is a latent bug that works accidentally for integers via `.toString()` coercion but will break silently for `NaN`, `undefined`, or non-integer values.

---

### Domain 4 — Shutdown and Drain State

**The failure:** `gracefulShutdown()` sets `gConfig.shutdownInProgress = true` and starts draining connections. But the readiness endpoint never checks that flag. During SIGTERM, the probe continues returning `200` for healthy metrics on existing keep-alive connections. The load balancer sees a healthy instance. It keeps routing traffic. The instance is half-dead — some connections are draining, some are still accepting. New requests hit a process that's 10 seconds from exit. They timeout. The on-call engineer sees a spike of 504s but the readiness dashboard shows green.

**Checklist:**

- [ ] Does the readiness/health endpoint check `shutdownInProgress` (or equivalent)?
- [ ] Does it return `503` immediately when shutdown is in progress?
- [ ] Is `shutdownInProgress` set *before* or *after* the monitoring system is destroyed? (Must be before — probe must see the flag before the metrics system is torn down.)
- [ ] During graceful shutdown, are new connections refused or just new requests on existing connections?
- [ ] What happens if a readiness check arrives between `shutdownInProgress = true` and `server.close()`?
- [ ] Is there a drain period where the probe returns `503` but existing requests are still served?
- [ ] Does the monitoring middleware itself check `shutdownInProgress` and skip recording for drain-phase requests?

**The fix pattern:**

```javascript
const readinessCheckService = async () => {
  // First check — before any metric calculation
  if (globalThis.gConfig?.shutdownInProgress) {
    return {
      is_ready: false,
      availability: 'not_available',
      load_percent: 100,
      checks: { shutdownInProgress: true },
      timestamp: new Date().toISOString(),
    };
  }
  // ... normal health checks
};
```

---

### Domain 5 — Config Flags That Are Semantic No-Ops

**The failure:** A config flag `is_api_rate_limit` is supposed to disable rate limiting when set to `false`. The implementation:

```javascript
if (globalThis.gConfig.is_api_rate_limit === false) return DEFAULT_RATE_LIMIT;
```

When the flag is `false`, this returns `DEFAULT_RATE_LIMIT` (500 requests/window) — rate limiting is *still active*. The flag doesn't disable anything. It just changes the limit from "per-path configured" to "default 500." The ops team thinks they disabled rate limiting during a traffic spike. They didn't. Users are still getting 429s.

**Checklist:**

- [ ] For every config flag: does the "off" path actually disable the feature?
- [ ] Does `is_X_enabled === false` result in the feature being completely bypassed, or just running with defaults?
- [ ] Is there a `skip` option (in rate limiters, middleware, etc.) that actually prevents execution?
- [ ] What is the observable difference between flag=true and flag=false? Can you verify it?
- [ ] Is the flag name accurate? Does `is_api_rate_limit` mean "rate limiting is active" or "rate limiting should be applied"?
- [ ] What happens when the flag is undefined/null? Does `=== false` silently fall through to the enabled path?

**The fix pattern:**

```javascript
// WRONG — "disabled" still limits at 500
max: async (request) => {
  if (globalThis.gConfig.is_api_rate_limit === false) return DEFAULT_RATE_LIMIT;
  // ...
}

// RIGHT — skip option actually bypasses the limiter
const rateLimiter = rateLimit({
  skip: () => globalThis.gConfig?.is_api_rate_limit === false,
  max: async (request) => { /* per-path logic */ },
});
```

---

### Domain 6 — Middleware Ordering and Measurement Scope

**The failure:** Monitoring middleware is registered *after* body parsers. Request latency is measured from "body parsed" to "response sent." But body parsing — especially for large JSON payloads, multipart uploads, or XML bodies — can take 50–500ms. That time is excluded from the latency metric. The readiness score says the system is fast. The user experiences slow responses. The metrics are correct but incomplete, which is worse than being wrong — it creates false confidence.

**Checklist:**

- [ ] Where in the middleware stack is the monitoring middleware registered?
- [ ] What work happens *before* timing starts? (Body parsing, cookie parsing, CORS, auth?)
- [ ] What work happens *after* timing ends? (Response compression, logging, error handlers?)
- [ ] Is the full request lifecycle measured, or only a subset?
- [ ] If middleware order changes in a future refactor, does the measurement scope change silently?
- [ ] For active request counting: does the count include requests still being parsed, or only requests that have entered the handler?

**The fix pattern:**

```javascript
// WRONG — excludes body parsing from latency
app.use(bodyParser.json(limitData));
app.use(bodyParser.urlencoded({ extended: true }));
app.use(monitoringMiddleware); // timing starts here — body already parsed

// RIGHT — captures full request lifecycle
app.use(monitoringMiddleware); // timing starts here — includes parsing
app.use(bodyParser.json(limitData));
app.use(bodyParser.urlencoded({ extended: true }));
```

**Note:** This requires the monitoring middleware to handle raw `req.body` (which may be undefined before parsing). The middleware should only read `req.method`, `req.path`, `res.statusCode` — none of which depend on body parsing.

---

### Domain 7 — Metric Exposure and Operational Security

**The failure:** The `/readiness` endpoint returns CPU percentage, heap usage, GC pressure, active request count, event loop lag, and P95 latency — all without authentication. An attacker probing the endpoint learns the system's current load profile. They can time a DDoS for when GC pressure is already high. They can identify the optimal payload size by watching latency respond to their requests. The rate limiter at 4 req/s provides some throttling but doesn't prevent information gathering.

**Checklist:**

- [ ] What information does the probe endpoint expose?
- [ ] Is any of it useful for timing attacks, DDoS optimization, or resource exhaustion attacks?
- [ ] Is the endpoint restricted to internal networks (K8s cluster-internal, VPC-only)?
- [ ] If internet-facing: does the response include the full `checks` object, or just `is_ready` and `availability`?
- [ ] Is the rate limit sufficient to prevent metric scraping?
- [ ] Can an attacker correlate probe responses with their own requests to fingerprint the system's capacity?

**The fix pattern:** For K8s-only probes, restrict to cluster-internal IP ranges. For internet-facing services, return only `is_ready` and strip the `checks` detail:

```javascript
app.get('/readiness', rateLimiter, async (req, res) => {
  const result = await readinessCheckService();
  const isInternal = req.ip?.startsWith('10.') || req.ip?.startsWith('172.');
  const statusCode = result.is_ready ? 200 : 503;
  res.status(statusCode).send(isInternal ? result : { is_ready: result.is_ready });
});
```

---

## Compound Failures — Where Domains Intersect

The most dangerous failures combine multiple domains:

**Self-measurement + Cumulative metrics:** The readiness probe measures itself (Domain 1) AND the metric it contaminates is cumulative (Domain 2). The probe's sub-millisecond latency drags the lifetime mean down, making the system appear healthier than it is. A real latency spike is averaged away by thousands of fast probe responses.

**Shutdown + Self-measurement:** During drain (Domain 4), the probe keeps running (Domain 1) and keeps recording itself as a fast, successful request. The metrics show a healthy, low-traffic system — because the only traffic is the probe itself.

**Config no-op + Middleware ordering:** The ops team "disables" rate limiting (Domain 5) during a traffic surge. It doesn't actually disable. Meanwhile, monitoring middleware is after body parsers (Domain 6), so the dashboard shows low latency even as requests queue in the parser. The ops team thinks their intervention worked. It didn't, and the dashboard confirms their false belief.

---

## Integration with Review Pipeline

This checklist runs as part of the existing review loop:

1. **Stage 0** — Pre-flight (console.log grep, import/export check)
2. **Stage 1** — Generator pass (architecture + correctness)
3. **Stage 2** — Reviewer pass (adversarial)
4. **Stage 2.5 — Runtime Behavior Review** ← THIS SKILL
5. **Stage 3** — Reviewer-of-Reviewer (severity rationalization audit)
6. **Stage 4** — Codex (aesthetic + maintainability)

Stage 2.5 only triggers for infrastructure PRs (see trigger criteria in frontmatter). For pure UI/component/business-logic PRs, it does not apply.

### Output Format

When Stage 2.5 produces findings, they follow the same format as Stage 2:

```
### Stage 2.5 — Runtime Behavior Review

**Domain 1 (Self-Measurement):** [finding or ✅ clear]
**Domain 2 (Cumulative vs Windowed):** [finding or ✅ clear]
**Domain 3 (Boundary Math):** [finding or ✅ clear]
**Domain 4 (Shutdown/Drain):** [finding or ✅ clear]
**Domain 5 (Config No-Ops):** [finding or ✅ clear]
**Domain 6 (Middleware Ordering):** [finding or ✅ clear]
**Domain 7 (Metric Exposure):** [finding or ✅ clear]
```

Each finding gets a severity (Critical/High/Medium/Low) and feeds into the same confidence gate as other Stage 2 findings.

---

## When NOT to Apply This Skill

This skill is infrastructure-scoped. Do NOT apply it to:

- Pure UI component PRs (React components, CSS, styling)
- Business logic PRs (form validation, workflow rules, data transformation)
- Translation/i18n PRs
- Pure refactors that don't touch middleware, metrics, or system lifecycle

The trigger is **what the code does at runtime**, not what language it's written in. A Node.js utility function that formats dates does not trigger this skill. A Node.js middleware that counts active requests does.
