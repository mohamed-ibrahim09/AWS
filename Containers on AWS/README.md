# ☁️ AWS Container Services — SAA-C03 Study Reference Guide

<div align="center">

![AWS](https://img.shields.io/badge/AWS-SAA--C03-orange?style=for-the-badge&logo=amazonaws)
![Docker](https://img.shields.io/badge/Docker-Containers-blue?style=for-the-badge&logo=docker)
![ECS](https://img.shields.io/badge/ECS-Container%20Orchestration-success?style=for-the-badge)
![EKS](https://img.shields.io/badge/EKS-Kubernetes-yellow?style=for-the-badge&logo=kubernetes)

### 🐳 Docker • Amazon ECS • Amazon EKS • ECR • App Runner • App2Container

*Comprehensive SAA-C03 study reference covering containerization fundamentals, AWS container orchestration, serverless containers, and legacy app modernization — with architecture diagrams, exam traps, and high-frequency facts.*

</div>

---

# 📖 Overview

This guide covers the **Containers domain** of the SAA-C03 exam — one of the most heavily scenario-tested areas. You'll learn:

* **Docker fundamentals** — what containers are and why they matter
* **Amazon ECS** — AWS-native container orchestration (EC2 and Fargate)
* **Amazon EKS** — managed Kubernetes on AWS
* **Amazon ECR** — container image storage
* **AWS App Runner** — fully managed container/source deployment
* **AWS App2Container** — legacy app containerization tool

---

# 🐳 What is Docker?

## Definition

Docker is a software platform that packages applications into **containers** — lightweight, portable units that include everything an app needs to run (code, runtime, libraries, dependencies).

## Why Containers?

| Benefit | Explanation |
| :--- | :--- |
| Runs anywhere | Same container behaves identically on laptop, EC2, or any cloud |
| No compatibility issues | Dependencies are bundled inside the container |
| Predictable behavior | "Works on my machine" problem eliminated |
| Less operational work | No per-environment configuration |
| Language/OS agnostic | Works with any language, any OS, any technology stack |

## Common Use Cases

* **Microservices architecture** — each service runs in its own isolated container
* **Lift-and-shift migrations** — move on-premises apps to AWS without rewriting them

---

## Docker on a Single Host

```
┌─────────────────────────────────────────────────────┐
│              EC2 Instance (Host OS)                 │
│                                                      │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐          │
│  │  Java    │   │ Node.js  │   │  MySQL   │          │
│  │ Container│   │Container │   │Container │          │
│  └──────────┘   └──────────┘   └──────────┘          │
│         each container is isolated,                  │
│         but all share the same host OS kernel        │
└─────────────────────────────────────────────────────┘
```

Multiple isolated containers run side-by-side on one server — each with its own filesystem, processes, and network namespace, but sharing the underlying kernel.

![Getting Started with Docker](assets/Getting%20Started%20with%20Docker.png)

---

## Docker vs. Virtual Machines

This is one of the most frequently tested conceptual comparisons in the exam.

![Docker vs. Virtual Machines](assets/Docker%20vs.%20Virtual%20Machines.png)

| Aspect | Virtual Machines | Docker Containers |
| :--- | :--- | :--- |
| OS per instance | Full Guest OS each | Shares Host OS kernel |
| Startup time | Minutes | Seconds |
| Resource overhead | High | Low |
| Isolation level | Strong (hardware-level) | Process-level |
| Density per host | Lower | Higher |

> **Exam framing:** Docker is "sort of" virtualization but is NOT the same as VMs — it virtualizes at the OS/process level, not the hardware level.

---

## Docker Repositories

Docker images are stored in **repositories** so they can be pulled and run anywhere.

| Repository | Type | Notes |
| :--- | :--- | :--- |
| **Docker Hub** | Public | `hub.docker.com` — base images for OS/technologies (Ubuntu, MySQL, etc.) |
| **Amazon ECR** | Private | AWS-native, IAM-secured, integrated with ECS/EKS |
| **Amazon ECR Public Gallery** | Public | `gallery.ecr.aws` — public AWS-hosted images |

---

## Docker Workflow — From Code to Container

```
1. Write a Dockerfile
   ┌─────────────────────────────┐
   │ FROM ubuntu:18.04            │  ← base image
   │ COPY . /app                  │  ← copy app code
   │ RUN make /app                │  ← build step
   │ CMD python3 /app/app.py      │  ← startup command
   └─────────────────────────────┘
        │
        ▼  docker build
2. Docker Image (a built, reusable template)
        │
        ▼  docker run
3. Docker Container (a running instance of the image)
        │
        ▼  docker push / docker pull
4. Docker Repository (Docker Hub or Amazon ECR)
   ← image stored and shared from here
```

| Step | Command | Result |
| :--- | :--- | :--- |
| Define | Write `Dockerfile` | Build instructions |
| Build | `docker build` | Docker **Image** (immutable template) |
| Run | `docker run` | Docker **Container** (live running instance) |
| Share | `docker push` / `docker pull` | Image stored/retrieved from a repository |

---

# 🛠️ AWS Container Services Landscape

| Service | Role |
| :--- | :--- |
| **Amazon ECS** | AWS-native container orchestration |
| **Amazon EKS** | Managed Kubernetes orchestration |
| **AWS Fargate** | Serverless compute engine for ECS *and* EKS |
| **Amazon ECR** | Container image registry/storage |

```
                  ┌─────────────────────────────┐
                  │      Amazon ECR             │
                  │   (stores Docker images)    │
                  └──────────┬──────────────────┘
                             │ pulls image
              ┌──────────────┴───────────────┐
              ▼                              ▼
      ┌───────────────┐              ┌───────────────┐
      │  Amazon ECS    │              │  Amazon EKS   │
      │ (AWS-native    │              │ (Kubernetes)  │
      │  orchestrator) │              │               │
      └───────┬────────┘              └───────┬───────┘
              │                               │
       ┌──────┴──────┐                 ┌──────┴──────┐
       ▼             ▼                 ▼             ▼
   EC2 Launch     Fargate           EC2 / Managed   Fargate
   Type           Launch Type       Node Groups     (serverless)
```

> **Key insight for the exam:** Fargate is not a separate orchestrator — it's a **serverless compute option** that both ECS and EKS can use instead of managing EC2 instances yourself.

---

# 📦 Amazon ECS (Elastic Container Service)

## Core Terminology

| Term | Definition |
| :--- | :--- |
| **Task Definition** | Blueprint describing container image, CPU/RAM, ports, IAM roles |
| **Task** | A running instance of a Task Definition (like a container instance) |
| **Service** | Maintains a desired number of running Tasks, integrates with load balancers |
| **Cluster** | Logical grouping of resources (EC2 instances or Fargate capacity) where Tasks/Services run |

```
ECS Cluster
   │
   ├── Service A (maintains desired count of Tasks)
   │      ├── Task 1 (running container)
   │      ├── Task 2 (running container)
   │      └── Task 3 (running container)
   │
   └── Service B
          └── Task 1
```

---

## ECS — EC2 Launch Type

You manage the underlying EC2 infrastructure yourself.

![Amazon ECS - EC2 Launch Type](assets/Amazon%20ECS%20-%20EC2%20Launch%20Type.png)

| Responsibility | Who handles it |
| :--- | :--- |
| Provisioning EC2 instances | **You** |
| Patching/maintaining EC2 OS | **You** |
| Running the ECS Agent | **You** (must be installed and registered) |
| Starting/stopping containers on the instances | AWS |

**ECS Agent** — a process that must run on every EC2 instance in the cluster. It registers the instance with ECS and reports its available capacity (CPU/RAM).

---

## ECS — Fargate Launch Type

Serverless — no EC2 instances to manage at all.

![Amazon ECS – Fargate Launch Type](assets/Amazon%20ECS%20%E2%80%93%20Fargate%20Launch%20Type.png)

| Aspect | Fargate |
| :--- | :--- |
| Infrastructure provisioning | ❌ None required |
| Configuration | Define CPU/RAM in Task Definition |
| Scaling | Just increase number of Tasks — no instance management |
| Pricing | Pay per Task based on vCPU/RAM allocated and runtime |

> **Exam comparison:** EC2 Launch Type = you manage servers, more control, potentially cheaper at scale. Fargate = zero infrastructure management, simpler operations, pay-per-task.

---

## ECS — EC2 vs Fargate Comparison

| Feature | EC2 Launch Type | Fargate Launch Type |
| :--- | :--- | :--- |
| Infrastructure management | Manual (you manage EC2) | ✅ None (serverless) |
| ECS Agent required | ✅ Yes | ❌ N/A |
| Scaling complexity | Higher (scale EC2 + tasks) | Lower (scale tasks only) |
| Cost model | Pay for EC2 instances | Pay per task (CPU/RAM/time) |
| Use case | Cost optimization at scale, custom AMIs | Simplicity, variable workloads |

---

## ECS — IAM Roles (Critical Exam Topic)

There are **two distinct IAM roles** in ECS, and confusing them is a classic exam trap.

### 1 — EC2 Instance Profile (EC2 Launch Type only)

Used by the **ECS Agent** running on the EC2 host itself.

| Permission needed for | Purpose |
| :--- | :--- |
| ECS API calls | Agent registers/reports to ECS service |
| CloudWatch Logs | Send container logs |
| Amazon ECR | Pull Docker images |
| Secrets Manager / SSM Parameter Store | Reference sensitive data referenced at container startup |

### 2 — ECS Task Role

Used by the **application code running inside the container/task itself**.

* Defined in the **Task Definition**
* Provides fine-grained, per-task permissions
* Different tasks/services can have completely different roles

```
ECS Cluster
   │
   ├── Task A (Task Role: S3 read access)      → can call S3 APIs
   └── Task B (Task Role: DynamoDB write access) → can call DynamoDB APIs
                  ↑ each task's permissions are isolated
```

> **Exam trap:** EC2 Instance Profile = permissions for the **agent/host**. ECS Task Role = permissions for the **application inside the container**. They are NOT interchangeable, and Fargate tasks only ever use the Task Role (no EC2 instance profile exists).

---

## ECS — Load Balancer Integration

| Load Balancer | Recommendation |
| :--- | :--- |
| **Application Load Balancer (ALB)** | ✅ Recommended for most use cases |
| **Network Load Balancer (NLB)** | Use only for high throughput/performance, or with AWS PrivateLink |
| **Classic Load Balancer (CLB)** | ❌ Not recommended — no advanced features, **doesn't support Fargate** |

> **Exam trap:** If a question mentions Fargate + a load balancer requirement, Classic Load Balancer is **never** the correct answer.

![Amazon ECS – Load Balancer Integrations](assets/Amazon%20ECS%20%E2%80%93%20Load%20Balancer%20Integrations.png)

---

## ECS — Data Volumes (Amazon EFS)

ECS Tasks can mount **Amazon EFS** file systems for persistent, shared storage.

![Amazon ECS – Data Volumes (EFS)](assets/Amazon%20ECS%20%E2%80%93%20Data%20Volumes%20(EFS).png)

| Feature | Detail |
| :--- | :--- |
| Compatible launch types | EC2 **and** Fargate |
| Cross-AZ sharing | ✅ Tasks in any AZ access the same data |
| Fargate + EFS | = **fully serverless** persistent storage |
| Use case | Persistent multi-AZ shared storage for stateful containers |

> **Exam trap:** **Amazon S3 cannot be mounted as a file system** on ECS tasks. If the question requires a true mounted file system, the answer is EFS (or FSx), never S3.

---

## ECS Service Auto Scaling

Automatically adjusts the number of running **Tasks** based on demand. This uses **AWS Application Auto Scaling** — a different mechanism from EC2 Auto Scaling Groups.

### Scaling Metrics

| Metric | Scales based on |
| :--- | :--- |
| ECS Service Average CPU Utilization | Task-level CPU usage |
| ECS Service Average Memory Utilization | Task-level RAM usage |
| ALB Request Count Per Target | Requests hitting the load balancer per task |

### Scaling Policy Types

| Policy | Behavior |
| :--- | :--- |
| **Target Tracking** | Maintains a target value for a chosen metric (e.g., keep CPU at 50%) |
| **Step Scaling** | Scales based on the magnitude of a CloudWatch Alarm breach |
| **Scheduled Scaling** | Scales at specific dates/times for predictable load patterns |

> **Critical distinction:** **ECS Service Auto Scaling** (Task count) is completely different from **EC2 Auto Scaling** (instance count). On Fargate, only Service Auto Scaling exists since there are no EC2 instances. On EC2 Launch Type, you may need BOTH mechanisms working together.

---

## EC2 Launch Type — Scaling the Underlying Instances

When using EC2 Launch Type, scaling Tasks alone isn't enough — you also need enough EC2 capacity to host them.

| Method | How it Works |
| :--- | :--- |
| **Auto Scaling Group (ASG) Scaling** | Scale EC2 instance count based on CPU utilization metrics |
| **ECS Cluster Capacity Provider** | Automatically provisions/scales EC2 infrastructure paired with an ASG — adds instances when ECS detects missing CPU/RAM capacity |

### Full Scaling Flow Example

![ECS Scaling – Service CPU Usage Example](assets/ECS%20Scaling%20%E2%80%93%20Service%20CPU%20Usage%20Example.png)

> **Capacity Provider is the key exam answer** for "how does ECS automatically add EC2 capacity when Tasks can't be placed due to insufficient resources."

---

## ECS Event-Driven Patterns (EventBridge Integration)

### Pattern 1 — S3 Upload Triggers ECS Task

![ECS tasks invoked by Event Bridge](assets/ECS%20tasks%20invoked%20by%20Event%20Bridge.png)

### Pattern 2 — Scheduled ECS Task (Batch Processing)

```
Amazon EventBridge (Scheduled Rule — every 1 hour)
        │
        ▼
   New ECS Task (Fargate) launched
        │  (uses ECS Task Role)
        ├──▶ Reads batch data from S3
        └──▶ Writes results back to S3
```

### Pattern 3 — SQS Queue Driving ECS Auto Scaling

```
Messages arrive in SQS Queue
        │
        ▼
ECS Service A Tasks poll the queue
        │
        ▼ (as queue depth increases)
ECS Service Auto Scaling adds more Tasks
        │
        ▼
More Tasks poll in parallel → faster queue drain
```

### Pattern 4 — Intercepting Stopped Tasks (Failure Alerting)

![ECS – Intercept Stopped Tasks using EventBridge](assets/ECS%20%E2%80%93%20Intercept%20Stopped%20Tasks%20using%20EventBridge.png)

**Event Pattern JSON:**
```json
{
  "source": [ "aws.ecs" ],
  "detail-type": [ "ECS Task State Change" ],
  "detail": {
    "lastStatus": [ "STOPPED" ],
    "stoppedReason": [ "Essential container in task exited" ]
  }
}
```

> **Exam takeaway:** EventBridge is the standard mechanism for both **triggering** ECS Tasks (on events or schedules) and **reacting** to ECS Task state changes (like failures).

---

# 🗃️ Amazon ECR (Elastic Container Registry)

## What is ECR?

A fully managed Docker container registry for storing, managing, and deploying container images.

| Feature | Detail |
| :--- | :--- |
| Repository types | **Private** and **Public** (ECR Public Gallery) |
| Backend storage | Backed by **Amazon S3** |
| Access control | **IAM** policies (permission errors = check the policy first) |
| Integration | Fully integrated with ECS and EKS |
| Vulnerability scanning | ✅ Supported |
| Versioning & tagging | ✅ Image tags supported |
| Lifecycle management | ✅ Automatic cleanup of old/unused images |

> **Exam trap:** "Image pull permission errors" in ECS/EKS almost always point to a missing or incorrect **IAM policy** on the role pulling from ECR (EC2 Instance Profile for EC2 Launch Type, or the Task Execution Role for Fargate).

---

# ⎈ Amazon EKS (Elastic Kubernetes Service)

## What is EKS?

A managed service for running **Kubernetes** — the open-source container orchestration system — on AWS, without managing the Kubernetes control plane yourself.

| Aspect | Detail |
| :--- | :--- |
| Alternative to | Amazon ECS (same goal, different — open-source — API) |
| Node support | EC2 (self-managed or managed node groups) or **Fargate** (serverless) |
| Cloud-agnostic | Kubernetes itself works the same on AWS, Azure, GCP |
| Best fit | Companies already running Kubernetes on-premises or in another cloud |
| Multi-region | One EKS **cluster per region** (no native cross-region cluster) |
| Observability | **CloudWatch Container Insights** for logs and metrics |

> **When to choose EKS over ECS:** The scenario mentions an **existing Kubernetes investment**, a need for **cloud portability**, or a team already skilled in `kubectl` and Kubernetes manifests. Choose **ECS** when the team wants the simplest AWS-native option with no Kubernetes complexity.

---

## EKS Architecture

![Amazon EKS - Diagram](assets/Amazon%20EKS%20-%20Diagram.png)

* **Nodes run in private subnets** — not directly internet-accessible
* **Public Service LB (ELB)** — routes external traffic into the cluster
* **Private Service LB (ELB)** — routes internal/service-to-service traffic
* **EKS Nodes** are automatically grouped into an Auto Scaling Group

---

## EKS — Node Types

| Node Type | Who Manages It | Notes |
| :--- | :--- | :--- |
| **Managed Node Groups** | AWS creates/manages EC2 nodes for you | Part of an ASG managed by EKS; supports On-Demand or Spot |
| **Self-Managed Nodes** | You create and register nodes yourself | Managed via your own ASG; can use the EKS Optimized AMI; supports On-Demand or Spot |
| **AWS Fargate** | Fully serverless | No nodes to manage at all |

| Comparison | Managed Node Groups | Self-Managed Nodes | Fargate |
| :--- | :--- | :--- | :--- |
| Provisioning | AWS-automated | Manual | None |
| Customization | Limited | Full control (custom AMIs) | None needed |
| Operational overhead | Low | High | Lowest |

---

## EKS — Data Volumes

EKS supports persistent storage via the **Container Storage Interface (CSI)**.

| Step | Requirement |
| :--- | :--- |
| 1 | Define a **StorageClass** manifest on the cluster |
| 2 | Use a **CSI-compliant driver** for the storage backend |

### Supported Storage Backends

| Storage | Works with Fargate? |
| :--- | :--- |
| **Amazon EBS** | ❌ No (EBS is single-AZ block storage, incompatible with Fargate's model) |
| **Amazon EFS** | ✅ Yes |
| **Amazon FSx for Lustre** | High-performance computing workloads |
| **Amazon FSx for NetApp ONTAP** | Enterprise NAS migration |

> **Exam trap:** If the scenario uses EKS **+ Fargate** and needs persistent shared storage, the answer is **EFS** — not EBS (EBS attaches to a single instance/AZ, which conflicts with Fargate's serverless model).

---

# 📊 Amazon ECS vs Amazon EKS — Master Comparison

| Feature | Amazon ECS | Amazon EKS |
| :--- | :--- | :--- |
| API | AWS-proprietary | Kubernetes (open-source, portable) |
| Learning curve | Lower | Higher (need Kubernetes knowledge) |
| Cloud portability | ❌ AWS-only | ✅ Works on any cloud running Kubernetes |
| Launch types | EC2, Fargate | EC2 (Managed/Self-managed), Fargate |
| Control plane cost | Free (no separate charge) | Hourly charge for the EKS control plane |
| Best for | AWS-native teams, simpler setups | Teams already using Kubernetes, multi-cloud strategy |
| Storage support | EFS (S3 cannot be mounted) | EBS, EFS, FSx for Lustre, FSx for ONTAP |

---

# 🚀 AWS App Runner

## What is App Runner?

A fully managed service for deploying **web applications and APIs** quickly — without needing any infrastructure or container orchestration experience.

![AWS App Runner](assets/AWS%20App%20Runner.png)

| Feature | Detail |
| :--- | :--- |
| Input options | Source code repo **or** a pre-built container image |
| Build/deploy | Fully automatic |
| Scaling | Automatic, based on traffic |
| High availability | Built-in |
| Load balancing | Built-in |
| Encryption | Built-in |
| Networking | VPC access supported |
| Integrations | Databases, caches, message queues |

## Workflow

```
1. Configure Settings (vCPU, RAM, Auto Scaling, Health Check)
        │
        ▼
2. Create & Deploy
        │
        ▼
3. Access via auto-generated URL
```

## Use Cases

* Web applications and REST APIs
* Microservices needing rapid deployment
* Teams wanting Heroku-like simplicity on AWS without managing ECS/EKS directly

> **When to choose App Runner over ECS/EKS:** The scenario emphasizes **simplicity and speed** — "deploy quickly without managing infrastructure or clusters." If the question mentions custom orchestration, complex networking, or Kubernetes — ECS or EKS is the better fit.

---

# 🔄 AWS App2Container (A2C)

## What is App2Container?

A **command-line tool** that automatically containerizes existing **Java** and **.NET** web applications — without requiring any code changes.

| Aspect | Detail |
| :--- | :--- |
| Supported languages | **Java** and **.NET** |
| Strategy | Lift-and-shift modernization |
| Code changes required | ❌ None |
| Source environments | On-premises bare metal, VMs, or any cloud |
| Output | CloudFormation templates (compute + network infrastructure) |
| Image registry | Registers generated images to **Amazon ECR** |
| Deployment targets | **ECS**, **EKS**, or **App Runner** |
| CI/CD | Supports pre-built CI/CD pipeline generation |

## App2Container — 4-Step Process

![AWS App2Container](assets/AWS%20App2Container.png)

> **When to choose App2Container:** The scenario describes an **existing legacy Java or .NET application** that needs to be modernized into containers **without rewriting code**. This is the classic "lift-and-shift to containers" answer.

---

# ⚠️ Exam Traps & Common Mistakes

> **Trap 1 — S3 cannot be mounted as a file system on ECS or EKS.**
> If the question needs a mountable, persistent shared file system, the answer is **EFS** (or FSx) — never S3.

> **Trap 2 — EC2 Instance Profile vs ECS Task Role.**
> EC2 Instance Profile = permissions for the ECS Agent on the host. ECS Task Role = permissions for the application inside the container. Fargate only uses Task Roles (no EC2 instances exist).

> **Trap 3 — Classic Load Balancer does not support Fargate.**
> If Fargate is mentioned alongside a load balancer requirement, never select Classic Load Balancer.

> **Trap 4 — EKS + Fargate cannot use EBS for storage.**
> EBS is single-AZ block storage. Use **EFS** for persistent storage in EKS-on-Fargate scenarios.

> **Trap 5 — ECS Service Auto Scaling ≠ EC2 Auto Scaling.**
> ECS Service Auto Scaling adjusts **Task count**. EC2 Auto Scaling (or ECS Capacity Providers) adjusts the **underlying instance count**. On EC2 Launch Type, you typically need both working together.

> **Trap 6 — Capacity Provider is the answer for automatic EC2 scaling tied to ECS demand.**
> If the scenario describes ECS Tasks failing to launch due to insufficient EC2 capacity, and asks for an automated fix, the answer is an **ECS Cluster Capacity Provider** paired with an ASG.

> **Trap 7 — EKS is one cluster per region.**
> There is no single EKS cluster spanning multiple AWS regions natively.

> **Trap 8 — App2Container only supports Java and .NET.**
> If the legacy app is in another language, App2Container is not the right tool.

> **Trap 9 — App Runner vs ECS/EKS.**
> App Runner is for **simplicity** — minimal config, no cluster management. If the scenario requires fine-grained orchestration control, custom networking, or Kubernetes — choose ECS/EKS instead.

> **Trap 10 — Docker containers share the host OS kernel; VMs do not.**
> This is the core architectural distinction tested when comparing Docker to traditional virtualization.

---

# 🧠 High-Frequency Exam Facts

* Docker containers share the **host OS kernel** — VMs each run a full Guest OS
* Docker images live in **repositories**: Docker Hub (public) or Amazon ECR (private/public)
* ECS = AWS-proprietary orchestration; EKS = managed **Kubernetes** (cloud-portable)
* **Fargate** is a serverless compute engine usable by **both** ECS and EKS
* ECS **EC2 Launch Type** requires running the **ECS Agent** on each instance
* ECS **Fargate Launch Type** = zero infrastructure management
* **EC2 Instance Profile** = permissions for the ECS Agent/host
* **ECS Task Role** = permissions for the application inside the container (defined in Task Definition)
* **ALB** is the recommended load balancer for ECS; **Classic LB doesn't support Fargate**
* **S3 cannot be mounted** as a file system — use **EFS** for ECS/EKS shared storage
* Fargate + EFS = fully serverless persistent storage
* ECS Service Auto Scaling uses **AWS Application Auto Scaling** (3 policy types: Target Tracking, Step, Scheduled)
* **ECS Capacity Provider** automatically scales EC2 instances paired with an ASG based on ECS demand
* EventBridge can both **trigger** ECS Tasks (on S3 events or schedules) and **react** to ECS Task state changes
* ECR is **backed by Amazon S3** and access-controlled via **IAM**
* EKS supports **Managed Node Groups, Self-Managed Nodes, or Fargate**
* EKS data volumes use **CSI drivers**: EBS, EFS, FSx for Lustre, FSx for NetApp ONTAP
* EKS + Fargate **cannot use EBS** — use EFS instead
* **App Runner** = fastest path from source code/container to a deployed, scaled, HTTPS-secured URL
* **App2Container** = Java/.NET legacy app containerization with **no code changes**, outputs CloudFormation + ECR + deploys to ECS/EKS/App Runner

---

# 📐 Architecture Pattern Quick Reference

| Requirement | Service / Pattern |
| :--- | :--- |
| Run containers with zero infrastructure management | ECS or EKS with **Fargate** |
| Run containers with full control over EC2 instances | ECS or EKS **EC2 Launch Type** |
| Already using Kubernetes on-prem or multi-cloud | **Amazon EKS** |
| Simplest AWS-native container orchestration | **Amazon ECS** |
| Persistent shared storage across containers in multiple AZs | **Amazon EFS** mounted to ECS/EKS tasks |
| Automatically add EC2 capacity for ECS as demand grows | **ECS Capacity Provider** + ASG |
| Scale Task count based on CPU/Memory/ALB requests | **ECS Service Auto Scaling** |
| Trigger a container task when a file lands in S3 | **EventBridge → ECS Task (Fargate)** |
| Run a container task on a schedule (e.g., hourly batch job) | **EventBridge Scheduled Rule → ECS Task** |
| Alert an admin when a container task fails | **EventBridge → SNS → Email** |
| Process messages from a queue with auto-scaling workers | **SQS → ECS Service with Auto Scaling** |
| Deploy a web app/API with minimal setup | **AWS App Runner** |
| Containerize a legacy Java/.NET app with no code changes | **AWS App2Container** |
| Store and manage Docker images on AWS | **Amazon ECR** |

---

# 📚 Conclusion

This guide covers the AWS container services most commonly tested in SAA-C03. The exam tests both **conceptual understanding** (Docker vs VMs, ECS vs EKS) and **scenario-based decision making** (which service/pattern fits a described requirement).

**Core decision framework:**

* Need the **simplest AWS-native** container platform? → **ECS**
* Need **Kubernetes** portability or already using K8s? → **EKS**
* Want **zero infrastructure management**? → **Fargate** (with either ECS or EKS)
* Need **persistent shared storage** across containers? → **EFS** (never S3)
* Need to **scale Task count**? → ECS Service Auto Scaling
* Need to **scale underlying EC2 capacity**? → ECS Capacity Provider + ASG
* Need the **fastest path to a deployed web app/API**? → **App Runner**
* Need to **containerize a legacy Java/.NET app** with no code changes? → **App2Container**
* Need to **store Docker images** on AWS? → **Amazon ECR**

---

<div align="center">

### ☁️ AWS SAA-C03 Containers Domain Reference

**Docker • Amazon ECS • Amazon EKS • Amazon ECR • AWS Fargate • App Runner • App2Container**

</div>