# ☁️ AWS Messaging & Integration Services — SAA-C03 Study Reference Guide

<div align="center">

![AWS](https://img.shields.io/badge/AWS-SAA--C03-orange?style=for-the-badge&logo=amazonaws)
![SQS](https://img.shields.io/badge/SQS-Queue%20Model-blue?style=for-the-badge)
![SNS](https://img.shields.io/badge/SNS-Pub%2FSub-success?style=for-the-badge)
![Kinesis](https://img.shields.io/badge/Kinesis-Real--Time%20Streaming-yellow?style=for-the-badge)

### 📨 SQS • SNS • Kinesis • Data Firehose • Amazon MQ

*Comprehensive SAA-C03 study reference covering application decoupling, messaging patterns, real-time streaming, and managed message brokers — with architecture diagrams, exam traps, and high-frequency facts.*

</div>


---

# 📖 Why Decouple Applications?

In a **tightly coupled** (synchronous) architecture, services call each other directly. If one service is overwhelmed or fails, the failure cascades through the entire system.

```
❌ Tightly Coupled (Synchronous)
────────────────────────────────
Web App ──direct call──▶ Video Encoder ──direct call──▶ Database
         ← if encoder crashes, entire flow fails →
```

```
✅ Loosely Coupled (Asynchronous / Decoupled)
─────────────────────────────────────────────
Web App ──▶ [Queue / Topic / Stream] ──▶ Video Encoder ──▶ Database
           ← failure isolated; each tier scales independently →
```
![Section Introduction](assets/Section%20Introduction.png)

**Real-world problem:** Your system usually encodes 10 videos/day. During a promotional event, suddenly 1000 videos arrive at once. A synchronous system crashes. A decoupled system queues the work and processes it at the encoder's pace.

**AWS decoupling services:**

| Service | Model | Best For |
| :--- | :--- | :--- |
| **Amazon SQS** | Queue (pull) | Decouple producers and consumers |
| **Amazon SNS** | Pub/Sub (push) | Fan-out notifications to many receivers |
| **Amazon Kinesis** | Real-time stream | Large-scale streaming data and analytics |
| **Amazon MQ** | Message broker | Migrate on-premises broker workloads |

---

# 📬 Amazon Simple Queue Service (SQS)

## What is a Queue?

A queue is a **temporary holding area** for messages. Producers push messages in; consumers pull messages out and process them independently.

```
Producer(s)                    Queue                    Consumer(s)
───────────                 ──────────                 ────────────
Web Server  ──SendMessage──▶ [msg][msg][msg] ──Poll──▶  EC2 Worker
Lambda      ──SendMessage──▶ [msg][msg]      ──Poll──▶  Lambda
EC2 App     ──SendMessage──▶ [msg]           ──Poll──▶  ECS Task
```

![Amazon SQS What's a queue](assets/Amazon%20SQS%20What's%20a%20queue.png)

---

## SQS Standard Queue

The **default** SQS queue type. Built for maximum throughput.

### Core Attributes

| Attribute | Value |
| :--- | :--- |
| Throughput | Unlimited |
| Message count | Unlimited |
| Default retention | **4 days** |
| Maximum retention | **14 days** |
| Latency | < 10 ms (publish and receive) |
| Max message size | **256 KB** |
| Delivery guarantee | **At-least-once** (may get duplicates) |
| Ordering | **Best-effort** (not guaranteed) |

> **Note on message size:** The SAA-C03 course notes say 1,024 KB but the AWS official limit for SQS is **256 KB**. Always go with **256 KB** in exam answers.

---

## SQS — Message Lifecycle

```
1. Producer sends message via SDK (SendMessage API)
        │
        ▼
2. Message stored in SQS queue (persisted until deleted)
        │
        ▼
3. Consumer polls queue (ReceiveMessage API — up to 10 msgs at once)
        │
        ▼
4. Message becomes INVISIBLE to other consumers (Visibility Timeout)
        │
        ├── Consumer processes successfully ──▶ DeleteMessage API ──▶ Message gone ✅
        │
                └── Consumer crashes / timeout expires ──▶ Message becomes VISIBLE again ──▶ Reprocessed ⚠️
```

![SQS – Consuming Message](assets/SQS%20–%20Consuming%20Message.png)

---

## SQS — Visibility Timeout

When a consumer polls a message, it becomes **invisible** to all other consumers for the duration of the **Visibility Timeout**.

| Scenario | Result |
| :--- | :--- |
| Consumer processes and deletes before timeout | ✅ Success — message deleted |
| Consumer crashes before timeout expires | ⚠️ Message reappears — processed again (duplicate) |
| Consumer needs more time | Call `ChangeMessageVisibility` API to extend timeout |

| Timeout Setting | Risk |
| :--- | :--- |
| Too high (e.g., 12 hours) | If consumer crashes, message is stuck invisible for hours |
| Too low (e.g., 5 seconds) | Fast consumer restarts → duplicates |
| **Default: 30 seconds** | Balanced starting point |

> **Key concept:** Visibility Timeout is what prevents two consumers from processing the same message simultaneously — it is NOT a deletion mechanism.

![SQS – Message Visibility Timeout](assets/SQS%20–%20Message%20Visibility%20Timeout.png)

---

## SQS — Long Polling vs Short Polling

| Feature | Short Polling | Long Polling |
| :--- | :--- | :--- |
| Behavior | Returns immediately even if queue is empty | Waits for messages to arrive (1–20 sec) |
| API calls | Many (wasteful when queue is empty) | Fewer (efficient) |
| Latency | Low | Slightly higher but acceptable |
| Cost | Higher (more API calls billed) | **Lower** |
| Recommendation | ❌ Avoid | ✅ **Preferred** |

Enable Long Polling:
* At queue level in console settings
* At API level using `WaitTimeSeconds` parameter (max `20`)

---

## SQS — Auto Scaling Integration

SQS integrates natively with EC2 Auto Scaling to dynamically scale consumers based on queue depth.

```
SQS Queue
    │
    │  CloudWatch Metric: ApproximateNumberOfMessages
    ▼
CloudWatch Alarm
    │
    │  (Queue depth > threshold)
    ▼
Auto Scaling Group ──▶ Launch more EC2 consumer instances
    │
    │  (Queue depth < threshold)
    ▼
Auto Scaling Group ──▶ Terminate excess EC2 consumer instances
```

**Metric to watch:** `ApproximateNumberOfMessages`
This tells you how many messages are waiting to be processed — the primary signal for scaling consumer capacity.

![SQS with Auto Scaling Group (ASG)](assets/SQS%20with%20Auto%20Scaling%20Group%20(ASG).png)

---

## SQS — Decoupling Application Tiers

Classic 3-tier architecture with SQS between layers:

```
┌──────────────┐     SendMessage      ┌─────────────────┐     ReceiveMessages     ┌──────────────────┐
│  Front-End   │ ──────────────────▶  │   SQS Queue     │ ──────────────────────▶ │  Back-End Worker │
│  Web App     │                      │  (infinitely    │                         │  (Video Encoder, │
│  (ASG)       │                      │   scalable)     │                         │   Data Processor)│
└──────────────┘                      └─────────────────┘                         └──────────────────┘
       ↑ scales independently                                                              ↑ scales independently
```

**Benefits:**
* Front-end doesn't wait for back-end to finish
* Back-end processes at its own pace
* Both tiers scale independently based on their own metrics
* If back-end goes down, messages accumulate safely in the queue — no data loss

![SQS to decouple between application tiers](assets/SQS%20to%20decouple%20between%20application%20tiers.png)

---

## SQS — Security

| Layer | Mechanism |
| :--- | :--- |
| In-transit encryption | HTTPS API (always enabled) |
| At-rest encryption | **AWS KMS keys** |
| Client-side encryption | Optional — application manages keys |
| API access control | **IAM policies** attached to users/roles |
| Queue-level access | **SQS Access Policies** (resource-based, similar to S3 bucket policies) |

**SQS Access Policies are used when:**
* Granting **cross-account access** to a queue
* Allowing **AWS services** (SNS, S3 events) to write to the queue directly

---

## SQS FIFO Queue

FIFO = **First In, First Out** — strict ordering guaranteed.

### FIFO vs Standard

| Feature | Standard | FIFO |
| :--- | :--- | :--- |
| Ordering | Best-effort | ✅ Strict (FIFO) |
| Duplicates | Possible | ✅ Exactly-once (with Deduplication ID) |
| Throughput | Unlimited | 300 msg/s (3,000 msg/s with batching) |
| Use case | High throughput, order doesn't matter | Order-critical workflows |

### FIFO Key Concepts

**Message Group ID** — mandatory parameter that groups related messages. All messages with the same Group ID are processed in strict order by one consumer.

```
Group ID: "order-001"  →  [msg1] → [msg2] → [msg3]  →  processed in this exact order
Group ID: "order-002"  →  [msgA] → [msgB]             →  processed in this exact order
(two groups processed in parallel — each group is ordered internally)
```

**Deduplication ID** — prevents duplicate messages from being delivered within a 5-minute deduplication interval. If two messages have the same Deduplication ID, the second one is silently discarded.

> **FIFO queue naming:** FIFO queue names must end with `.fifo` suffix — e.g., `MyQueue.fifo`

![SQS – FIFO Queue](assets/Amazon%20SQS%20–%20FIFO%20Queue.png)

---

# 📢 Amazon Simple Notification Service (SNS)

## What is SNS?

SNS implements the **Pub/Sub (Publish/Subscribe)** pattern. One message published to a topic is delivered to **all subscribers** simultaneously — no consumer needs to poll.

```
                          ┌──────────────────┐
                          │   SNS Topic      │
Event Source ──Publish──▶ │  (e.g., Orders)  │
                          └────────┬─────────┘
                                   │ Fan-Out
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
               SQS Queue      Lambda Fn      Email Sub
              (fulfillment)  (analytics)   (admin alert)
```

### SNS Limits

| Limit | Value |
| :--- | :--- |
| Subscriptions per topic | **12,500,000** |
| Topics per account | **100,000** |
| Message size | **256 KB** |
| Message persistence | ❌ Not persisted — if delivery fails, message is lost |

![Amazon SNS](assets/Amazon%20SNS.png)

---

## SNS Subscribers (Destinations)

SNS can deliver messages to:

| Subscriber Type | Notes |
| :--- | :--- |
| **Amazon SQS** | Most common — adds persistence and retry |
| **AWS Lambda** | Serverless processing per notification |
| **Kinesis Data Firehose** | Stream notifications to S3, Redshift, etc. |
| **Email** | Plain text or JSON notifications |
| **SMS & Mobile Push** | GCM (Google), APNS (Apple), Amazon ADM |
| **HTTP/HTTPS Endpoints** | Webhooks to external services |

![Amazon SNS to Subscribers](assets/Amazon%20SNS%20to%20Subscribers.png)

---

## SNS — AWS Service Integrations

Many AWS services publish events directly to SNS without any custom code:

* CloudWatch Alarms → alert notifications
* S3 Events → object created/deleted notifications
* AWS Budgets → cost threshold alerts
* RDS Events → database state changes
* Auto Scaling → scale-out/scale-in notifications
* DynamoDB → table events
* CloudFormation → stack state changes
* Lambda → function execution events

![SNS integrates with a lot of AWS services](assets/SNS%20integrates%20with%20a%20lot%20of%20AWS%20services.png)

---

## SNS — Security

Identical security model to SQS:

| Layer | Mechanism |
| :--- | :--- |
| In-transit encryption | HTTPS API |
| At-rest encryption | AWS KMS keys |
| API access control | IAM policies |
| Topic-level access | **SNS Access Policies** (resource-based) |

SNS Access Policies needed for:
* Cross-account topic publishing
* Allowing S3 or other AWS services to publish to an SNS topic

---

## SNS + SQS — Fan-Out Pattern

**The most important SNS pattern for the exam.**

Instead of sending to one destination, publish once to SNS and let it fan out to multiple SQS queues simultaneously.

```
              ┌──────────────────────────────────────────┐
              │             SNS Topic                    │
S3 Event ──▶  │          "new-order"                     │
              └────────┬──────────────┬──────────────────┘
                       │              │              │
                       ▼              ▼              ▼
                  SQS Queue       SQS Queue      Lambda
                 (fulfillment)   (analytics)   (fraud check)
```

**Benefits:**
* **Fully decoupled** — publisher doesn't know about consumers
* **No data loss** — SQS persists messages even if consumer is down
* **Extensible** — add more subscribers without changing the publisher
* **Cross-region** — SQS queues in other AWS regions can subscribe

**Critical requirement:** The SQS queue's **Access Policy** must explicitly allow the SNS topic to write (`sns:SendMessage` from the SNS topic ARN).

![SNS + SQS Fan Out](assets/SNS%20+%20SQS%20Fan%20Out.png)

---

## SNS Fan-Out — S3 Events to Multiple Destinations

**Problem:** S3 only allows **one event rule per event type + prefix combination**.

```
❌ Cannot do this directly:
S3 (images/*, ObjectCreated) ──▶ SQS Queue 1
S3 (images/*, ObjectCreated) ──▶ SQS Queue 2   ← Second rule FAILS
```

```
✅ Solution — Fan-Out via SNS:
S3 (images/*, ObjectCreated) ──▶ SNS Topic ──▶ SQS Queue 1
                                            ──▶ SQS Queue 2
                                            ──▶ Lambda Function
```

---

## SNS — Message Filtering

By default, every subscriber receives **every message** published to the topic.

**Message Filtering** lets you attach a JSON filter policy to each subscription, so subscribers only receive messages matching their filter.

```json
// Filter policy for "Fulfilled Orders" subscription
{
  "status": ["Placed", "Shipped"]
}

// Filter policy for "Cancelled Orders" subscription
{
  "status": ["Cancelled", "Declined"]
}
```

```
SNS Topic (all orders)
    │
    ├── SQS Queue A (filter: status=Placed,Shipped)   → receives only active orders
    ├── SQS Queue B (filter: status=Cancelled)         → receives only cancellations
    └── Email Sub   (no filter policy)                 → receives ALL messages
```

![SNS – Message Filtering](assets/SNS%20–%20Message%20Filtering.png)

---

## SNS FIFO Topic

Just like SQS FIFO, SNS FIFO topics guarantee ordering and deduplication.

| Feature | SNS Standard | SNS FIFO |
| :--- | :--- | :--- |
| Message ordering | No | ✅ Yes (by Message Group ID) |
| Deduplication | No | ✅ Yes (Deduplication ID or Content-Based) |
| Throughput | High | Limited (same as SQS FIFO) |
| Subscribers | SQS, Lambda, HTTP, Email, SMS | **SQS queues only** |

**SNS FIFO + SQS FIFO Fan-Out** = ordered, deduplicated, fan-out delivery to multiple queues.

![SNS – FIFO Topic](assets/Amazon%20SNS%20–%20FIFO%20Topic.png)

![SNS FIFO + SQS FIFO Fan Out](assets/SNS%20FIFO%20+%20SQS%20FIFO%20Fan%20Out.png)

---

# 🌊 Amazon Kinesis Data Streams

## What is Kinesis Data Streams?

Kinesis Data Streams is a **real-time data streaming service** designed to ingest, buffer, and process large volumes of streaming data continuously. Think of it as a high-speed conveyor belt that retains all the data that passed through it for up to 365 days.

```
Producers                    Kinesis Data Streams               Consumers
─────────                    ────────────────────               ─────────
Web Servers  ──▶ Shard 1 ──▶ [data][data][data]  ──▶ EC2 / Lambda / KDA
IoT Devices  ──▶ Shard 2 ──▶ [data][data][data]  ──▶ EC2 / Lambda / KDA
Mobile Apps  ──▶ Shard 3 ──▶ [data][data][data]  ──▶ EC2 / Lambda / KDA
```

![Amazon Kinesis Data Streams](assets/Amazon%20Kinesis%20Data%20Streams.png)

---

## Kinesis Data Streams — Key Features

| Feature | Detail |
| :--- | :--- |
| Data retention | Up to **365 days** |
| Replay capability | ✅ Yes — consumers can re-read old data |
| Deletion | ❌ Cannot delete data — expires automatically after retention period |
| Max record size | **1 MB** |
| Ordering | Guaranteed within the **same Partition Key / Shard** |
| At-rest encryption | AWS KMS |
| In-transit encryption | HTTPS |

---

## Kinesis Data Streams — Shards

A **shard** is the base throughput unit of Kinesis Data Streams. More shards = more throughput.

| Direction | Per Shard Throughput |
| :--- | :--- |
| **Ingest (write)** | 1 MB/s OR 1,000 records/second |
| **Read (Standard)** | 2 MB/s shared across all consumers |
| **Read (Enhanced Fan-Out)** | 2 MB/s **per consumer** (push model) |

**Partition Key** — producers assign a Partition Key to each record. Records with the same Partition Key always go to the same shard → guaranteed ordering for that key.

```
Partition Key: "user-123"  → always goes to Shard 2 → ordered for that user
Partition Key: "user-456"  → always goes to Shard 1 → ordered for that user
```

---

## Kinesis — Capacity Modes

| Feature | Provisioned Mode | On-Demand Mode |
| :--- | :--- | :--- |
| Capacity management | Manual — choose shard count | Automatic |
| Default capacity | Based on shards × 1 MB/s | 4 MB/s in / 4,000 records per second |
| Auto-scaling | ❌ Manual resharding | ✅ Scales based on peak from last 30 days |
| Pricing | Per shard provisioned per hour | Per stream per hour + data in/out per GB |
| Best for | Predictable, steady workloads | Unpredictable or variable workloads |

---

## Kinesis Producers & Consumers

### Producers
* **AWS SDK** — direct API (`PutRecord`, `PutRecords`)
* **Kinesis Producer Library (KPL)** — batching, compression, retries, high throughput
* **Kinesis Agent** — log file shipping from servers
* **AWS IoT, CloudWatch Logs, etc.**

### Consumers
* **AWS SDK** — direct `GetRecords` polling (Standard — 2 MB/s shared per shard)
* **Kinesis Client Library (KCL)** — manages shard assignment, checkpointing across multiple consumers
* **AWS Lambda** — event-driven, serverless processing
* **Kinesis Data Analytics** — real-time SQL or Apache Flink processing
* **Enhanced Fan-Out** — each registered consumer gets dedicated 2 MB/s push per shard

---

# 🔥 Amazon Data Firehose (formerly Kinesis Data Firehose)

## What is Data Firehose?

Data Firehose is a **fully managed, serverless** service for loading streaming data into storage and analytics destinations. Unlike Kinesis Data Streams, Firehose is not for real-time processing — it buffers data and delivers in **near real-time** batches.

```
Producers ──▶ Data Firehose ──[optional Lambda transform]──▶ Destination
```

---

## Data Firehose — Destinations

| Category | Destinations |
| :--- | :--- |
| **AWS Native** | Amazon S3, Amazon Redshift, Amazon OpenSearch, CloudWatch Logs |
| **Third-Party** | Splunk, Datadog, New Relic, MongoDB |
| **Custom** | Any HTTP endpoint |

---

## Data Firehose — Key Features

| Feature | Detail |
| :--- | :--- |
| Management | Fully managed, serverless, auto-scaling |
| Latency | **Near real-time** (buffering based on size or time) |
| Data storage | ❌ No storage — Firehose does not retain data |
| Replay | ❌ No replay capability |
| Formats | CSV, JSON, Parquet, Avro, Raw Text, Binary |
| Compression | gzip, snappy, zip |
| Transformation | Optional **Lambda function** per delivery stream |
| Failed records | Sent to a separate **S3 backup bucket** |
| Pricing | Pay per GB transferred |

---

## Data Firehose — Processing Flow

```
Producer (SDK / KPL / IoT)
        │
        ▼ (records up to 1 MB)
 Kinesis Data Firehose
        │
        ├── [Optional] Lambda transformation
        │      (e.g., CSV → JSON, filter fields, enrich records)
        │
        ▼ (buffered by size [1–128 MB] or time [60–900 sec])
 Batch write to destination
        │
        ├──▶ Amazon S3 (primary destination)
        ├──▶ Amazon Redshift (via S3 COPY command)
        ├──▶ Amazon OpenSearch
        └──▶ Third-party (Splunk, Datadog, etc.)
                │
                └──▶ Failed records ──▶ S3 Backup Bucket
```

---

## Kinesis Data Streams vs Data Firehose

| Feature | Kinesis Data Streams | Amazon Data Firehose |
| :--- | :--- | :--- |
| Purpose | Collect and process streaming data | Load streaming data into destinations |
| Latency | **Real-time** (milliseconds) | **Near real-time** (seconds to minutes) |
| Data storage | Up to **365 days** | ❌ No storage |
| Replay | ✅ Yes | ❌ No |
| Consumer code | Required (SDK, KCL, Lambda) | ✅ Fully managed — no consumer code |
| Capacity | Provisioned shards or On-Demand | Auto-scaling |
| Transformation | External (Lambda triggered separately) | ✅ Built-in Lambda transform |
| Destinations | Custom consumers | S3, Redshift, OpenSearch, 3rd party |
| Use case | Custom real-time processing | ETL to storage/analytics |

> **Memory trick:** Streams = you build the consumer. Firehose = AWS delivers for you.

---

# 📊 SQS vs SNS vs Kinesis — Master Comparison

| Feature | SQS | SNS | Kinesis Data Streams |
| :--- | :--- | :--- | :--- |
| **Model** | Queue (pull) | Pub/Sub (push) | Stream (pull/push) |
| **Consumers** | One consumer per message | All subscribers get every message | Many independent consumers |
| **Persistence** | Up to **14 days** | ❌ Not persisted | Up to **365 days** |
| **Replay** | ❌ No | ❌ No | ✅ Yes |
| **Ordering** | FIFO queues only | FIFO topics only | Per shard (Partition Key) |
| **Max message size** | **256 KB** | **256 KB** | **1 MB** |
| **Throughput** | Unlimited (Standard) | Unlimited | Per shard (1 MB/s in) |
| **Scaling** | Automatic | Automatic | Manual (shards) or On-Demand |
| **Primary use** | Decouple services | Fan-out notifications | Real-time big data / analytics |
| **Typical consumers** | EC2, Lambda | SQS, Lambda, HTTP, Email, SMS | EC2, Lambda, KDA, KDF |

---

# 🐇 Amazon MQ

## What is Amazon MQ?

Amazon MQ is a **managed message broker service** for Apache ActiveMQ and RabbitMQ. It exists to help companies migrate **existing on-premises applications** to AWS without rewriting their messaging code.

## The Problem It Solves

On-premises applications are often built on open messaging protocols:

| Protocol | Full Name |
| :--- | :--- |
| **MQTT** | Message Queuing Telemetry Transport (IoT, lightweight) |
| **AMQP** | Advanced Message Queuing Protocol (RabbitMQ default) |
| **STOMP** | Simple Text Oriented Messaging Protocol |
| **OpenWire** | ActiveMQ native protocol |
| **WSS** | WebSocket Secure |

SQS and SNS use **proprietary AWS APIs** — existing applications would need full rewrites to use them.

**Amazon MQ uses standard protocols** — no application code changes required.

---

## Amazon MQ vs SQS/SNS

| Feature | Amazon MQ | SQS / SNS |
| :--- | :--- | :--- |
| Protocol | MQTT, AMQP, STOMP, OpenWire | AWS proprietary |
| Scale | Limited (runs on servers) | ✅ Unlimited scale |
| Serverless | ❌ No (server-based) | ✅ Yes |
| Migration from on-prem | ✅ Easy (same protocol) | Requires code changes |
| Queue features | ✅ Yes (like SQS) | SQS only |
| Topic features | ✅ Yes (like SNS) | SNS only |
| Multi-AZ | ✅ Active/Standby | ✅ Built-in |

> **When to choose Amazon MQ:** Any scenario mentioning migration from on-premises broker, or mentioning MQTT, AMQP, STOMP, RabbitMQ, or ActiveMQ protocols.

> **When to choose SQS/SNS:** New cloud-native applications where maximum scale and serverless are priorities.

---

## Amazon MQ — High Availability Architecture

```
┌──────────────────────────────────────────┐
│              us-east-1 Region            │
│                                          │
│  ┌─────────────────┐                     │
│  │  Availability   │  Active Broker      │
│  │   Zone A        │  ← serves traffic   │
│  │  [MQ Broker]    │                     │
│  └────────┬────────┘                     │
│           │                              │
│           │  Amazon EFS (shared storage) │
│           │  ← both brokers share same   │
│           │    message store             │
│           │                              │
│  ┌────────┴────────┐                     │
│  │  Availability   │  Standby Broker     │
│  │   Zone B        │  ← takes over       │
│  │  [MQ Broker]    │    if Active fails  │
│  └─────────────────┘                     │
└──────────────────────────────────────────┘
```

* **Active broker** handles all traffic
* **Standby broker** stays synchronized via shared Amazon EFS storage
* On Active failure → automatic failover to Standby
* Clients reconnect to the same endpoint (DNS-based failover)

![Amazon MQ – High Availability](assets/Amazon%20MQ%20–%20High%20Availability.png)

---

# ⚠️ Exam Traps & Common Mistakes

> **Trap 1 — SQS message size:**
> The exam expects **256 KB** as the SQS message size limit — not 1,024 KB.

> **Trap 2 — SNS message persistence:**
> SNS does **NOT** persist messages. If a subscriber is unavailable when a message is published, that message is **lost**. The fix is to pair SNS with SQS (Fan-Out pattern) so SQS holds the message until the consumer is ready.

> **Trap 3 — FIFO throughput:**
> SQS FIFO is limited to **300 messages/second** without batching and **3,000 messages/second** with batching. Do not recommend FIFO for extremely high-throughput scenarios.

> **Trap 4 — Kinesis vs SQS for ordering:**
> Both support ordering, but differently. SQS FIFO is ordered globally within a group. Kinesis is ordered **per shard** using Partition Keys. Kinesis is the right answer for high-throughput ordered streaming at scale.

> **Trap 5 — Visibility Timeout is not deletion:**
> Making a message invisible is not the same as deleting it. The consumer **must** call `DeleteMessage` explicitly after processing. If it doesn't, the message reappears after the timeout.

> **Trap 6 — Kinesis replay vs SQS:**
> SQS has **no replay** — once a message is deleted, it's gone. Kinesis retains all data for up to 365 days and **multiple consumers** can independently read the same data at different positions.

> **Trap 7 — Amazon MQ vs SQS/SNS:**
> If the scenario mentions an **existing on-premises application** using MQTT, AMQP, or RabbitMQ → choose **Amazon MQ**. If it's a new cloud-native app → choose **SQS/SNS**.

> **Trap 8 — Data Firehose is NOT real-time:**
> Firehose buffers data and delivers in batches — it is **near real-time** (seconds to minutes delay). If the requirement says "real-time millisecond processing" → use **Kinesis Data Streams**.

> **Trap 9 — S3 fan-out limitation:**
> S3 only supports **one event rule per event type + prefix**. To send to multiple destinations, use S3 → SNS → multiple SQS queues (Fan-Out pattern).

> **Trap 10 — Long Polling WaitTimeSeconds:**
> Maximum value is **20 seconds**. Long Polling is almost always preferred over Short Polling for efficiency.

---

# 🧠 High-Frequency Exam Facts

* SQS Standard = **at-least-once delivery**, best-effort ordering
* SQS FIFO = **exactly-once delivery**, strict ordering, limited throughput
* SQS default visibility timeout = **30 seconds** (max 12 hours)
* SQS max retention = **14 days**, default = **4 days**
* SQS max message size = **256 KB**
* SQS Long Polling max wait = **20 seconds** (`WaitTimeSeconds`)
* SQS CloudWatch metric for scaling = `ApproximateNumberOfMessages`
* FIFO queue name must end with **`.fifo`**
* SNS = **push** model; SQS = **pull** model
* SNS max subscriptions per topic = **12,500,000**
* SNS does **NOT** persist messages — pairing with SQS adds persistence
* SNS FIFO subscribers can **only be SQS queues**
* Fan-Out requires SQS queue **Access Policy** to allow SNS to write
* Kinesis max record size = **1 MB**
* Kinesis retention = up to **365 days**
* Kinesis **Standard** consumer = 2 MB/s shared per shard (pull)
* Kinesis **Enhanced Fan-Out** = 2 MB/s **per consumer** per shard (push)
* Kinesis ordering guaranteed per **Partition Key → same Shard**
* Data Firehose = **near real-time**, no replay, no storage
* Data Firehose failed records → **S3 backup bucket**
* Amazon MQ = for **existing apps** using MQTT, AMQP, STOMP, RabbitMQ, ActiveMQ
* Amazon MQ HA = **Active/Standby** across two AZs using shared **Amazon EFS**

---

# 📐 Architecture Pattern Quick Reference

| Requirement | Pattern |
| :--- | :--- |
| Decouple two microservices | SQS between them |
| Fan out one event to many consumers | SNS → multiple SQS queues |
| S3 event to multiple destinations | S3 → SNS → SQS Fan-Out |
| Ordered processing of financial transactions | SQS FIFO or SNS FIFO |
| Real-time clickstream / log analytics | Kinesis Data Streams |
| Load streaming data into S3 / Redshift | Kinesis Data Firehose |
| Ordered + deduplicated fan-out | SNS FIFO → SQS FIFO queues |
| Migrate on-premises RabbitMQ / ActiveMQ | Amazon MQ |
| Scale EC2 consumers with queue depth | SQS + CloudWatch + ASG |
| Persist SNS messages for retry | SNS + SQS (Fan-Out) |
| Real-time transform before storing | Kinesis Data Streams → Lambda → S3 |
| ETL pipeline with managed delivery | Data Firehose with Lambda transform |

---

# 📚 Conclusion

This guide covers the AWS messaging and integration services most commonly tested in SAA-C03. The exam primarily tests **pattern recognition** — understanding which service to choose for a given architectural requirement.

**Core decision framework:**

* Need to **decouple** two services asynchronously? → **SQS**
* Need to **notify many receivers** at once? → **SNS**
* Need **real-time streaming** with replay? → **Kinesis Data Streams**
* Need **managed ETL delivery** to S3/Redshift? → **Data Firehose**
* Need to **migrate existing broker** (RabbitMQ/ActiveMQ)? → **Amazon MQ**
* Need **ordered + deduplicated** queue? → **SQS FIFO** or **SNS FIFO**
* Need **fan-out + persistence**? → **SNS + SQS**

---

<div align="center">

### ☁️ AWS SAA-C03 Messaging & Integration Reference

**SQS • SNS • Kinesis Data Streams • Data Firehose • Amazon MQ • Decoupling • Event-Driven Architecture**

</div>