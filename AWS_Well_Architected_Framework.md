# AWS Well Architected Framework

## What is AWS Well Architected Framework
AWS Well Architected Framework is a set of best practices for AWS customers and partneres to evaluate architecture and provide a set of guidelines to evaluate how well the architecture is aligned to AWS best practices.

## The 6 Pillars of AWS Well Architected Framework:

- Operational Excellence 
- Security
- Reliability
- Performance Efficiency
- Cost Optimization
- Sustainability

## What are the things to monitor and chceck in each of the pillars


### Operational Excellence
    The ability to support development and run workloads effectively, gain insight into their operations, and to continuously improve supporting processes and procedures to deliver business value.

- Infrastructure as Code (IAC)
- CI/CD
- Montioring Tools, Alerts, Logs, Errors, Traces
- Incident Monitoring
- Bug Reporting and Resolution

### Security
    The security pillar describes how to take advantage of cloud technologies to protect data, systems, and assets in a way that can improve your security posture. 

- Identity and Access Management (Polices, Roles, Users, User Groups)
- Service Boundaries
- Encryption at Rest, Encryption at Transit, KMS
- Secret Manager, SSM Parameter Store
- Security Groups
- NACLs (Network Access Control List)
- Service Boundaries
- WAF (Web Application Firewall)
- Shield for DDOS Attacks

### Reliability
	The reliability pillar encompasses the ability of a workload to perform its intended function correctly and consistently when it’s expected to. This includes the ability to operate and test the workload through its total lifecycle. This paper provides in-depth, best practice guidance for implementing reliable workloads on AWS.

- Multi AZ Design
- DR Strategy (Pilot Light, Warm Standby, Active Active)
- Backup and Restore
- RTO and RPO
- Health Checks


### Performance Efficiency
    The ability to use computing resources efficiently to meet system requirements, and to maintain that efficiency as demand changes and technologies evolve.

- Auto Scaling, Predictive Scaling
- Event Driven Architecture
- SQS, SNS for queuing of requests
- RDS Proxy, Connection Pooling
- Lambda Concurrency
- Aurora Serverless
- Caching Strategies

### Cost Optimization
    The ability to run systems to deliver business value at the lowest price point.
- Cost Allocation Tags
- Cost Expolrer
- Budgets and Alarms
- Validating usage of NAT Gateways over VPC Endpoints
- Serverless Resources vs Provisoned Resources
- Right Sizing of Provisioned Respirces
- Savings Plan
- Storage Lifecycle

### Sustainability
    The ability to continually improve sustainability impacts by reducing energy consumption and increasing efficiency across all components of a workload by maximizing the benefits from the provisioned resources and minimizing the total resources required.
- Utilization
- Managed Services
- Graviron suitability
- Elimination of Idle Capacity


## Case Study

### Situation (S)
A telecom provider had a customer-facing billing and payments workload on AWS. The platform was stable in normal traffic, but during bill-run windows the API latency crossed 8 seconds, operational teams manually scaled components, and monthly AWS spend had increased by nearly 45% over two quarters. Security findings also showed broad IAM roles, inconsistent KMS key usage, and incomplete restore testing.

### Task (T)
My task was to run an architecture review that produced an actionable remediation roadmap. The goal was not to declare the architecture “bad”; it was to identify high-risk issues, quantify business impact, sequence improvements, and make trade-offs explicit across reliability, security, performance, operations, cost, and sustainability.

### Action (A)
- I first defined the workload boundary: customer entry points, API tier, authentication, compute services, databases, queues, object storage, observability, CI/CD, and external dependencies. 
- I captured business metrics: peak transactions per minute, RTO, RPO, revenue impact per minute of outage, compliance requirements, expected growth, and cost ownership. 
- I then ran pillar-specific sessions. 
- For operational excellence, I reviewed IaC coverage, deployment rollback, runbooks, alarms, and incident history. 
- For security, I reviewed identity foundation, encryption, logging, network exposure, secret rotation, and incident response. 
- For reliability, I reviewed multi-AZ design, backup restore tests, dependency failure behavior, quotas, and scaling limits. 
- For performance, I reviewed latency sources, caching, database query plans, concurrency, and queue depth. 
- For cost, I reviewed cost allocation tags, idle resources, NAT processing, log retention, database sizing, Savings Plans readiness, and storage lifecycle. 
- For sustainability, I reviewed utilization, managed services, Graviton suitability, and elimination of idle capacity. 
- I converted findings into HRI/MRI-style risks with owner, business impact, proposed fix, cost impact, and reversibility.

### Result (R)
- The review produced a 1 Quarter Technical Resilence Roadmap Backlogs.
-  High-priority fixes included database backup restore testing, least-privilege role redesign, NAT-to-VPC-endpoint optimization, autoscaling based on queue depth, CloudWatch alarm rationalization, and deployment rollback automation. 
- In the expected outcome model, latency risk reduced through async decoupling, security blast radius reduced through role boundaries, and projected monthly cost reduced by 20–30% by eliminating idle capacity and unnecessary NAT/data-transfer paths.

### Whiteboard
    Customer / Business Objective

        |
        v
    Workload Boundary -> Dependencies -> Data Classification -> RTO/RPO -> Traffic Profile -> Cost Ownership
        |
        v
    Well-Architected Review Sessions
        |-- Operational Excellence: IaC, runbooks, CI/CD, alarms, incident learnings
        
        |-- Security: IAM, KMS, logging, network, secrets, detection
        
        |-- Reliability: AZ/Region failure, backup restore, quotas, scaling, dependency failures
        
        |-- Performance: latency, caching, compute/database fit, load tests
        
        |-- Cost: tags, rightsizing, commitments, storage lifecycle, data transfer
        
        |-- Sustainability: utilization, managed services, efficient architectures
        
        |
        v
    Risk Register -> Prioritized Remediation Roadmap -> Measured Improvements


## Tradeoff Examples:

- Multi AZ Architecture imporves Reliability but impacts Cost Optimizations

- Event Driven Architecure with Caching, Queues and Event Bridge increases Performace Efficiency but increases Operational Overhead with complex monitoring systems for identification of failure points.

- Strong encryption, restrictive security groups, IAM Roles and Policies increases Security Postures but can create recovery complexities in case of loss of credentiasl

- Aggresive Caching improves Performance and Efficiency but can introduce stale or eventually consistent data into the system.

## Conclusion
The Well Architected Framework is not a checklist of resources but a guideline for customer and business owners to adhere as closely to the AWS Best Practices. It is not a definitive but iterative process and should be reviewed periodically to intorduce changes and trade-offs wherever neccessary.