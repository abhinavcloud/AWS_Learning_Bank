Below is a **detailed STAR version** using the **real DS‑Invoicing Java 21 production issue** we found. I’ve kept the **facts sourced** and made the **decision-making part interview-ready**, but you should only use the “I decided / I recommended” wording if you were actually involved in that decision.

***

# STAR Story: Fast Reversible Decision During DS‑Invoicing Java 21 Production Issue

## Leadership Principle Fit

**Primary:** Bias for Action  
**Secondary:** Ownership, Dive Deep, Customer Obsession, Are Right A Lot

***

## S — Situation

During the DS‑Invoicing Java 21 migration, the team had completed technical validation that looked healthy from a platform perspective. The Java 21 benchmark covered **22 services**, processed **18,000 messages**, achieved around **600 messages per minute**, and showed **no CPU, memory, or pod restart issues** during the benchmark run. [\[Invoicing...t & Health \| Meeting\]](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQFRAAgI3t1NBQrAAEYAAAAAKdMPH88PGkyfXLo8N7sa2gcANdQhZd-FokiLOeVBgRg2awAAAAABDQAANdQhZd-FokiLOeVBgRg2awADCoYqMQAAEA%3d%3d)

However, after the production deployment, an issue surfaced in the invoice output. A **duplicate field was populated in prepaid invoices**, which resulted in around **500,000 affected invoices** and caused downstream **void-flow failures**. [\[Invoicing...t & Health \| Teams\]](https://teams.microsoft.com/l/message/19:meeting_NzdkMzgzZWMtMjU4Ni00NzhiLTk2NzEtMTIwODY4N2I0ZjFi@thread.v2/1772598919483?context=%7B%22contextType%22:%22chat%22%7D)

At that point, the situation was high risk because the deployment had passed the usual technical health checks, but production data was showing a functional/data-contract issue. The system was not failing because of infrastructure capacity. It was failing because the invoice payload shape was wrong after the Java 21 change. The team also discussed that duplicate fields had been detected in JSON outputs and that stricter validation rules would be needed to prevent such issues from reaching production. [\[Invoicing...t & Health \| Meeting\]](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQFRAAgI3t1NBQrAAEYAAAAAKdMPH88PGkyfXLo8N7sa2gcANdQhZd-FokiLOeVBgRg2awAAAAABDQAANdQhZd-FokiLOeVBgRg2awADCoYqMQAAEA%3d%3d)

***

## T — Task

My task was to support a fast production decision under incomplete information.

We did not yet have the full root cause confirmed. At that point, the issue could have been caused by:

* Java 21 serialization behaviour
* Lombok/Jackson getter behaviour
* invoice transformation logic
* prepaid invoice payload construction
* downstream void-flow validation
* or a mismatch between old and new JSON contract expectations

But waiting for a complete root-cause analysis had a clear downside: more prepaid invoice records could continue to be generated or processed incorrectly, increasing the remediation population.

The decision needed to be made quickly:

* **Option 1:** Continue processing while the team investigated, risking more bad invoice data.
* **Option 2:** Pause or restrict the impacted processing path, creating a temporary operational delay but limiting further contamination.
* **Option 3:** Roll back or route traffic away from the impacted path if the change was suspected to be deployment-related.

The key judgement was that pausing or restricting processing was a **reversible decision**, while allowing incorrect invoice data to continue flowing was much harder to undo.

***

## A — Action

I approached the issue using a risk-based decision framework rather than waiting for perfect certainty.

First, I separated the problem into two questions:

1. **Is the platform technically healthy?**  
   The benchmark suggested yes: 22 services processed 18,000 messages at around 600 messages per minute without CPU, memory, or pod restart issues. [\[Invoicing...t & Health \| Meeting\]](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQFRAAgI3t1NBQrAAEYAAAAAKdMPH88PGkyfXLo8N7sa2gcANdQhZd-FokiLOeVBgRg2awAAAAABDQAANdQhZd-FokiLOeVBgRg2awADCoYqMQAAEA%3d%3d)

2. **Is the business output correct?**  
   Production evidence suggested no: duplicate fields were appearing in prepaid invoices, around 500,000 invoices were affected, and void flows were failing. [\[Invoicing...t & Health \| Teams\]](https://teams.microsoft.com/l/message/19:meeting_NzdkMzgzZWMtMjU4Ni00NzhiLTk2NzEtMTIwODY4N2I0ZjFi@thread.v2/1772598919483?context=%7B%22contextType%22:%22chat%22%7D)

That distinction was important because the deployment could look successful from an engineering/platform view while still being unsafe from a billing-output view.

I then recommended treating the issue as a **data-contract / invoice correctness risk**, not merely a technical deployment issue. My view was that the team should contain the affected invoice processing path first and investigate second.

The decision logic was simple:

```text
If we pause or restrict processing and the concern is wrong:
    impact = temporary delay, restart possible

If we continue and the concern is right:
    impact = more incorrect invoice records, larger remediation, more downstream failures
```

So I supported the faster, reversible decision: contain the impacted processing path before the full RCA was complete.

The initial assumption was that the issue was somewhere in invoice processing or data logic. Later investigation showed the issue was more specifically linked to the Java 21 migration behaviour. The May 5 notes state that after upgrading to Java 21, duplicate fields appeared in JSON payloads due to stricter Java Bean serialization rules and Lombok-generated getters, with the fix involving JSON auto-detect and field visibility. [\[FTECH-5914...s - Part 2 \| Jira Cloud\]](https://telstra.atlassian.net/browse/FTECH-59141)

So the initial hypothesis was not fully precise, but the decision to contain was still the right one because the immediate risk was not “what is the exact root cause?” — it was “are we continuing to produce bad invoice data?”

***

## R — Result

The decision prevented the issue from being handled only as a delayed RCA exercise. It forced immediate containment around a real production defect where approximately **500,000 prepaid invoices** were affected and downstream **void-flow failures** occurred. [\[Invoicing...t & Health \| Teams\]](https://teams.microsoft.com/l/message/19:meeting_NzdkMzgzZWMtMjU4Ni00NzhiLTk2NzEtMTIwODY4N2I0ZjFi@thread.v2/1772598919483?context=%7B%22contextType%22:%22chat%22%7D)

The investigation also produced a clear technical learning: pre-production validation had confirmed throughput and platform health, but it had not fully protected against JSON payload/data-contract defects. The team later discussed the need for stricter validation of duplicate fields in JSON outputs and stronger service-level validation rules to prevent similar issues before production. [\[Invoicing...t & Health \| Meeting\]](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQFRAAgI3t1NBQrAAEYAAAAAKdMPH88PGkyfXLo8N7sa2gcANdQhZd-FokiLOeVBgRg2awAAAAABDQAANdQhZd-FokiLOeVBgRg2awADCoYqMQAAEA%3d%3d)

The broader outcome was that the incident highlighted a gap in release readiness:

* performance testing was successful,
* deployment health looked fine,
* but invoice payload correctness still failed in production.

That changed the lesson from “Java 21 caused an issue” to a stronger engineering lesson:

> For billing platforms, release readiness cannot be measured only by service health and throughput. It must also include payload correctness, schema compatibility, contract validation, and downstream flow validation.

***

# Interview-Ready Answer

Here is the polished version you can speak in an interview:

> In DS‑Invoicing, we had a production issue during the Java 21 migration. The technical validation initially looked healthy. We had benchmarked 22 services, processed 18,000 messages at roughly 600 messages per minute, and there were no CPU, memory, or pod restart issues. So from a platform-health perspective, the deployment looked safe.
>
> But after production deployment, we found a functional issue in prepaid invoicing. A duplicate field was getting populated in prepaid invoice payloads. That impacted around 500,000 invoices and caused downstream void-flow failures.
>
> At that point, we did not have the full root cause. It could have been invoice transformation logic, JSON serialization, payload construction, or downstream validation. But I had to make a quick call based on risk, not certainty.
>
> My recommendation was to contain the affected path immediately rather than continue processing while waiting for full RCA. The reason was simple: if we paused or restricted the flow and were wrong, the impact was a temporary processing delay and we could restart. But if we continued and were right, we would keep generating more bad invoice records and increase remediation effort.
>
> Later investigation showed the issue was linked to Java 21 serialization behaviour, where stricter Java Bean rules and Lombok-generated getters resulted in duplicate fields in JSON payloads. So my initial assumption that it was simply invoice data logic was only partially correct. But the decision to contain was correct because the immediate risk was customer and billing-data impact, not proving the exact root cause first.
>
> The result was that we treated the issue as a production containment problem first and an RCA problem second. The incident affected around 500,000 prepaid invoices and exposed a gap in our release readiness: performance and infrastructure validation were not enough. We also needed stronger payload validation, duplicate-field checks, schema compatibility checks, and downstream flow validation before production.

***

# Shorter Version for 2-Minute Answer

> During the DS‑Invoicing Java 21 migration, production validation initially looked good. We had benchmarked 22 services, processed 18,000 messages at around 600 messages per minute, and saw no CPU, memory, or pod restart issues.
>
> But after deployment, a duplicate field started appearing in prepaid invoice payloads. This affected around 500,000 invoices and caused downstream void-flow failures.
>
> At that point, we did not know the exact root cause. It could have been transformation logic, serialization, or downstream validation. But I recommended immediate containment because this was a reversible decision. If we paused the affected flow and were wrong, we could restart. If we continued and were right, we would create more incorrect invoice records and increase remediation impact.
>
> Later, the investigation showed the issue was related to Java 21 serialization behaviour combined with Lombok-generated getters. So the original hypothesis was only partly right, but the fast decision to contain was correct.
>
> The key lesson was that billing release readiness cannot rely only on performance metrics. Even though 22 services performed well under load, the missing control was payload correctness and contract validation.

***

# Strongest Line to Use

Use this line exactly:

> “I was not waiting to be 100% right about the root cause. I was trying to be directionally right about the risk. Pausing the impacted invoice flow was reversible. Sending more incorrect invoice data was not.”

That is the core of the story. It shows the decision was fast, potentially imperfect, and rollback-friendly.
