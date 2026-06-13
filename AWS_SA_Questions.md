# AWS SA Questions

## 1. How does Lambda and EKS Scale

Thanks for the question. Lambda is the backbone of serverless compute however there are some important parameters that we need to understand if we are using lambdas as our compute service.

The first thing to understand that Lambda does remain always on. It is on demand.

If there is no request or no traffic there is no instance of lambda which is running. There is a subtle caveat in that which we will come to later.

So, no traffic then there is no lambda. Now as soon as traffic hits the api gateway, lambda invocations start and lambda instances start coming up. Important thing to note here is that for one request there is only one lambda invocation. One to one mapping.



## 2. Have you used Well Architected Framework and How


## 3. How do you secure applications on AWS


## 4. What are some of the GenAI use cases we can implement


## 5. What is RAG in AI


## 6. Create or Explain a Microservice


## 7. How do you select or compare services on AWS


## 8. How can you make your application scalable for big traffic day

### Front End Systems:

#### Infrastructure Level
- Use CDN to cache some of the content at the edge level
- Pre Warm Application Load Balancers, submitting support request from AWS Console
- Auto Scaling Group with Scheduled Scaling
- Minimum no and desired no of EC2 instances to be appropriate and more than 1.
- Provision some EC2 instances in warm pool which are pre-intialized to handle the initial burst of traffic
- Use a lightweight AMI for the EC2 instances so EC2 provisioning time is minimum and use Launch Templates.

### Application Level
- I can use a Queue service to hold the incoming traffic

### Backend System:
- I can use Event Driven Architecure with SQS/SNS+SQS/EvenBridge and scale my backend instances based on queue depth


### Database:
- Use RDS Proxy to connection pool into the RDS Databse if using RDS
- Use Cache wherever it can be used like for browse services
- Create partitions in DB intelligently to avoid hot partition or use GSI and LSI in case of Dynamo DB
- Create Read Replicas and Multi-AZ RDS DB Clusters with failover mechanisms
- For Aurora, we can use Aurora Serverless or Aurora Global Databases
- For Search, we can use Elastic Search

---

## 9. What is an AI Agent
- AI Agents is a software program which uses LLM for reasoning and can autonomously and independenly chose best set of actions and their order of executions till task completion and complete the required tasks.
- This is how AI Agents are diferent from conditional workflows and LLMs
- Example: Let's say we are using an agent to fix an issue in an AWS resources
    - The user can give the prompt to AI Agent
    - The agent can decide what are the set of tasks in sequence or parallel which can be invoked.
    - For example, getting logs from CloudWatch
    - Then it analyses the logs from the Cloudwatch and analyse if the error is coming from upstream or not.
    - Search the documentation to seek resolution
    - Invoke another tool to fix the issue 
    - Return Task Complete to the user.



## 10. How do you achieve DR for your cloud application

- Disaster Recovery is a very important aspect for a Cloud Architecture which touches upon the Reliability, Performace Efficiency, Cost Optimization and Sustainability pillars of Well Architected Framework. 
- The Disaster Recovery Strategy for a specific cloud application will depend on the trade offs and acceptable risks and decisiones between these Well Architected Framework pillars.
- Also, the decison to focus on which pillar depends upon two important parameters. The Recovery Point Object and the Recovery Time Objective
- Recovery Point Objective is the parameters which determines how much of data loss is acceptable. And from which point the data should be restored. If the recovery point objective is high that means the company can accept signifcant data loss from the point the last backup was taken. If Recovery Point Objective is very low or minimal or zero that measn almost no data loss is acceptable.
- Recovery Time Objective is the paramter which determines how soon after the disaster the system should be up and running in full capacity. If a recovery time objective is high that means that the system is allowed to be down for several hours and a recovery time objective of zero means there can absolutely be no downtime.
- Considering these factors, AWS recommends four Disaster Receovery Strategy
    
    - Backup and Restore - HIGHEST RTO, HIGHEST RPO. -> This is just take regular backups and snapshots. In the event of a diaster, restart the system from scratch and use the last available backup for data restoration.
    
    - Pilot Light - MEDIUM RTO, MEDIUM RPO - -> In this strategy only the most critical applications and databses are kept running in another region with replications. In case of a disaster, the most critical parts of the applications will keep running and continue to serve customers while the other services are started or replicated across the other region. The failover mechanism routes traffic to the other region.
    
    - Warm Standby: LOW RTO, LOW RPO: In this case the whole application is running on a different region and replication of data is syncronised. However, the application is not running at full capacity for example the no of instances or read replicas are small as compared to the main region. The failover mechanism routes traffic to the other region,

    - Active Active: LOWEST/NO RTO, LOWEST/NO RPO: In this case the entire application is replicated across multiple region and the requests are served at both regions simultaneously using DNS Routing. The Databses are syncronsied and replicated across regions. In case of a disaster in one region, the other region now handles all the volume of traffic and scales accordingly.



## 11. How do you secure your application on the cloud

- For a cloud application security is important, not at just outer level or web surface, security is important at each level and at each layer and at each service. AWS specifically provides us with all the rewuired tools to make the architecure secure and follow the Security Pillar of the Well Architected Framework and adhere to the Well Architected Framework guidelines.

- Also its not just implementing security mechanisms but also constantly monitoring for any security gaps or misses.

- If I had to design or build security for an AWS Cloud Application, i would first think in layers and then drill down to each resrouce.

    - At the outer or web/internet level, I would use Route 53 for DNS and Amazon Certificate Manager which provides TLS certificates which enables my application dns endpoints to have HTTPS protocol.

    - Then at ALB or API Gateway Level, I have two distinct level of security mechanisms. 
        - I can use AWS WAF (Web Access Firewall) to prevent SQL Injection, Cross Site Scriptings etc.
        - I can use Shield and Shield Advanced to prevent DDOS Attacks
        - Also I can use API Gateway as a Rate Limiter to prevent potential bot swarms attacking the application endpoints.

    - Then once the request is inside the VPC, NACLs which are stateless are my first point of defense where I can prohibit any specific IPs.

    - Once its passes NACL, then at resource level, I can use security groups around my resources to control ingress and egress. For example, my lambda (if inside VPC) or EC2 security group can only accept ingress from my ALB Security group. This prevents any unauthorized access to my resources. Also I can control the egress from a resource to another security group so that any other resources cannot serve requests from my resources.

    - Apart from security groups I can implement IAM Identity Policies for Users and Resources (Via Trust Policy) to create Roles for each resource which can restrict role based access and do only specific tasks which are restricted by the IAM Policies.

    - Also I can implement Resource Based Policies like S3 bucket policies which can allow or deny and cross account access.

    - If I am implementing AWS Organzations and Organization Groups I can implement Service Control Policies and Resource Policies across Accounts and Organizations to Allow or Deny any specific actions or resources.

    - I can use permission boundaries to have a maximum set of permissions for any resources.

    - For data encryption I would use Encryption at Rest and use KMS for encryption in transit. Also for credential management I will use AWS Secret Manager or SSM Paramter store.

    - On the monitoring part, I can use AWS Inspector for EC2, Lambdas etc and AWS GuardDuty and montior access by AWS CloudTrail or Access Analyzer.


## 12. Describe an architecure you designed


## 13. Biggest challenge faces during desiging your application on AWS


## 14. How do you pick one service over another in AWS


## 15. What is your favorite AWS Service and how would you improve it


## What is AWS Service X

