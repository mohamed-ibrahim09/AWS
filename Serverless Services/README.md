# ☁️ AWS Serverless Services — SAA-C03 Study Reference Guide

<div align="center">

![AWS](https://img.shields.io/badge/AWS-SAA--C03-orange?style=for-the-badge&logo=amazonaws)
![Lambda](https://img.shields.io/badge/Lambda-Serverless%20Compute-blue?style=for-the-badge&logo=awslambda)
![DynamoDB](https://img.shields.io/badge/DynamoDB-NoSQL-success?style=for-the-badge)
![APIGateway](https://img.shields.io/badge/API%20Gateway-REST%20%26%20WebSocket-yellow?style=for-the-badge)

### ⚡ Lambda • DynamoDB • API Gateway • Step Functions • Cognito • Edge Computing

*Comprehensive SAA-C03 study reference covering serverless compute, NoSQL databases, API management, workflow orchestration, and user authentication — with architecture diagrams, exam traps, and high-frequency facts.*

</div>

---

# 📖 Overview

This guide covers the **Serverless domain** of the SAA-C03 exam — one of the highest-density scenario topics. You'll learn:

* **AWS Lambda** — serverless compute fundamentals, limits, concurrency, cold starts
* **Lambda + VPC/RDS** — networking and database integration patterns
* **Amazon DynamoDB** — NoSQL database, capacity modes, streams, global tables
* **API Gateway** — REST/WebSocket API management and security
* **AWS Step Functions** — serverless workflow orchestration
* **Amazon Cognito** — user authentication and federated identity
* **Edge Computing** — CloudFront Functions vs Lambda@Edge

---

# ☁️ What is Serverless?

## Definition

Serverless is a paradigm where developers deploy **code and functions** without provisioning, managing, or scaling servers themselves.

> **Common misconception:** Serverless does **NOT** mean there are no servers. It means **you** don't see, manage, or provision them — AWS does.

## Evolution

> **Common misconception:** Serverless does **NOT** mean there are no servers. It means **you** don't see, manage, or provision them — AWS does.

## Evolution

```
Serverless originally = FaaS (Function as a Service)
        │
        │  pioneered by AWS Lambda
        ▼
Now includes ANY managed service:
   Databases (DynamoDB, Aurora Serverless)
   Messaging (SNS, SQS)
   Storage (S3)
   Compute (Fargate)
```

## Key AWS Serverless Services

| Category | Services |
| :--- | :--- |
| Compute | AWS Lambda, AWS Fargate |
| Database | Amazon DynamoDB, Aurora Serverless |
| Storage | Amazon S3 |
| Messaging | SNS, SQS, Kinesis Data Firehose |
| API | API Gateway |
| Auth | AWS Cognito |
| Orchestration | Step Functions |

---

## Sample Serverless Web App Architecture


![Serverless in AWS](assets/Serverless%20in%20AWS.png)



This pattern — S3 for static hosting, API Gateway + Lambda for dynamic logic, DynamoDB for data, Cognito for auth — is the **canonical serverless web app** referenced repeatedly across SAA-C03 scenarios.

---

# ⚡ AWS Lambda

## Why Lambda vs EC2?

| Aspect | Amazon EC2 (Virtual Servers) | AWS Lambda (Virtual Functions) |
| :--- | :--- | :--- |
| Server management | You manage servers | ✅ No servers to manage |
| Constraint | Limited by RAM/CPU | Limited by **time** (short executions) |
| Running state | Continuously running | Runs **on-demand** |
| Scaling | Manual intervention | ✅ Fully automated |
| Billing | Pay for uptime | Pay per request + duration |

---

## Lambda Benefits

| Benefit | Detail |
| :--- | :--- |
| Easy pricing | Pay per request + compute time; free tier: **1M requests + 400,000 GB-seconds/month** |
| Broad integration | Works across the entire AWS ecosystem |
| Easy monitoring | Native **CloudWatch** integration |
| Flexible resources | Up to **10 GB RAM**; more RAM also boosts CPU and network performance |

---

## Lambda Language Support

| Category | Options |
| :--- | :--- |
| Native runtimes | Node.js, Python, Java, C# (.NET Core) / PowerShell, Ruby |
| Custom runtimes | Community-supported via **Custom Runtime API** (e.g., Rust, Go) |
| Container images | Supported — but must implement the **Lambda Runtime API** |

> **Exam trap:** For running **arbitrary Docker images** (not specifically built for the Lambda Runtime API), **ECS/Fargate is the preferred choice**, not Lambda container images. Lambda containers are still fundamentally *functions* under the hood — same time/memory limits apply.

---

## Lambda Main Integrations

![AWS Lambda Integrations](assets/AWS%20Lambda%20Integrations%20Main%20ones.png)

---

## Lambda Use Case Examples

### Serverless Thumbnail Creation
![Example Serverless Thumbnail Creation](assets/Example%20Serverless%20Thumbnail%20creation.png)

### Serverless CRON Job
![Example Serverless CRON Job](assets/Example%20Serverless%20CRON%20Job.png)

---

## Lambda Pricing Deep Dive

### Pay Per Request
| Tier | Cost |
| :--- | :--- |
| First 1,000,000 requests/month | **Free** |
| After free tier | $0.20 per 1M requests ($0.0000002/request) |

### Pay Per Duration (1 ms increments)
| Tier | Cost |
| :--- | :--- |
| First 400,000 GB-seconds/month | **Free** |
| After free tier | $1.00 per 600,000 GB-seconds |

**Free tier examples by memory size:**

| Function RAM | Equivalent Free Seconds |
| :--- | :--- |
| 1 GB | 400,000 seconds |
| 128 MB | 3,200,000 seconds |

> **Insight:** Lower memory allocation stretches your free tier much further in raw execution time — but higher memory also means faster execution (more CPU), so total GB-seconds consumed may actually be similar or lower for a well-optimized function.

---

## Lambda Limits (Per Region) — MEMORIZE THESE

### Execution Limits

| Limit | Value |
| :--- | :--- |
| Memory allocation | **128 MB – 10 GB** (1 MB increments) |
| Max execution time | **900 seconds (15 minutes)** |
| Environment variables size | **4 KB** |
| `/tmp` disk capacity | **512 MB – 10 GB** |
| Concurrent executions | **1,000** (soft limit — can request increase) |

### Deployment Limits

| Limit | Value |
| :--- | :--- |
| Deployment package (compressed .zip) | **50 MB** |
| Uncompressed deployment size | **250 MB** |
| Container image size | Up to **10 GB** (via ECR) |

> **Exam trap:** If a function needs to run **longer than 15 minutes**, Lambda is the WRONG answer — consider Step Functions (to orchestrate multiple Lambda calls), Fargate, or EC2/Batch instead.

---

## Lambda Concurrency & Throttling

**Concurrency** = the number of in-flight (currently executing) requests your function is handling at once.

```
Region-wide default: 1,000 concurrent executions (shared across ALL functions)
```

### Reserved Concurrency

Sets a **dedicated cap** for one specific function — it acts like a private lane that:
* **Guarantees** that function can always scale up to its reserved amount
* **Limits** that function so it can never exceed the reserved amount
* **Reduces** the pool available to all other functions in the account

```
Total Account Concurrency: 1000
        │
        ├── Function A: Reserved Concurrency = 200  (guaranteed, capped at 200)
        │
        └── Remaining pool for all other functions = 800
```

### Throttling Behavior

| Invocation Type | Behavior on Throttle |
| :--- | :--- |
| **Synchronous** (API Gateway, ALB, direct SDK call) | Returns `ThrottleError - HTTP 429` immediately |
| **Asynchronous** (S3 events, SNS, EventBridge) | Automatically retries, then routes to **DLQ (Dead Letter Queue)** if it keeps failing |

---

## The Concurrency Starvation Problem

```
1000 total concurrent executions available
        │
        ▼
┌────────────────────────────────────────────┐
│  Function A (behind ALB, high traffic)      │
│  consumes all 1000 concurrent executions    │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│  Function B (API Gateway, few users)        │
│  Function C (called via SDK/CLI)            │
│  → BOTH GET THROTTLED! ⚠️                   │
└────────────────────────────────────────────┘
```

> **The fix:** Set **Reserved Concurrency** on critical functions to guarantee they always have capacity, regardless of what other functions are doing.

---

## Asynchronous Retry Behavior

For throttling errors (429) and system errors (500-series) on **async** invocations:

```
Event fails ──▶ Lambda returns event to internal queue
                        │
                        ▼
              Retries with EXPONENTIAL BACKOFF
              (1 sec → up to 5 min max interval)
                        │
                        ▼
              Keeps retrying for UP TO 6 HOURS
                        │
                        ▼ (still failing)
                   Dead Letter Queue (DLQ)
                   or On-Failure Destination
```

---

## Cold Starts & Provisioned Concurrency

### What is a Cold Start?

```
New Lambda instance needed
        │
        ▼
   INIT PHASE (cold start)
   - Download code
   - Load runtime
   - Run code OUTSIDE the handler (imports, SDK setup, DB connections)
        │
        ▼
   INVOKE PHASE
   - Run the actual handler function
        │
   First request = HIGH LATENCY (waited through Init + Invoke)
   Subsequent requests on same warm instance = LOW LATENCY (skip Init)
```

The larger your dependencies (SDKs, libraries, VPC ENI attachment), the longer the Init phase takes.

### Provisioned Concurrency — The Fix

```
Provisioned Concurrency = pre-warm instances BEFORE they're invoked
        │
        ▼
   Init phase already completed in advance
        │
        ▼
   EVERY invocation goes straight to Invoke phase
        │
        ▼
   Cold start NEVER happens for provisioned instances
```

| Feature | Reserved Concurrency | Provisioned Concurrency |
| :--- | :--- | :--- |
| Purpose | Caps/guarantees a concurrency **quota** | Pre-warms instances to **eliminate cold starts** |
| Analogy | A dedicated lane (limits usage) | Cars already running in the lane, ready to go |
| Cost impact | No extra cost | Extra cost — you pay for idle warm capacity |
| Managed by | Manual configuration | Can be managed by **Application Auto Scaling** (schedule or target utilization) |

> **Exam trap:** Reserved Concurrency and Provisioned Concurrency solve **different problems**. Reserved = guarantee/limit capacity. Provisioned = eliminate cold start latency. A question about "first request is slow" → Provisioned Concurrency. A question about "one function is starving others" → Reserved Concurrency.

> **Historical note:** Cold starts for Lambda functions **inside a VPC** were dramatically reduced by AWS improvements in Oct/Nov 2019 (Hyperplane ENI architecture) — this used to be a much bigger concern before that.

---

## Lambda SnapStart

| Feature | Detail |
| :--- | :--- |
| Performance gain | Up to **10x faster** startup |
| Cost | **No extra cost** |
| Supported languages | **Java, Python, .NET** |
| Mechanism | Pre-initialized **snapshot of memory and disk state**, cached for reuse |

### SnapStart Process

```
On function publish:
1. Lambda fully initializes your function once
2. Takes a SNAPSHOT of memory + disk state
3. Snapshot cached for fast, low-latency access
        │
        ▼
On invocation:
Function resumes DIRECTLY from the snapshot
→ Init phase is SKIPPED entirely
→ Goes straight to Invoke
```

> **SnapStart vs Provisioned Concurrency:** Both eliminate cold starts, but SnapStart does it **for free** (for supported languages) by caching a snapshot, while Provisioned Concurrency costs money to keep instances continuously warm.

---

# 🌐 Customization at the Edge

## Why Run Code at the Edge?

Modern applications execute logic **geographically close to users** to minimize latency — instead of a request traveling all the way to a single origin region.

| Feature | Detail |
| :--- | :--- |
| Servers to manage | None |
| Deployment | Global (all CloudFront edge locations) |
| Pricing | Pay only for what you use |

**Use cases:** CDN content customization, website security, dynamic web apps, SEO, intelligent routing, bot mitigation, real-time image transformation, A/B testing, user authentication, user prioritization, tracking/analytics.

---

## CloudFront Functions vs Lambda@Edge

![CloudFront Functions](assets/CloudFront%20Functions.png)

### Full Comparison Table

![CloudFront Functions vs Lambda@Edge](assets/CloudFront%20Functions%20vs.%20Lambda@Edge.png)

### When to Use Which

| Use CloudFront Functions for | Use Lambda@Edge for |
| :--- | :--- |
| Cache key normalization (headers, cookies, query strings) | Longer execution / complex logic |
| Header manipulation (insert/modify/delete) | Adjustable CPU/memory needs |
| URL rewrites or redirects | 3rd-party library dependencies (AWS SDK calls) |
| Lightweight request auth (JWT validation) | Network access to external services |
| | File system access or reading the request body |

> **Exam trap:** If the scenario needs to call **another AWS service** (like S3 or DynamoDB) or use the **AWS SDK** from within edge logic, CloudFront Functions **cannot do this** — it has no network access. The answer must be **Lambda@Edge**.


---

# 🔌 Lambda Networking & Database Integrations

## Lambda's Default Network Behavior

> **Critical default:** By default, a Lambda function runs **OUTSIDE your VPC**, inside an AWS-managed VPC.

| Impact | Detail |
| :--- | :--- |
| Public internet access | ✅ Yes, by default |
| Access to YOUR VPC resources | ❌ No — cannot reach private RDS, ElastiCache, internal ALBs |

![Lambda by default](assets/Lambda%20by%20default.png)

---

## Placing Lambda Inside a VPC

To let Lambda reach private resources, you must explicitly configure:

| Setting | Purpose |
| :--- | :--- |
| VPC ID | Which VPC to attach to |
| Subnets | Which subnets Lambda can use |
| Security Groups | What traffic is allowed |

```
Lambda Function (configured with VPC + Subnets + SG)
        │
        ▼
Creates an ENI (Elastic Network Interface)
in your specified subnet
        │
        ▼
Can now reach:
  ✅ Amazon RDS (private subnet)
  ✅ ElastiCache
  ✅ Internal ALBs
```

> **The tradeoff:** Once Lambda is placed in your VPC, it **loses default public internet access**. To reach the public internet or public AWS services (like S3, DynamoDB) from a VPC-attached Lambda, you need a **NAT Gateway** (for private subnets) or **VPC Endpoints** (for direct private access to supported AWS services).

```
Lambda in VPC (private subnet)
        │
        ├──▶ NAT Gateway ──▶ Internet (for public internet access)
        │
        └──▶ VPC Endpoint ──▶ S3 / DynamoDB (private access, no internet needed)
```

---

## Lambda with RDS Proxy

### The Problem

```
High-traffic Lambda functions
        │
        ▼
Each invocation opens its OWN database connection
        │
        ▼
Under load: THOUSANDS of connections hit the database
        │
        ▼
Database becomes OVERWHELMED / runs out of connections ❌
```

### The Solution — RDS Proxy

![Lambda with RDS Proxy](assets/Lambda%20with%20RDS%20Proxy.png)

| Benefit | Detail |
| :--- | :--- |
| **Scalability** | Pools/shares DB connections instead of opening new ones per invocation |
| **Availability** | Reduces failover time by **66%**; preserves connections during failover |
| **Security** | Enforces **IAM authentication**; credentials stored in **Secrets Manager** |

> **Critical requirement:** RDS Proxy is **NEVER publicly accessible**. This means the Lambda function connecting to it **MUST be deployed inside your VPC**.

---

## Invoking Lambda from RDS / Aurora

You can trigger Lambda **from inside** a database instance (RDS for PostgreSQL and Aurora MySQL).

```
User registers ──▶ INSERT event in database
                          │
                          ▼
                   Database invokes Lambda
                          │
                          ▼
                   Lambda sends welcome email
```

| Requirement | Detail |
| :--- | :--- |
| Network | DB instance needs **outbound traffic** to Lambda (public internet, NAT GW, or VPC Endpoints) |
| Permissions | DB instance needs a **Lambda Resource-based Policy** + **IAM Policy** to invoke the function |

---

## RDS Event Notifications

> **Key distinction:** RDS Event Notifications tell you about the **DB instance itself** (created, stopped, started, storage full) — **NOT** about changes to the actual data inside tables.

| Aspect | Detail |
| :--- | :--- |
| Event categories | DB instance, DB snapshot, DB Parameter Group, DB Security Group, RDS Proxy, Custom Engine Version |
| Latency | Near real-time (up to **5 minutes**) |
| Delivery | **SNS**, or subscribe via **EventBridge** (which can then trigger SQS or Lambda) |

![RDS Event Notifications](assets/RDS%20Event%20Notifications.png)

> **Exam trap:** If a question asks about reacting to **data changes** (INSERT/UPDATE/DELETE on rows), RDS Event Notifications is the WRONG answer — that requires invoking Lambda directly from the DB, or using **DynamoDB Streams** if it's a NoSQL table.

---

# 🗃️ Amazon DynamoDB

## Overview

| Characteristic | Detail |
| :--- | :--- |
| Management | Fully managed, no maintenance/patching |
| Availability | Highly available — replicated across multiple AZs |
| Data model | **NoSQL** (not relational), supports transactions |
| Scale | Millions of requests/sec, trillions of rows, 100s of TB |
| Latency | **Single-digit millisecond** consistent performance |
| Security | Integrated with IAM |
| Cost | Low cost, supports auto-scaling |
| Table classes | Standard, and **Standard Infrequent Access (IA)** |

---

## DynamoDB Basics

![DynamoDB Table Example](assets/DynamoDB%20–%20Table%20example.png)

| Concept | Detail |
| :--- | :--- |
| Primary Key | Must be decided at table creation (Partition Key alone, or Partition + Sort Key) |
| Items | Equivalent to rows — unlimited count per table |
| Attributes | Equivalent to columns — can be added over time, can be null |
| Max item size | **400 KB** |
| Schema flexibility | ✅ Rapidly evolve schema — no rigid table structure required |

### Supported Data Types

| Category | Types |
| :--- | :--- |
| Scalar | String, Number, Binary, Boolean, Null |
| Document | List, Map |
| Set | String Set, Number Set, Binary Set |

---

## DynamoDB Capacity Modes

| Mode | How It Works | Best For |
| :--- | :--- | :--- |
| **Provisioned** (default) | You specify RCU/WCU ahead of time; supports auto-scaling add-on | Predictable, steady workloads |
| **On-Demand** | Automatically scales with traffic, no capacity planning | Unpredictable workloads, spiky traffic |

| Mode | Planning Required | Cost |
| :--- | :--- | :--- |
| Provisioned | ✅ Yes | Lower (if well-tuned) |
| On-Demand | ❌ No | Higher ($$$) |

> **RCU** = Read Capacity Unit, **WCU** = Write Capacity Unit — the billing units for Provisioned Mode.

---

## DynamoDB Accelerator (DAX)

An **in-memory cache** specifically built for DynamoDB.

![DynamoDB Accelerator (DAX)](assets/DynamoDB%20Accelerator%20(DAX).png)

| Feature | Detail |
| :--- | :--- |
| Latency | **Microseconds** for cached reads |
| Code changes | ❌ None needed — compatible with existing DynamoDB APIs |
| Default TTL | **5 minutes** |
| Purpose | Solves **read congestion** on hot items |

### DAX vs ElastiCache

| Feature | DAX | ElastiCache |
| :--- | :--- | :--- |
| Purpose | Caches individual DynamoDB objects, Query & Scan results | Caches aggregation results, general-purpose application data |
| Scope | DynamoDB-specific | Any application data |
| API compatibility | Same as DynamoDB API | Requires application logic to query cache separately |

---

## DynamoDB Stream Processing

An **ordered stream of item-level changes** (create/update/delete) on a table.

**Use cases:** Real-time reactions (welcome emails), real-time analytics, populating derivative tables, cross-region replication, triggering Lambda on changes.

### Two Stream Types

| Type | Retention | Consumers | Processing |
| :--- | :--- | :--- | :--- |
| **DynamoDB Streams** | 24 hours | Limited number | AWS Lambda Triggers, or DynamoDB Stream Kinesis Adapter |
| **Kinesis Data Streams** (newer) | **1 year** | High number | Lambda, Kinesis Data Analytics, Kinesis Data Firehose, AWS Glue Streaming ETL |

### Architecture Flow

```
Application ──▶ DynamoDB Table ──▶ Streams
                                       │
                       ┌───────────────┼───────────────┐
                       ▼               ▼               ▼
                    Lambda        KCL Adapter      (processing layer)
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      SNS          DDB Table      Kinesis Data
   (messaging)   (transformation)   Firehose
                                      │
                        ┌─────────────┼─────────────┐
                        ▼             ▼             ▼
                    Redshift      S3 (archive)   OpenSearch
                    (analytics)                  (indexing)
```

---

## DynamoDB Global Tables

```
Region: us-east-1              Region: eu-west-1
┌─────────────────┐            ┌─────────────────┐
│  DynamoDB Table  │ ◀────────▶│  DynamoDB Table  │
│  (Active)         │  Active-  │  (Active)         │
└─────────────────┘  Active     └─────────────────┘
   READ + WRITE        two-way      READ + WRITE
                     replication
```

| Feature | Detail |
| :--- | :--- |
| Purpose | Low-latency access to the same table from multiple regions |
| Replication type | **Active-Active** (two-way) |
| Access pattern | Applications can **READ and WRITE** in any participating region |
| Prerequisite | **DynamoDB Streams must be enabled** |

> **Exam trap:** Global Tables is Active-Active, unlike many other AWS replication features which are Active-Passive (like RDS Multi-AZ). Both regions can accept writes simultaneously.

---

## DynamoDB Time To Live (TTL)

![DynamoDB TTL](assets/DynamoDB%20–%20Time%20To%20Live%20(TTL).png)

**Use cases:** Keep only current/relevant data, meet regulatory data retention rules, manage web session expiration automatically.

---

## DynamoDB Backups for Disaster Recovery

| Backup Type | Window | Recovery Process | Best For |
| :--- | :--- | :--- | :--- |
| **Continuous Backups (PITR)** | Last **35 days** | Creates a **new table** at chosen point in time | Accidental writes/deletes, fine-grained recovery |
| **On-Demand Backups** | Until explicitly deleted | Creates a **new table** | Long-term retention, compliance archiving |

> **Both backup types create a NEW table on restore** — you never restore "in place" over the original table.

On-Demand Backups can be managed via **AWS Backup**, which also enables **cross-region copy**.

---

## DynamoDB Integration with Amazon S3

### Export to S3

| Requirement | Must enable **PITR** first |
| :--- | :--- |
| Time range | Any point within the last 35 days |
| Performance impact | ❌ None — doesn't affect table's read capacity |
| Formats | DynamoDB JSON or **ION** |
| Use cases | Analyze via **Athena**, audit snapshots, ETL before re-import |

### Import from S3

| Requirement | Detail |
| :--- | :--- |
| Formats | CSV, DynamoDB JSON, or ION |
| Write capacity consumed | ❌ None |
| Result | Creates a **new table** |
| Error handling | Logged in **CloudWatch Logs** |

```
S3 Data ──▶ Import ──▶ New DynamoDB Table (no WCU consumed)

DynamoDB Table (PITR enabled) ──▶ Export ──▶ S3 (JSON/ION) ──▶ Athena Analysis
```

---

# 🌉 AWS API Gateway

## Overview

> **Core concept:** AWS Lambda + API Gateway = a complete API with **zero infrastructure to manage**.

## Key Features

| Feature | Detail |
| :--- | :--- |
| Protocols | REST and **WebSocket** |
| Versioning | Handle API versions (v1, v2) and environments (dev, test, prod) |
| Security | Authentication and Authorization built-in |
| Throttling | API keys and request rate limiting |
| Import | **Swagger / OpenAPI** import for quick API definition |
| Transformation | Transform and validate requests/responses |
| SDK generation | Auto-generate client SDKs and API specs |
| Caching | Cache API responses to reduce backend load |

---

## API Gateway Integration Types

```
                 ┌─────────────────┐
Client ────────▶ │   API Gateway    │
                 └────────┬─────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
  Lambda Function     HTTP Endpoint      AWS Service
  (invoke function)  (on-prem HTTP,     (Step Functions,
                       internal ALB)     SQS, etc.)
```

| Integration | Purpose |
| :--- | :--- |
| **Lambda Function** | Easiest way to expose a REST API backed by a function |
| **HTTP** | Expose existing HTTP backends (on-prem, internal ALB) — add rate limiting, caching, auth, API keys |
| **AWS Service** | Directly expose AWS APIs (e.g., start a Step Function, post to SQS) with authentication and rate control layered on top |

### Example — Direct AWS Service Integration

```
Client ──▶ API Gateway ──▶ Kinesis Data Streams ──▶ Kinesis Data Firehose ──▶ S3 (.json files)
```

No Lambda needed in this pattern — API Gateway talks directly to Kinesis.

---

## API Gateway Endpoint Types

| Type | Best For | How It Works |
| :--- | :--- | :--- |
| **Edge-Optimized** (default) | Global clients | Routed through **CloudFront Edge locations** for lower latency; API Gateway itself still lives in one region |
| **Regional** | Clients in the same region | Direct regional access; can manually combine with CloudFront for custom caching/CDN control |
| **Private** | Internal-only access | Accessible **only from your VPC** via an **interface VPC endpoint (ENI)**; access controlled by a **resource policy** |

```
Edge-Optimized:
Global User ──▶ CloudFront Edge Location ──▶ API Gateway (single region)

Regional:
Regional User ──▶ API Gateway (same region) directly

Private:
VPC-internal client ──▶ Interface VPC Endpoint (ENI) ──▶ API Gateway (private)
```

---

## API Gateway Security

### Authentication Options

| Method | Best For |
| :--- | :--- |
| **IAM Roles** | Internal applications, AWS-to-AWS calls |
| **Cognito** | External users (mobile apps, public users) |
| **Custom Authorizer** | Your own custom auth logic (e.g., third-party JWT validation) |

### Custom Domain Name (HTTPS)

Uses **AWS Certificate Manager (ACM)** for the SSL certificate.

| Endpoint Type | Certificate Region Requirement |
| :--- | :--- |
| Edge-Optimized | Certificate **must be in us-east-1** |
| Regional | Certificate must be in the **same region as the API Gateway** |

Also requires a **CNAME or A-alias record** in **Route 53** pointing to the API Gateway domain.

> **Exam trap:** This mirrors the CloudFront/ACM us-east-1 requirement you may already know — Edge-Optimized API Gateway uses CloudFront under the hood, so it inherits the same **us-east-1 certificate rule**.

---

# 🔀 AWS Step Functions

## What is Step Functions?

A serverless service to build **visual workflows** that orchestrate multiple Lambda functions (and other AWS services) as a coordinated state machine.

## Core Features

| Feature | Detail |
| :--- | :--- |
| Sequencing | Run steps in order |
| Parallel execution | Run multiple branches simultaneously |
| Conditions | Branch logic based on data |
| Timeouts | Set max duration per step |
| Error handling | Built-in retry and catch logic |
| Human approval | ✅ Supported (pause workflow for manual sign-off) |

## Integrations

Step Functions can orchestrate:
* AWS Lambda
* Amazon EC2
* Amazon ECS
* On-premises servers
* API Gateway
* SQS queues
* And more AWS services via direct SDK integrations

## Visual Workflow States

![AWS Step Functions](assets/AWS%20Step%20Functions.png)

## Use Cases

* Order fulfillment pipelines
* Data processing / ETL pipelines
* Web application backend orchestration
* Any multi-step workflow needing retries, branching, or human approval

> **Exam trap:** If a Lambda-based workflow needs to run **longer than 15 minutes total** (Lambda's max execution time), or requires **complex branching/retry logic across multiple functions**, Step Functions is almost always the correct answer — it can coordinate many short Lambda invocations into a workflow of any length.

---

# 👤 Amazon Cognito

## Purpose

Give end-users of your web or mobile application an **identity** — either for logging in, or for accessing AWS resources directly.

## Two Main Components

```
┌──────────────────────────┐         ┌──────────────────────────┐
│  Cognito User Pools (CUP) │         │ Cognito Identity Pools    │
│  "Who is this user?"      │────────▶│ "What can this user       │
│  (Authentication)         │  Token   │  access in AWS?"         │
│                           │         │  (Authorization/AWS       │
│  Sign-up, sign-in,        │         │   credentials)            │
│  password reset, MFA      │         │                           │
└──────────────────────────┘         └──────────────────────────┘
```

| Component | Purpose |
| :--- | :--- |
| **User Pools (CUP)** | Sign-in functionality for app users; integrates with API Gateway & ALB |
| **Identity Pools (Federated Identity)** | Provides **temporary AWS credentials** so users can access AWS resources directly |

> **Cognito vs IAM:** Use Cognito when you have "hundreds/thousands/millions of mobile or web users" needing sign-up/sign-in, or need to "authenticate with SAML." IAM is for internal AWS access (employees, services) — not for managing large external user bases.

---

## Cognito User Pools (CUP) — Features

A **serverless user database** for web and mobile apps.

| Feature | Detail |
| :--- | :--- |
| Simple login | Username (or email) + password |
| Password reset | ✅ Built-in |
| Verification | Email & phone number verification |
| MFA | ✅ Multi-factor authentication supported |
| Federated identities | Sign in via Facebook, Google, SAML |

---

## CUP Integrations

### With API Gateway

![Cognito User Pools (CUP) Integrations](assets/Cognito%20User%20Pools%20(CUP)%20-%20Integrations.png)

> **Exam insight:** ALB-native Cognito authentication means you can add login functionality to a traditional EC2/ECS-backed web app **without writing any authentication code** — ALB handles it via listener rules.

---

## Cognito Identity Pools (Federated Identities)

Exchanges a login token for **temporary AWS credentials**.

### Supported Login Sources

* Cognito User Pools
* Social Identity Providers (Google, Facebook)
* SAML 2.0
* OpenID Connect

### Full Flow Diagram

![Cognito Identity Pools Diagram](assets/Cognito%20Identity%20Pools%20–%20Diagram.png)

### Permissions Model

| Aspect | Detail |
| :--- | :--- |
| Policy source | IAM policies applied to the temporary credentials are **defined in Cognito** |
| Customization | Can be scoped per-user via `user_id` for fine-grained access |
| Default roles | Separate default IAM roles exist for **authenticated** and **guest (unauthenticated)** users |

---

## Row-Level Security in DynamoDB via Cognito

Restrict each user to **only their own items** in a shared DynamoDB table using IAM policy conditions.

```json
{
  "Condition": {
    "ForAllValues:StringEquals": {
      "dynamodb:LeadingKeys": ["${cognito-identity.amazonaws.com:sub}"]
    }
  }
}
```

| Component | Role |
| :--- | :--- |
| `dynamodb:LeadingKeys` | Condition key restricting access to items matching the Partition Key |
| `${cognito-identity.amazonaws.com:sub}` | Dynamically resolves to the caller's unique Cognito Identity ID |

**Effect:** Every user, using the SAME IAM role, can only read/write DynamoDB items where the Partition Key equals their own identity — enforced at the AWS API level, no application-side filtering needed.

---

# 📐 Building a Serverless REST API — End-to-End Example

```
┌──────────┐    REST API    ┌───────────────┐   Proxy Request   ┌────────────┐   CRUD    ┌──────────────┐
│  Client   │ ─────────────▶ │  API Gateway   │ ─────────────────▶│   Lambda    │ ─────────▶│  DynamoDB     │
└──────────┘                └───────────────┘                    └────────────┘           └──────────────┘
     ▲                              │
     │                              ▼
     │                     ┌───────────────┐
     └─────────────────────│    Cognito     │
        (auth token)       │  User Pools    │
                            └───────────────┘
```

This is the **canonical serverless REST API pattern** for SAA-C03: API Gateway as the front door, Cognito for auth, Lambda for business logic, DynamoDB for storage.

---

# ⚠️ Exam Traps & Common Mistakes

> **Trap 1 — Lambda default networking.**
> A default Lambda function runs OUTSIDE your VPC and CANNOT reach private RDS/ElastiCache. You must explicitly configure VPC, Subnets, and Security Groups.

> **Trap 2 — VPC Lambda loses internet access.**
> Once placed in a VPC, Lambda loses default public internet access. Needs a **NAT Gateway** (internet) or **VPC Endpoint** (private access to S3/DynamoDB, etc.).

> **Trap 3 — RDS Proxy requires VPC.**
> RDS Proxy is never publicly accessible — the Lambda calling it must be deployed inside a VPC.

> **Trap 4 — RDS Event Notifications ≠ data change events.**
> These notify about the DB **instance** state (started/stopped), not row-level data changes. For data changes, use DB-triggered Lambda invocation or DynamoDB Streams.

> **Trap 5 — Reserved vs Provisioned Concurrency.**
> Reserved = guarantees/limits a quota (fixes the "one function starves others" problem). Provisioned = pre-warms instances (fixes the "cold start latency" problem). Different problems, different solutions.

> **Trap 6 — Lambda 15-minute hard limit.**
> If a workload needs more than 900 seconds, Lambda alone cannot do it — use Step Functions to orchestrate multiple invocations, or move to Fargate/EC2/Batch.

> **Trap 7 — CloudFront Functions have no network access.**
> If edge logic needs to call another AWS service or use the AWS SDK, only **Lambda@Edge** can do this — CloudFront Functions cannot.

> **Trap 8 — Lambda@Edge must be authored in us-east-1.**
> Regardless of where you want it to run globally, the function must be created in us-east-1 before CloudFront replicates it.

> **Trap 9 — DynamoDB Global Tables require Streams.**
> You cannot enable Global Tables without first enabling DynamoDB Streams — it's a hard prerequisite.

> **Trap 10 — DynamoDB restores always create a NEW table.**
> Neither PITR nor On-Demand Backup restores "in place" — both always produce a brand-new table.

> **Trap 11 — Edge-Optimized API Gateway certificate region.**
> Just like CloudFront, an Edge-Optimized API Gateway custom domain certificate must be in **us-east-1**, regardless of the API's actual region.

> **Trap 12 — DAX vs ElastiCache.**
> DAX is DynamoDB-specific (drop-in compatible with DynamoDB API, caches individual items/queries). ElastiCache is general-purpose (used for aggregated/computed results, requires application-level cache logic).

---

# 🧠 High-Frequency Exam Facts

* Lambda max execution time = **900 seconds (15 min)**; max memory = **10 GB**
* Lambda free tier = **1M requests + 400,000 GB-seconds/month**
* Default concurrency limit = **1,000 per region** (soft limit, can be increased)
* Sync invocation throttle = **HTTP 429**; async throttle = retries then **DLQ**
* Async retries use **exponential backoff**, up to **6 hours** total
* **Reserved Concurrency** = caps/guarantees a function's quota
* **Provisioned Concurrency** = pre-warms instances, eliminates cold starts
* **SnapStart** = free 10x faster starts for Java/Python/.NET via memory/disk snapshot
* Lambda is OUTSIDE your VPC by default — needs explicit VPC config to reach RDS
* VPC-attached Lambda needs **NAT Gateway** or **VPC Endpoints** for internet/AWS service access
* **RDS Proxy** must be paired with a **VPC-deployed** Lambda (never public)
* RDS Proxy reduces failover time by **66%**
* CloudFront Functions = JavaScript, <1ms, 2MB memory, Viewer events only, no network access
* Lambda@Edge = Node.js/Python, 5-10 sec, up to 10GB memory, all 4 trigger points, has network access
* Lambda@Edge must be authored in **us-east-1**
* DynamoDB max item size = **400 KB**
* DynamoDB Streams retention = **24 hours**; Kinesis-based = **1 year**
* DynamoDB Global Tables = **Active-Active**, requires **Streams enabled**
* DynamoDB PITR window = **35 days**; both backup types create a **new table** on restore
* DAX default cache TTL = **5 minutes**
* Edge-Optimized API Gateway cert must be in **us-east-1** (same rule as CloudFront)
* API Gateway Private endpoint requires an **Interface VPC Endpoint**
* Step Functions = orchestrates multi-step, long-running, branching workflows across Lambda and other services
* Cognito User Pools = authentication (who is this user); Cognito Identity Pools = authorization (temporary AWS credentials)
* DynamoDB row-level security uses `dynamodb:LeadingKeys` + `${cognito-identity.amazonaws.com:sub}`

---

# 📐 Architecture Pattern Quick Reference

| Requirement | Service / Pattern |
| :--- | :--- |
| Run code without managing servers | **AWS Lambda** |
| Function needs to reach private RDS | Lambda deployed in **VPC** with correct subnets/SGs |
| Function needs internet access from inside a VPC | **NAT Gateway** |
| High-traffic Lambda overwhelming database connections | **RDS Proxy** |
| Eliminate cold start latency | **Provisioned Concurrency** or **SnapStart** (Java/Python/.NET) |
| Prevent one function from starving others of concurrency | **Reserved Concurrency** |
| Workflow longer than 15 minutes / complex branching | **AWS Step Functions** |
| Lightweight edge logic (headers, redirects, cache keys) | **CloudFront Functions** |
| Edge logic needing AWS SDK / network access | **Lambda@Edge** |
| NoSQL database with millisecond latency at scale | **Amazon DynamoDB** |
| Cache DynamoDB reads with microsecond latency | **Amazon DAX** |
| React to DynamoDB row-level changes in real time | **DynamoDB Streams + Lambda** |
| Multi-region active-active NoSQL table | **DynamoDB Global Tables** |
| Auto-delete expired data | **DynamoDB TTL** |
| Analyze DynamoDB data with SQL-like queries | **Export to S3 (PITR) + Athena** |
| Expose Lambda as a REST/WebSocket API | **API Gateway** |
| API accessible only from within a VPC | **API Gateway Private Endpoint** |
| Global users needing low API latency | **API Gateway Edge-Optimized Endpoint** |
| Authenticate mobile/web app users at scale | **Cognito User Pools** |
| Give users temporary AWS credentials for direct S3/DynamoDB access | **Cognito Identity Pools** |
| Restrict users to their own DynamoDB rows | **Cognito Identity Pools + IAM `LeadingKeys` condition** |
| Add login to an ALB-fronted app with no custom code | **ALB + Cognito User Pools integration** |

---

# 📚 Conclusion

This guide covers the AWS serverless services most heavily tested in SAA-C03. The exam tests both **deep numeric limits** (15-minute timeout, 256KB/400KB size limits, concurrency numbers) and **architectural pattern recognition** (when to use Reserved vs Provisioned Concurrency, CloudFront Functions vs Lambda@Edge, DAX vs ElastiCache).

**Core decision framework:**

* Need **serverless compute**? → **Lambda** (under 15 min) or **Step Functions** (longer/complex workflows)
* Need to reach a **private VPC resource** from Lambda? → Configure **VPC access**, add **RDS Proxy** if DB connections spike
* Need to **eliminate cold starts**? → **Provisioned Concurrency** (any language, costs extra) or **SnapStart** (Java/Python/.NET, free)
* Need **edge logic**? → **CloudFront Functions** (lightweight, no network) or **Lambda@Edge** (complex, needs network/SDK)
* Need a **NoSQL database** at massive scale? → **DynamoDB**, add **DAX** for read caching
* Need **multi-region active-active data**? → **DynamoDB Global Tables**
* Need to **expose an API**? → **API Gateway**, paired with **Lambda** or direct AWS service integration
* Need **user authentication**? → **Cognito User Pools**
* Need users to access **AWS resources directly**? → **Cognito Identity Pools**

---

<div align="center">

### ☁️ AWS SAA-C03 Serverless Domain Reference

**AWS Lambda • Amazon DynamoDB • API Gateway • Step Functions • Amazon Cognito • CloudFront Functions • Lambda@Edge**

</div>