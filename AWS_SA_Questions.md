# AWS SA Questions

## 1. How does Lambda and EKS Scale

Thanks for the question. Lambda is the backbone of serverless compute however there are some important parameters that we need to understand if we are using lambdas as our compute service.

The first thing to understand that Lambda does not remain always on. It is on demand.

If there is no request or no traffic there is no instance of lambda which is running. There is a subtle caveat in that which we will come to later.

So, no traffic then there is no lambda. Now as soon as traffic hits the api gateway, lambda invocations start and lambda instances start coming up. Important thing to note here is that for one request there is only one lambda invocation. Its one to one mapping.

As soon as more traffic and requests starts coming in, more and more instance of lambda functions are invoke. But there is a limit. The no of concurrent lambda services which can be invoked depend on the **Reserved Concurrency** of the lambda function. By default its 1000 for an account (which can be increased by requestinng AWS to increase service quotas) and we can allcate from this pool to each service.
Once the **Reserved Concurrency** limit is reached for any lambda service the incoming requests starts to get throttled.
**Reserved Concurreny** is important because if we have many lambda services in our architecture then, its important that all services can scale up according to demand and not one service consumes all the reserved concurrency limit.

Another important factore in lambda concurrency is **Provisioned Concurrency**. It is the no of concurrent lambda instances which are pre initialized to immediately serve any request.

**Provisioned Concurrency** of a lambda function **can not* be greater than the **Reserved Concurrency**.

Some other areas on lambdas we can look upon are the lambda initialization time. Its important to keep this time as low as possible so use light weight libraries or package libraries in local lambda layers.


Now the next service we are going to talk about is EKS or Elastic Kubernetes Service
Scaling of EKS is at two level. One level is the **worker node level* and the another level is at the **cluster level*.
At the worker node level, the pods are running through deployments and their cpu and memonry requests and limits are set. This is monitored continously by the **Horizontal Pod Autoscalar** or **HPA**. HPA continously tracks the resource consumed for each deployment for which its defined. Based on the criteria set for the the deployment (for example Average Utilization at 50%), when the criteria is reached, the **Kube-Scheduler** at **control plane* schedules another pod so that the total average utilization comes down for the deployment. This scaling activities continues at the worker node level. 
Now there comes a time, when multiple pods are scheduled and running and the total worker node (EC2 instance) cpu or memory is reached. At that time inside the worker node when **kube scheduler** scheudles pod, there is no available cpu and memory for it. So it remains **Pending Unschedulable** staus. At this point the **Cluster Autoscalar or Karepnter** look for such pods and if found it provisoned another worker node (EC2 instance) via the Auto Scaling Group for cluster Auto Scalar or EC2 Fleet for Karepenter. Now the kube-scheduler finds available cpu/memory for the pod which is pending and scheduled pod in the new worker node (EC2 instance).

---

## 2. Have you used Well Architected Framework and How
Thanks for the question. 

The AWS Well Architected Framwework is a set of best practices for customers and organizations to evaluate their cloud architecture on AWS so that it closely aligns to AWS Best Practices.

There are **Six** AWS Well Architected Pillar.
- 1. Operational Excellence
- 2. Security
- 3. Reliability
- 4. Performance Efficiency
- 5. Cost Optimization
- 6. Sustainability

It is not a definitive but iterative process and should be reviewed periodically to intorduce changes and trade-offs wherever neccessary. It depends on the system requirements and the business priorrities. For example, if the system requirement or business priority is Reliability and Efficiency it will have an impact on Cost Optimization. If the priority is on Security then it can have an impact on Operational Excellence.

One such example of a trade-off between the well architected pillar I encounterd recently was during the designing of a highly available and Performance Efficient Online Ticketing System. One of the requirements was that during high traffic days the Databse should be able to support the millions of transactions but on normal days the traffic can scale even to 0. For this inconsistent traffic requirement and to optimize costs we decided to use Aurora Serverless which can scale back to 0 ACUs if no requests are served and has the auto pause functionality. But to serve traffic on high traffic days or events, we connected it with a RDS Proxy so that the incoming requests are throttled through the connection pool so that the database is not overloaded. However, since the RDS proxy always keep open connection to database, the auto pause and zero scaling functionality of Aurora Serverless was never utilised and the DB was billed 24/7. This led to high AWS bills and we needed to replace the Aurora Serverless with Provisoned Aurora Cluster with Read Replicas.

Another such example was using, Elasticache Serveless service to serve read request from lambdas inside the private VPCs. Since the lambdas were inside VPC, and Elasticache Serverless is not VPC bound, it created multiple vpc endpoints per AZ to create connections to the lambda services. This resulted in high performance but the cost of vpc endpoint was too high. To mititgae it we needed to provision the Elasticache inside VPC which increased operational overhead but reduced costs, because now lambdas could directly talk to Elasticache without the need for VPC Endpoints.

---

## 3. What are some of the GenAI, Agentic AI and RAG use cases we can implement

**Generative AI** or Gen AI is built on Foundational Large Language Models that excel at understanding, analysing and generating unstructured data at scale.

**RAG** or Retreival Augmented Generation helps in one of the major limitations of Gen AI which is understating, analysic and generating information specific to an enterprise or a use case which may or may not be part of the LLMs general models.

**Agentic** AI is a set of software programs which are built on top of LLMs which can take independent autonomous descision and plan and execute a sequence of independent steps to complete the required tasks.

A great example of enterprise level use case for these technologies is the creating customer tickets and service requests and handling them.

First we have our local, entrprise data on our local databse. This can be fed into an Amazon Bedrock Embedding Large Language Model to create a Vector Database on OpenSearch or Kendra or S3 Vector DB. once this process is complete, it becomes our RAG Knowledge Base.

Then we create our AI Agents on top of the Bedrock using any of the LLMs which uses MCP layer to connect the MCP servers which serve differnt tools like JIRA, Salesforce, or Confluence. Using MCP is a great way to leverage the tools and enahnce the capabilities of AI Agents because now the Agents can just trigger the MCP Client and decide which MCP server to connect to perform the set of actions on the respecitve tool.

Next to decrease our token cost and context window we make rules that only the most essential information will go through the RAG Vector Database and the MCP Servers which is essential to perform the tasks. The API Gateway integrations will transforms the request into shorter contexts holding essential informations only.

Once the user submits a complain through the context window, the API Gateway transforms the request. There will be SQS queue which will take the incoming requests. The AI Agent will take the incoming request from the queue and run it through the RAG Vector Database and then with the augmented information trigger the apprpriate MCP server to take actions on specific tools and then return the ticket no to the customer for tracking purpose.

---

## 4. Create or Explain a Microservice

The most important features of microservices are that they are independently deployable, maintained, invoked and executed without any dependencies. They have their own persistence layer and the failure of one microservice doesnt impact the functioning of the other microservices. Also each microservice can be a diffrernt service based on the use case and system requirements. For example, in the same architecture, for long running process we can use EC2 instances while for short workloads we can use Lambda functions. For relations between entities we can use RDS or Aurora and for faster performance we can use DynamoDB. Also they can be independently coded and implemented. This means that some microservices can be on nodeJS, some can be on Python or other can be on Go. This is called polyglot architecture.

A great example of microservice architecture can be an Online Marketplace like Amazon.
Where each functional requirement can be a different implementation of microservices inside the same ecosystem
For example:
    Sync Architecure
        API GATEWAY -> /search -> Lambda -> OpenSearch
        API GATEWAY -> /browse -> ECS Fargate -> ElastiCache Redis -> Aurora/RDS
        API GATEWAY -> /cart -> EKS -> ElastiCache -> DynamoDB
    EDS/ASync Architecture
        Event Choreogaphy (Complex, Efficient, Operational Overhead)
            Order Placed -> Event Bridge -> Payment -> Lambda -> 3rd Party -> Lambda ->  RDS
                                        -> Inventory -> ECS -> Dynamo DB
                                        -> Shipping -> EC2 -> RDS
                                        -> Notifications -> SNS
        Workflow Orchestration (Simple, Less Montioring, ACID Transactions)
            Order Placed -> AWS Step Functions   -> Payment -> Lambda -> 3rd Party -> Lambda ->  RDS
                                                 -> Inventory -> ECS -> Dynamo DB
                                                 -> Shipping -> EC2 -> RDS
                                                -> Notifications -> SNS
        
    
---

## 7. How do you select or compare services on AWS


---

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

---

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



## 15. What is your favorite AWS Service and how would you improve it


## 16. What is High Availibility



## 20. What is EDA


## 21. How do you pick one service over another in a Microservice architecrure

