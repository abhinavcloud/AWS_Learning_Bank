# Earn Trust — Pricing Engine Incident (Revised)

## Situation
I owned the Pricing and Discount Engine for a greenfield CRM and Order Management migration for a large telco (10M subscribers). Under normal load the engine handled roughly 1M catalog/pricing lookups a day — from app opens, USSD/IVR self-care, retail POS terminals, and partner reseller APIs — averaging around 12 RPS. It needed to support predictable festival peaks. I chose a Lambda-based design for cost and simplicity, and it worked well in staging and on normal days.

## Task
Deliver a reliable, low-cost pricing service that met peak traffic SLAs for the upcoming Eid promotion. After launch, the service began failing under the Eid peak; my task became: own the failure, stop customer impact, fix the real gap, and restore stakeholder trust.

## Action

**1. Detect and quantify the problem quickly**
Eid traffic wasn't a gradual ramp — it was event-driven. A push notification announcing a flash discount went out, and a large share of subscribers opened the app within minutes. Each session triggered multiple pricing calls (several promo cards shown at once, plus browsing), so the notification produced a sharp, front-loaded spike rather than flat sustained load — peaking around 5k RPS for roughly the first 10-15 minutes after the notification, then tapering.

Observability showed 15% throttling and median cold-start spikes of 300-500ms during that window. This happened 3-4 times over the day, each time a new promo notification went out.

I called an immediate cross-functional huddle with Dev, SRE, the Dev Manager, Product Owner, and Scrum Master to align on mitigation.

**2. Mitigate customer impact immediately**
I owned the communication and informed external product stakeholders transparently within 30 minutes, explaining the root cause and mitigation plan. I initiated a rollback and routed traffic to the legacy pricing stack to stop revenue impact while we worked on a durable fix.

**3. Own the actual mistake and fix it**
The root cause wasn't that Lambda was the wrong platform — it was that I launched with Lambda's default on-demand scaling and never configured it for a peak we already knew was coming. I'd validated the design against average load and normal-day traffic, but I hadn't load-tested against the specific notification-driven burst pattern, and I hadn't set up:
- **Scheduled provisioned concurrency** — AWS Application Auto Scaling supports scheduling provisioned concurrency to scale up ahead of a known event and back down after. I hadn't configured this, so every burst hit cold starts from a cold pool.
- **Reserved concurrency** on the pricing function, to guarantee it wouldn't be starved by other functions sharing the account-level concurrency pool during the same event.
- **A caching layer** (ElastiCache) in front of the catalog/discount-rules data, so repeated lookups for the same promo SKUs during a burst didn't all hit the backend fresh.

This was a planning gap, not an architecture failure — I'd built for the traffic I measured, not the traffic pattern the business already knew was coming from marketing's own notification schedule. I worked with SRE and the Dev Manager to size the fix using our incident telemetry (throttle counts, RPS, latency percentiles) and prioritized it with Product and the Scrum Master.

I sized the fix from first principles rather than guessing: concurrency needed = peak RPS × average execution duration. With a cache in front of the catalog data, a warm lookup ran around 80ms, so 5,000 RPS × 0.08s ≈ 400 concurrent executions needed at peak. I added a ~25% buffer for variance and provisioned 500, with reserved concurrency set to 600 so the function couldn't be starved by other functions sharing the account's concurrency pool during the same event. I scheduled provisioned concurrency to scale from a baseline of ~50 (covering normal traffic) up to 500 about 10 minutes ahead of each known promo notification, held through the burst and its tail, then back down — 3-4 times a day, tied to marketing's actual send schedule.

**4. Execute and validate**
The team configured scheduled scaling of provisioned concurrency ahead of each known promo window, added reserved concurrency for the pricing function, and put ElastiCache in front of the hot catalog data — all within one sprint (2 weeks). We validated with a canary rollout against a replayed traffic pattern from the incident, then monitored throttles/latency/error rate as we shifted back from the legacy stack. I presented before/after metrics to stakeholders and led a blameless post-mortem.

## Result
- Throttling dropped to 0% during subsequent promo bursts of comparable size.
- 99.9% availability maintained through later festival peaks, including bursts we hadn't specifically load-tested for.
- Revenue impact from the incident was contained by the rollback; no repeat incidents in later promotional events.
- Stakeholder confidence was restored — product leadership acknowledged the quick remediation and ownership.
- I introduced a launch checklist requiring known-event traffic patterns (not just average RPS) to be explicitly load-tested and scheduled-scaled before any promo-linked service goes live — the team still uses it.

---

## One-Minute Soundbite
"I launched a Lambda-based pricing engine for a 10M-subscriber telco's Eid promotion, and it throttled under a predictable notification-driven traffic spike — 15% throttling, 300-500ms cold starts. The real mistake wasn't picking Lambda — it was launching on default on-demand scaling without configuring provisioned concurrency for a peak marketing had already scheduled. I owned it immediately: rolled back to protect revenue, then fixed the actual gap — scheduled provisioned concurrency ahead of known promo windows, reserved concurrency so the function wasn't starved, and a caching layer to cut backend load. Zero throttling in every subsequent peak. I turned it into a launch checklist so we never again ship a promo-linked service without load-testing the known traffic pattern."

## Ready Answer: "Why didn't you just scale Lambda concurrency reactively — why plan ahead?"
"Lambda's default scaling is reactive and still pays a cold-start cost on the way up — for a burst that ramps in minutes right after a push notification, that ramp-up lag is exactly when customers feel it. Since we already knew the promo schedule from marketing, this wasn't unpredictable traffic — it was a known event we hadn't planned capacity for. Scheduled provisioned concurrency lets you pre-warm ahead of a known window and scale back down after, so you get serverless's cost benefit on quiet hours without paying the cold-start tax during the one window you know is coming. That's the piece I missed at launch — I'd tested against average load, not the specific event-driven burst shape."

## Ready Answers to Other Likely Follow-ups
- **How much provisioned/reserved concurrency did you actually configure?** Concurrency needed = peak RPS × average execution duration. With caching in front of the catalog, a warm lookup ran ~80ms, so 5,000 RPS × 0.08s ≈ 400 concurrent executions at peak. I provisioned 500 (25% buffer) and set reserved concurrency to 600 so the function couldn't be starved by other functions sharing the account pool. This is the number most likely to get a direct follow-up — have it cold.
- **Why did the burst hit 5k RPS specifically?** A promo notification triggers a large share of subscribers to open the app within minutes; each session fires several pricing calls (multiple SKUs/promo cards shown at once, plus browsing), so the request count is amplified per session, not one request per person. Worth having this ready but it's a secondary point now — the interview center is the planning gap, not the traffic math.
- **Why not switch to EC2/containers instead?** I considered it, but the actual root cause was a scaling-configuration gap, not a platform mismatch — Lambda with scheduled provisioned concurrency and caching solved it without giving up serverless's cost and ops benefits for the other 23 hours of the day.
- **How did you communicate the bad news?** Led with customer impact, owned the mistake plainly (I hadn't planned for a peak we already knew about), gave a clear remediation and verification timeline.
- **What concrete change prevented recurrence?** A launch checklist requiring known-event traffic patterns to be explicitly load-tested and scheduled-scaled before go-live — not just average RPS sign-off.
- **How did you get the $200k/day estimate?** Be ready to state plainly whether this was a rough back-of-envelope calc or a finance-validated figure — don't present it as more precise than it is.

---

## 90-Second Verbal Script

**(0:00-0:15)**
"I owned the Pricing and Discount Engine for a greenfield CRM migration for a 10M-subscriber telco. I built it on Lambda for cost and simplicity, and it worked fine on normal days."

**(0:15-0:35)**
"For Eid, marketing sent promo notifications, and each one triggered a sharp spike — a large share of subscribers opening the app within minutes, each session firing several pricing calls. That pushed us to about 5,000 requests per second for 10-15 minutes at a time. We hit 15% throttling and 300-500ms cold starts. I owned it immediately: stakeholders informed within 30 minutes, rolled back to the legacy stack to stop revenue loss."

**(0:35-1:00)**
"The real mistake was mine: I'd launched on Lambda's default on-demand scaling and never configured it for a peak the business already knew was coming. I sized the fix from first principles — at 5,000 RPS with an 80-millisecond warm lookup, that's about 400 concurrent executions needed, so I provisioned 500 with a buffer, and set reserved concurrency to 600 so the function couldn't be starved by others. I scheduled that to scale up 10 minutes ahead of each known promo window and back down after, plus added a cache in front of the catalog data. The team shipped it in one sprint."

**(1:00-1:20)**
"Result: zero throttling in every subsequent peak, 99.9% availability held through later festivals. I presented before/after metrics and led a blameless post-mortem."

**(1:20-1:30)**
"I turned it into a launch checklist — known-event traffic patterns get explicitly load-tested and scheduled-scaled before go-live now. I owned a planning gap, fixed the real cause, and stayed transparent throughout — that's what rebuilt trust."

---

| Metric | Value |
|---|---|
| Subscribers | 10M |
| Normal-day catalog lookups | ~1M/day (~12 RPS avg) |
| Peak burst RPS | ~5,000 RPS |
| Burst trigger | Promo push notification |
| Burst duration / frequency | ~10-15 min spike, 3-4x per day |
| Observed throttling | 15% |
| Cold-start latency | 300-500ms |
| Rollback time to legacy | ~30 minutes |
| Remediation delivery | 2 weeks / 1 sprint |
| Warm execution duration (with cache) | ~80ms |
| Peak concurrency needed (RPS × duration) | ~400 |
| Provisioned concurrency (with buffer) | 500 |
| Reserved concurrency | 600 |
| Baseline provisioned concurrency (off-peak) | ~50 |
| Fix components | Scheduled provisioned concurrency, reserved concurrency, ElastiCache |
| Post-fix throttling | 0% |
| Post-fix availability | 99.9% |
| Estimated avoided revenue loss | ~$200k/day (peak) |