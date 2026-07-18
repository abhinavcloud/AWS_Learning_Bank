# Ownership — ECDE Data Approval Story

> **Fill in the bracketed [ ] placeholders with real numbers before using this — an invented metric that falls apart under a follow-up will hurt you more than an honest, smaller one.**

## Situation
Telstra was undergoing a modernization program under its T30 roadmap, migrating legacy on-prem infrastructure to AWS. As part of this, we were migrating databases — and access to them — from legacy on-prem systems to the cloud. This was sensitive work because it changed data residency from on-prem to cloud, and the database held PII for Telstra's Consumer, Enterprise, SMB, and Government customers: names, emails, residential addresses, mobile numbers, race, ethnicity, and social security numbers, credit card numbers and payment methods.

## Task
While designing the migration solution, I discovered the data migration required sign-off from the ECDE (Enterprise and Customer Data Evaluation) team — and that each customer segment had different approvers with different concerns (Enterprise approvers cared about employee headcount, department, and contact data; Government approvers had a different, more sensitive set of requirements). This wasn't something the Product Owner or business had a clear path to get through, and it wasn't technically my job — my scope was the architecture and technical migration — but without these approvals, the dev team couldn't start build work at all.

## Action
This was flagged in a meeting where the Product Owner explained the business was expecting these approvals but didn't have a clear owner or plan to get them. I took ownership of it myself, even though it sat outside my defined technical scope, because I understood the design well enough to explain it credibly to each approver, and because without this the team would be blocked with no clear timeline.

I identified the distinct approvers across [12-15] customer segments — Consumer, Enterprise, SMB, and Government — and reached out to each independently rather than trying to run one combined session, since their concerns were different enough that a single generic pitch wouldn't hold up. For each, I scheduled a working session and walked through the specific data residency and access patterns relevant to their segment, with particular focus on encryption at rest and in transit and our secrets management approach, since that's what most of the pushback centered on.

[Describe the friction moment here — e.g., a specific approver pushed back on something, or you found a design gap mid-process that needed fixing before you could get sign-off. This is the part that shows real ownership under difficulty, not just "I had meetings and got yeses."]

Where I hit resistance or gaps, I [went back to the architecture / looped in the security team / adjusted the design] rather than escalating it as someone else's problem, since I'd taken this on as mine to close.

## Result
I got sign-off from every required approver across all [X] segments and documented the approvals, presenting them to the Domain leadership team. This let the dev team start build work at [X weeks] earlier than they otherwise would have, with confidence the data residency and access design was already validated. It also became a benchmark — [describe reuse: e.g., "two other migration workstreams later used my approval documentation as their template" or "the approach was adopted as the standard process for future ECDE reviews"] — giving future teams a clear roadmap of who to approach, in what order, and with what materials.

---

## One-Minute Soundbite
"During a sensitive on-prem-to-cloud data migration at Telstra involving PII across Consumer, Enterprise, SMB, and Government customers, I discovered the migration needed sign-off from [X] different approvers across the ECDE team — work that wasn't technically my job, but without it the dev team couldn't start building at all. I took ownership of it myself: identified each segment's specific approvers, ran individual sessions focused on their specific concerns — mainly encryption and secrets management — [and worked through a specific point of resistance/gap]. I got every sign-off, documented it for leadership, and the process became the template other migration workstreams used afterward."

## Ready Answer: "Why did you take this on if it wasn't your job?"
"My scope was the technical architecture and migration design, but I was the person who understood the design well enough to explain it credibly to each approver — and the Product Owner didn't have a clear plan to get these sign-offs. Without them, the whole workstream was blocked with no owner and no timeline. I didn't wait to be assigned it; closing that gap was more valuable to the program than staying strictly inside my lane, so I took it on."

## Ready Answers to Other Likely Follow-ups
- **How did you handle the different requirements across customer segments?** [Describe: did you build one base narrative and adapt it, or fully separate materials per segment? What made Government different from Enterprise, concretely?]
- **What was the hardest pushback you got?** [This needs a real answer — a specific approver, a specific objection, and specifically what you did about it. This is likely to be asked directly since your story doesn't currently have visible conflict.]
- **How long did the whole approval process take, start to finish?** [Fill in — days or weeks, and how that compares to what would have happened without you driving it.]
- **What did the "benchmark" documentation actually contain?** [e.g., approver contact list per segment, a standard slide deck / one-pager per segment, a sign-off tracking sheet — be concrete.]

---

## 90-Second Verbal Script

**(0:00-0:20)**
"I was working on a sensitive data migration at Telstra — moving PII for Consumer, Enterprise, SMB, and Government customers from on-prem to AWS, which meant a change in data residency. While designing the solution, I found out this needed sign-off from the ECDE team, and each customer segment had different approvers with different concerns."

**(0:20-0:45)**
"This wasn't technically my job — my scope was the technical architecture — but the Product Owner didn't have a clear plan to get these approvals, and without them the dev team couldn't start building. I took ownership of it myself: identified the approvers across each segment, and ran individual sessions with each one focused on their specific concerns, mainly encryption and secrets management."

**(0:45-1:05)**
"[Insert the friction moment: a specific pushback, objection, or gap you had to resolve before getting sign-off.]"

**(1:05-1:25)**
"I got sign-off from every approver across all segments, documented it, and presented it to Domain leadership. That let the dev team start building [X weeks] earlier than they would have otherwise."

**(1:25-1:30)**
"It became the template — [other teams/future ECDE reviews] used my documentation afterward. I didn't wait to be asked; I saw what was blocking the team and closed it."

---

| Metric | Value |
|---|---|
| Customer segments requiring separate approval | 4 (Consumer, Enterprise, SMB, Government) |
| Number of distinct approvers/stakeholders | [X] |
| Time to get full sign-off | [X days/weeks] |
| Time saved for dev team start | [X weeks] |
| PII data types involved | Name, email, address, mobile number, race, ethnicity, SSN |
| Later reuse of the approval process/docs | [X — how many teams/workstreams reused it] |