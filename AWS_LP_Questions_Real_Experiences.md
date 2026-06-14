# AWS Solution Architect Interview — Leadership Principle STAR Answer Bank

---

## 1. Customer Obsession

### Vague Interview Question
Tell me about a time you designed a cloud solution by working backwards from customer impact rather than just choosing the most advanced AWS service.

### Leadership Principle Being Probed
**Customer Obsession**

### STAR Answer
**Situation:** We are currently undergoing a massive migration and transformation project at Telstra which involves migrating close to 5M Enerprise and Government customers from Legacy Biller called FlexCab and MICA to our Invoicing Cloud Stack which currently serves Individual Customers only. The migration not only involved migrating the backend data and logic but also creating entirely new modern invoicing templates for Enterprise and Government customers.

**Task:** The task was massive with one clear goal in mind which was when the Enterprise and Government customers are handed out their new bills there must not be a massive uptick in customer complaints or enquiries due to errors in bills or introducing new terms and concepts which they are not familiar or something that comes up as a surprise or shock to them. 

**Action:** The first point of action was to understanding and comparing our invoice templates and the Flexcab Invoicing templates and the business and mapping rules. We made sure that every paramter was accounted for and no information will be lost out when the bills are generated on the new platform. If any new parameters need to be mapped out we set out a parallel team to work on these implementions on the cloud platform.

For transformation we evaluated AWS Glue but rejected it because it doesn't natively handle EBCDIC (Extended Binary Coded Decimal Interchange Code) over customer built EC2 instances which would split these files into smaller chunks and parse them to be processed further.

Also, when the final bills were out we ran a reconcilation AWS batch job before sending out any bills to customer to compare the legacy bills with the new modern bills before decommisionin the legacy biller.

**Result:** This approach led us to optimise cost and also simultaenously enhance customer delight with their new modern format bills with less that 1% tickets or enquiries opened due to billing errors and less than 5% enquiries on understanding the new bill.

---

## 2. Ownership

### Vague Interview Question
Tell me about a time you identified a design issue that was technically working but still not right for the business.

### Leadership Principle Being Probed
**Ownership**

### STAR Answer
**Situation:** Telstra has a marketplace from where its sells various different products like devices, iphones and appliances to its customer base. The marketplace activities generally have less traffic throught the year except on some events and festivals like Black Friday, IPhone Launch or Chrismas and New Year where both Telstra employees and customer check them out for discounts and promotions. During an architecure review I noticed that the backend database is incurring huge costs even when its sitting idle for most of the time of the year. I did a deep dive and ran thorugh the cost and usage reports and went throught the cost exmplorer to pin down the issue which was the database was a provisioned RDS which was highly avaialble acorss multi az with standby and read replicas which was burning money.

**Task:** The task was to create a consistent invoicing database that could store customer invoice details and could support huge bursts of traffic, remain highly availble during hug load and still providing with ACID transactions however do not incur costs during most of the downtime during the year.

**Action:** While the architecture was correct and it supported the Reliabilty and Performance Efficiency Pillar of the Well Architected framework, the costs was runnig high. We migrated the RDS DB from a Provisoned RDS to an Aurora Serverless V2 Database which could scale to 0 and auto pause when there are no request coming for a specified duration.

**Result:** The migration from a provisoned highle available database to a managed highly available serverless database proved a huge win with the cost of db for this particular part of the infrastructure almost dropping by 75%  which resulted in massive cost savings for the domain and the organization.


---

## 3. Invent and Simplify

### Vague Interview Question
Describe a time when you simplified an architecture without losing the core capability.

### Leadership Principle Being Probed
**Invent and Simplify**

### STAR Answer
**Situation:** We are currently undergoing a massive migration and transformation project at Telstra which involves migrating close to 5M Enerprise and Government customers from Legacy Biller called FlexCab and MICA to our Invoicing Cloud Stack which currently serves Individual Customers only. The migration not only involved migrating the backend data and logic but also creating entirely new modern invoicing templates for Enterprise and Government customers.

**Task:** The task was massive with one clear goal in mind which was the the handling of legacy data files which were in EBCDIC (Extended Binary Coded Decimal Interchange Code) and converting them into simple json files which our cloud platform could understand and no informaion is lost during this transformation. Also the files were running into GBs so we needed a mechanism to break and split them before trnsfroming the,.

**Action:** Several options was considered. One of the highly thought about option was to use AWS Glue to understand and transform these binary encoded files to actual json files which can be used as input for generating the html/pdf bills. However, using glue would have required extra handling because Glue does not natively parse EBCIDIC encoded data. Also the size of these files would be in GBs and would require long running GLUE jobs which was not cost optimized. In the end we decided on a strategy of transfering these huge files into S3 bucket using multi-part upload. to be picked up by a custom service running on EC2 instances. The service would read and understand the data and split the huge file into smaller chunks. 

**Result:** This approach led us to creating our custom built service which could handle and trasnform binary formatted data over the use of AWS Glue service whicch would have required custom code and was not suitable for long runnig workloads and also helped in cost savings.

---

## 4. Are Right, A Lot

### Vague Interview Question
Tell me about a time you had to make a service selection decision where there was no universally correct answer.

### Leadership Principle Being Probed
**Are Right, A Lot**

### STAR Answer
**Situation:** While designing AWS workloads for DS-Invocing, I am frequently faced with choices for event driven architecure. Whether I go towards a a workflow orchestrator or event choreography pattern is the obvious debate. In Telstra we have the Marketplace where we sell products. Each successfully sold products generates a payment receipt, however if payment is sucessfull and any other step is failed we need to void the payment receipt and generate a refund receipt.

**Task:** My task was to make the right architectural choice based on functional requirements for a workflow orchestrator vs event choreography.

**Action:** I evaluated the functional requirement and understood when the payment was completed we created a payment record and stored in our database and generated a payment receipt. However if a subsequent step like shipping failed we must generate a refund receipt. We understood that for this we would need the payment data against that receipt number. However, shipping service didnt have the payment id record so it could not have triggered the refund service. In this case instead of choreography where shipping service would need to fetch the payment record somehow we created a the workflow orcehstrator as AWS Step Function so that the shipping service would just send faliure to orchestator and then it the orchestrator would fetch the payment details and call the refund service with the data. It would also call the payment service generate a void payment receipt over the original paid receitp.

**Result:** This approach helped us to better control the flow of steps and have a single point of orchestration whiose logic can be built, understood, managed and montiored easily over the complex compemsatory steps of choregraphy which increases complecity and monitoring issues in faliure paths.

---

## 5. Learn and Be Curious

### Vague Interview Question
Tell me about a time you learned a new cloud pattern and applied it to solve a business problem.

### Leadership Principle Being Probed
**Learn and Be Curious**

### STAR Answer
**Situation:** In Telstra, we are starting with our GenAI intiatives roadmaps. For this all teams were asked to submit GenAI ideas and Proof of Concepts which can be implemented at organization level.

**Task:** The objective was to design a practical AI-assisted ticket creation and resolution flow using AWS services without sending unnecessary context or increasing token cost.

**Action:** I designed a pattern where enterprise data is converted into embeddings using Amazon Bedrock embedding models and stored in a vector database such as OpenSearch, Kendra, or S3 Vector storage. This becomes the RAG knowledge base. On top of that, AI agents can use Bedrock foundation models and connect through an MCP-style tool layer to systems such as Jira, Salesforce, or Confluence. I also introduced request transformation through API Gateway so only essential context is passed to the model and downstream tools. For decoupling, incoming complaints can be placed on SQS, and the agent can process them asynchronously using the RAG knowledge base and appropriate tools.

**Result:** The design showed how GenAI can be made enterprise-ready by combining retrieval, tool execution, asynchronous processing, and cost control. The learning was that useful AI architecture is not just about calling an LLM; it requires grounding, integration, context reduction, and operational controls.

---

## 6. Hire and Develop the Best

### Vague Interview Question
Tell me about a time you helped others understand a complex architecture or raised the technical bar for the team.

### Leadership Principle Being Probed
**Hire and Develop the Best**

### STAR Answer
**Situation:** In cloud architecture discussions, teams often understand individual AWS services but struggle with how those services interact under real workload conditions, especially around scaling, failure, and cost.

**Task:** My role was to explain complex topics such as Lambda concurrency, EKS scaling, Well-Architected trade-offs, and disaster recovery in a way that helped others make better decisions independently.

**Action:** I broke the concepts into operational layers. For Lambda, I explained that one request maps to one invocation and concurrency must be managed using reserved concurrency and provisioned concurrency. For EKS, I separated pod-level scaling using HPA from node-level scaling using Cluster Autoscaler or Karpenter. For Well-Architected reviews, I explained that reliability, performance, security, and cost are trade-offs, not isolated checkboxes. I used concrete examples such as RDS Proxy preventing Aurora Serverless auto-pause and ElastiCache Serverless introducing VPC endpoint costs.

**Result:** This approach improved team understanding because people could connect AWS theory to real design consequences. It also raised the bar for architecture discussions by shifting them from “which service should we use?” to “what behaviour will this design create under load, failure, and cost constraints?”

---

## 7. Insist on the Highest Standards

### Vague Interview Question
Tell me about a time when you did not accept an architecture just because it was functional.

### Leadership Principle Being Probed
**Insist on the Highest Standards**

### STAR Answer
**Situation:** During one of the initiates in Telstra, I was closely involved with the platform team to review and approve ther implementation of the architecture. In one of such reviews, I flagged several instances of broad IAM policies, and security groups. Two major issues I noticed were that for all the lambda services they used one broad role which had multiple policies for databases, caches, queues and buckets which were not required by each and every lambda. The other issue I noticed was that while the security groups were having ingress from the upstream resource security group they also had another ingress rule which allowed all IPs. This effecitively nullified the use of security groups as ingress.

**Task:** The task was to dive deep into each policy, roles and security group implementation and do a overall review of the terraform IaC code

**Action:** I went over the whole security architecture and created multiple tickets flagging each of the security gaps across multiple resources and assigned them to the platform team to fix. I worked closely with the platform team and regularly conducted architecture review sessions and clarified any issues or doubts while working parallely on the documentation extensively.

**Result:** The result was that once this initiative was closed, we were able to successfully reduce our blast radius by 80% and was able to get first pass approval from the security team for production. The extensive documentation on this intitative was marked as a baseline for all future security implementations.

---

## 8. Think Big

### Vague Interview Question
Tell me about a time you designed for future scale instead of only solving the immediate requirement.

### Leadership Principle Being Probed
**Think Big**

### STAR Answer
**Situation:** For a marketplace or ticketing workload, the immediate requirement may start with a few APIs, but the future system can include search, browse, cart, order placement, payment, inventory, shipping, and notifications.

**Task:** I needed to think beyond a single service and design an architecture that could scale into a broader ecosystem of independently deployable services.

**Action:** I designed the system as microservices with clear boundaries and independent persistence. Search could use API Gateway, Lambda, and OpenSearch. Browse could use ECS Fargate, ElastiCache, and Aurora or RDS. Cart could use EKS, ElastiCache, and DynamoDB. Order placement could trigger asynchronous workflows using EventBridge, SNS/SQS, Lambda, ECS, EC2, Step Functions, and downstream databases. I considered both event choreography and workflow orchestration depending on whether the priority was flexibility, monitoring simplicity, or transaction control.

**Result:** The design allowed the application to grow from a few APIs into a larger distributed platform. It avoided creating a single tightly coupled system and instead created room for independent scaling, independent deployment, and technology choices based on each microservice’s requirement.

---

## 9. Bias for Action

### Vague Interview Question
Tell me about a time you had to prepare an application for a known high-traffic event.

### Leadership Principle Being Probed
**Bias for Action**

### STAR Answer
**Situation:** For big traffic days, waiting for the system to fail before scaling is risky. The architecture must be prepared before the event starts.

**Task:** My task was to identify proactive actions that could increase readiness across frontend, application, backend, and database layers.

**Action:** At the frontend and infrastructure layer, I used CDN caching, considered ALB pre-warming through AWS support, configured scheduled scaling, maintained more than one EC2 instance, and used warm pools for faster burst handling. At the application layer, I introduced queues to absorb sudden traffic. At the backend layer, I used event-driven architecture with SQS, SNS plus SQS, or EventBridge and scaled consumers based on queue depth. At the database layer, I used caching, read replicas, Multi-AZ clusters, RDS Proxy where appropriate, intelligent partitioning, and search offloading through OpenSearch.

**Result:** These actions reduced the risk of reactive firefighting during peak load. The system had mechanisms to absorb traffic, scale predictably, protect downstream services, and continue serving customers during demand spikes.

---

## 10. Frugality

### Vague Interview Question
Tell me about a time when you reduced AWS cost without compromising the core business requirement.

### Leadership Principle Being Probed
**Frugality**

### STAR Answer
**Situation:** In one design, ElastiCache Serverless was used for cache access from Lambda functions inside private VPCs. It gave a managed experience but created multiple VPC interface endpoints across Availability Zones, which increased the network cost even when cache traffic was low.

**Task:** The goal was to reduce AWS cost while preserving the application’s low-latency cache requirement.

**Action:** I reviewed the full cost behaviour instead of only the cache service pricing. The issue was the hidden networking cost from endpoint creation. I redesigned the cache to use provisioned ElastiCache inside the VPC so Lambda could connect to the cache without requiring the same endpoint-heavy model. This meant accepting more operational responsibility, but it aligned better with the actual workload and cost objective.

**Result:** The architecture became more cost-efficient while preserving the core cache capability. The lesson was that AWS cost optimization requires looking at the complete architecture — compute, data, network, managed-service side effects, and traffic patterns — not only the headline price of a service.

---

## 11. Earn Trust

### Vague Interview Question
Tell me about a time you had to explain an uncomfortable technical trade-off clearly to stakeholders.

### Leadership Principle Being Probed
**Earn Trust**

### STAR Answer
**Situation:** A design using Aurora Serverless with RDS Proxy was originally selected to support inconsistent traffic and connection pooling. The expectation was that the database would scale down during idle periods.

**Task:** I had to communicate that the design did not fully deliver the expected cost behaviour because RDS Proxy kept connections open and reduced the practical benefit of Aurora Serverless auto-pause.

**Action:** I explained the trade-off transparently. RDS Proxy helped with connection management and protected the database during high concurrency, but it also changed the idle-state cost behaviour. I framed the issue through Well-Architected pillars: reliability and performance efficiency improved, but cost optimization was negatively impacted. I then proposed an alternative using a provisioned Aurora cluster with read replicas where the cost and behaviour were more predictable.

**Result:** This built trust because the decision was not hidden or defended blindly. Stakeholders could see the technical reason, business impact, and available options. The conversation moved from blame to informed trade-off management.

---

## 12. Dive Deep

### Vague Interview Question
Tell me about a time you went below the surface of an AWS service and found the real scaling or cost behaviour.

### Leadership Principle Being Probed
**Dive Deep**

### STAR Answer
**Situation:** Lambda and EKS are both scalable compute options, but their scaling models are very different. A shallow answer like “both auto-scale” is not enough for architecture decisions.

**Task:** I needed to understand and explain the actual scaling mechanics so the right service could be selected and configured correctly.

**Action:** For Lambda, I looked at invocation behaviour, concurrency, reserved concurrency, throttling, and provisioned concurrency. Lambda scales by creating more concurrent executions, but each request maps to a Lambda invocation, and concurrency limits matter. For EKS, I separated pod-level and node-level scaling. HPA scales pods based on CPU or memory metrics. When nodes no longer have capacity and pods remain pending, Cluster Autoscaler or Karpenter provisions additional worker nodes through Auto Scaling Groups or EC2 Fleet-style capacity. I also considered pod resource requests and limits because they directly affect scheduling.

**Result:** The deep understanding helped avoid wrong assumptions. Lambda scaling is concurrency-driven and quota-aware, while EKS scaling involves both Kubernetes scheduling and underlying EC2 capacity. That depth is critical when designing for high traffic and predictable behaviour.

---

## 13. Have Backbone; Disagree and Commit

### Vague Interview Question
Tell me about a time you challenged a design decision because the trade-off was not acceptable.

### Leadership Principle Being Probed
**Have Backbone; Disagree and Commit**

### STAR Answer
**Situation:** A managed service may look like the obvious choice because it reduces operational overhead. For example, ElastiCache Serverless can be attractive because it is highly managed, but in a VPC-based Lambda architecture it can introduce additional VPC endpoint costs.

**Task:** I had to challenge the assumption that the most managed option was automatically the best architecture choice.

**Action:** I disagreed with the default direction and explained the trade-off clearly. The serverless cache model improved operational simplicity and availability, but the hidden network cost did not fit the workload’s cost target. I recommended provisioned ElastiCache inside the VPC. I also acknowledged the downside: more operational ownership and potential code/configuration changes around endpoints. Once the decision was made, I committed to the selected path and aligned the implementation with that trade-off.

**Result:** The architecture became more cost-aware without ignoring reliability or operational implications. The important part was to challenge the design respectfully with evidence, then commit fully once the final decision was taken.

---

## 14. Deliver Results

### Vague Interview Question
Tell me about a time you turned architecture principles into a concrete, deployable design.

### Leadership Principle Being Probed
**Deliver Results**

### STAR Answer
**Situation:** Designing cloud architecture is not useful if it remains at a diagram level. The system must translate into services, scaling mechanisms, security controls, and operational behaviour.

**Task:** My task was to produce an AWS architecture that could support microservices, scalability, security, high availability, and cost-aware operation.

**Action:** I mapped requirements to concrete AWS services. For compute, I used Lambda, ECS, EKS, or EC2 depending on workload type. For scaling, I used Lambda concurrency, HPA, Cluster Autoscaler or Karpenter, Auto Scaling Groups, scheduled scaling, queues, and backend scaling based on queue depth. For data, I used Aurora/RDS, DynamoDB, read replicas, caching, and OpenSearch. For security, I layered WAF, Shield, API Gateway throttling, NACLs, security groups, IAM, KMS, Secrets Manager, CloudTrail, GuardDuty, Inspector, and Access Analyzer. For DR, I evaluated backup and restore, pilot light, warm standby, and active-active strategies based on RTO and RPO.

**Result:** The output was not just a theoretical recommendation. It was a practical AWS design approach that connected service selection, scaling, security, reliability, and cost trade-offs into an implementable solution.

---

## 15. Strive to be Earth’s Best Employer

### Vague Interview Question
Tell me about a time you made architecture easier for engineers to operate, support, or reason about.

### Leadership Principle Being Probed
**Strive to be Earth’s Best Employer**

### STAR Answer
**Situation:** Distributed AWS systems can become difficult for teams to operate when service boundaries, scaling paths, and failure modes are unclear.

**Task:** My responsibility was to design in a way that not only met business requirements but also reduced unnecessary operational burden for engineering teams.

**Action:** I used microservice boundaries so each service could be independently deployed, maintained, invoked, and scaled. I avoided forcing one compute model everywhere. Long-running processes could use EC2 or ECS, short-running workloads could use Lambda, container orchestration could use EKS, relational data could use Aurora/RDS, fast key-value access could use DynamoDB, and low-latency reads could use ElastiCache. I also used queues and Step Functions where they simplified operational visibility and failure handling.

**Result:** The architecture became easier for engineers to reason about because each service had a clear responsibility, persistence model, scaling model, and failure boundary. That improves team effectiveness and reduces operational stress during incidents or releases.

---

## 16. Success and Scale Bring Broad Responsibility

### Vague Interview Question
Tell me about a time you considered broader responsibility beyond immediate functionality while designing an AWS system.

### Leadership Principle Being Probed
**Success and Scale Bring Broad Responsibility**

### STAR Answer
**Situation:** As cloud applications scale, architecture decisions affect more than feature delivery. They impact security, customer trust, availability, cost, sustainability, and operational risk.

**Task:** I had to design with broader responsibility in mind, especially for security and disaster recovery.

**Action:** For disaster recovery, I evaluated RTO and RPO first, then selected from backup and restore, pilot light, warm standby, or active-active multi-region patterns. For security, I designed in layers: TLS using ACM, DNS through Route 53, WAF for web threats, Shield for DDoS protection, API Gateway throttling, NACLs, security groups, IAM roles and policies, resource policies, SCPs, permission boundaries, KMS encryption, Secrets Manager or SSM Parameter Store, CloudTrail, GuardDuty, Inspector, and Access Analyzer. I also considered cost and sustainability because over-provisioning everything for maximum resilience may not be responsible if the business requirement does not justify it.

**Result:** The architecture balanced functionality with broader operational and business responsibility. It protected customers, reduced risk, and aligned resilience investment with actual recovery requirements instead of blindly over-engineering.

---

# Quick Revision Index

| Leadership Principle | Strongest Technical Anchor |
|---|---|
| Customer Obsession | Big traffic day, caching, CDN, queues, customer experience |
| Ownership | Aurora Serverless + RDS Proxy cost behaviour |
| Invent and Simplify | ElastiCache Serverless to provisioned ElastiCache simplification |
| Are Right, A Lot | AWS service selection using requirements and trade-offs |
| Learn and Be Curious | GenAI, RAG, Bedrock, vector DB, MCP-style tooling |
| Hire and Develop the Best | Explaining Lambda/EKS scaling and Well-Architected trade-offs |
| Insist on the Highest Standards | Security, scalability, DR, Well-Architected review |
| Think Big | Marketplace/ticketing microservices and event-driven design |
| Bias for Action | Proactive high-traffic readiness plan |
| Frugality | Reducing VPC endpoint/cache costs |
| Earn Trust | Transparent communication of trade-offs |
| Dive Deep | Lambda concurrency and EKS pod/node scaling mechanics |
| Have Backbone; Disagree and Commit | Challenging managed-service default choices |
| Deliver Results | Turning architecture into concrete AWS services |
| Strive to be Earth’s Best Employer | Operable, clear microservice boundaries |
| Success and Scale Bring Broad Responsibility | Security, DR, sustainability, and responsible scale |

---

# Final Self-Check

- All 16 Amazon/AWS Leadership Principles are covered.
- Each section contains a vague behavioural interview question.
- Each section explicitly names the leadership principle being probed.
- Each answer is written in STAR format.
- The answers are based on the provided AWS technical material: Lambda/EKS scaling, Well-Architected Framework, GenAI/RAG/Agentic AI, microservices, service selection, scalability, DR, security, and architecture trade-offs.
- The content is tailored for an AWS Solution Architect interview and avoids generic non-technical behavioural answers.
