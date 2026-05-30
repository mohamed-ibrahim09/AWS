# 💻 Amazon EC2 — Elastic Compute Cloud: Complete Reference Guide

<div align="center">

![AWS](https://img.shields.io/badge/AWS-EC2-orange?style=for-the-badge&logo=amazonaws)
![Compute](https://img.shields.io/badge/Compute-IaaS-blue?style=for-the-badge)
![Networking](https://img.shields.io/badge/Networking-Security%20Groups-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Comprehensive-success?style=for-the-badge)

### ⚡ Elastic Compute Cloud (EC2) Foundations & Architecture Guide

*A comprehensive guide to AWS EC2 architecture, virtual hardware subsystems, bootstrap automation mechanics, instance taxonomies, perimeter network security groups, and cloud financial orchestration models.*

</div>

---

## 📖 Overview

This comprehensive reference manual explores the architectural design, sizing subsystems, programmatic network filters, and resource management models of Amazon Elastic Compute Cloud (Amazon EC2). Understanding these foundational layers is critical for designing scalable, secure, and cost-optimized cloud-native infrastructure.

---

## 🏛️ 1. Architectural Foundation & The Compute Engine

Amazon Elastic Compute Cloud (Amazon EC2) represents a core pillar of the AWS cloud fabric. In legacy, on-premises corporate infrastructure, deploying compute capacity required physical hardware procurement, data center rack assembly, and weeks of manual configuration.

```
[ Legacy Infrastructure ]
  Physical Hardware Rack ──► CapEx Purchase ──► Manual Assembly (Weeks)

[ AWS Compute Infrastructure ]
  API Plane / Web Console ──► Virtual Machine ──► Multi-Tenant Hypervisor (Seconds)

```

Within the cloud systems model, EC2 implements **Infrastructure as a Service (IaaS)**. This structural architecture decouples your applications from physical server management. AWS handles host facilities, hardware lifecycles, and underlying hypervisor layers, while you retain full administrative authority over the virtual hardware plane.

```
┌────────────────────────────────────────────────────────┐
│               EC2 VIRTUAL HARDWARE MATRIX              │
├───────────────────┬────────────────────────────────────┤
│ Operating System  │ Linux, Windows, macOS platforms    │
├───────────────────┼────────────────────────────────────┤
│ Compute Power     │ vCPU Core Allocation & Generation  │
├───────────────────┼────────────────────────────────────┤
│ Volatile Memory   │ RAM Density (Measured in GiB)      │
├───────────────────┼────────────────────────────────────┤
│ Storage Fabrics   │ Ephemeral Instance Store vs. EBS   │
├───────────────────┼────────────────────────────────────┤
│ Network Interface │ Line Rates, Bandwidth & Public IPs │
├───────────────────┼────────────────────────────────────┤
│ Perimeter Defense │ Stateful Security Group Firewalls  │
└───────────────────┴────────────────────────────────────┘

```

### The Sizing & Configuration Subsystems

When designing an EC2 computing node, engineers select and configure five independent infrastructural subsystems:

* **Operating System (OS Engine):** Pre-configured platform architectures available via Amazon Machine Images (AMIs), spanning various flavors of Linux, Microsoft Windows, and Apple macOS.
* **Compute Provisioning (vCPUs):** The specific virtual processing cores, hardware clock rates, and multi-threading options deployed to execute application instructions.
* **Volatile Memory Layer (RAM):** The raw random-access memory block capacity required to buffer execution payloads and run application daemons.
* **Network Interfaces (NIC Subsystem):** Network capability settings including provisioned link speeds, target bandwidth caps, and public or private IP address assignment routines.
* **Storage Architecture:** Split into two distinct execution models:
  * **Network-Attached Storage (EBS & EFS):** Block and file systems that access data across an internal network fabric, allowing data to survive instance termination.
  * **Hardware Instance Store:** Non-volatile storage devices (NVMe or enterprise SSDs) physically wired to the motherboard hosting the virtual server, providing extreme read/write throughput with ephemeral lifecycles.

---

## ⚡ 2. Initialization Mechanics: The Bootstrap Plane

Configuring infrastructure nodes by manually running shell commands after boot creates configuration drift and introduces human error at scale. Amazon EC2 solves this with the **EC2 User Data** automation subsystem.

```
[ API Provision Request ] ──► [ Hardware Mapping ] ──► [ Metadata Evaluation ]
                                                              │
                                                              ▼
[ Ready State ] ◄─── [ Custom Package Initialization ] ◄─── [ Script Execution as root ]

```

### Script Execution & Lifecycle Constraints

* **The Bootstrap Process:** During instance creation, you write or pass a shell configuration script directly into the launch configuration via the User Data payload block.
* **Execution Boundary:** **The user data script executes exactly once during the instance's initial creation boot lifecycle.** Standard system reboots, hardware migrations, or stop/start operations do not trigger re-execution.
* **Security & Administrative Privileges:** The initialization engine triggers the user data script with maximum administrative capabilities under the **`root`** user account. This lets you execute system updates and install core software libraries without building complex authentication workflows into the script.

### Production Automation Use Cases

The bootstrap plane is a secure entry point for system configuration tools:

* Downloading and installing security patches, hotfixes, and base operating system updates.
* Installing core container engines, applications, and system runtimes (e.g., Docker, Python, Nginx, Git).
* Securing configuration variables, application binaries, or infrastructure dependencies from deployment storage endpoints or code repositories.

---

## 🗂️ 3. Instance Taxonomy: Classes & Workload Optimization

AWS organises EC2 hardware into clear, distinct instance families. Each family is engineered to optimize a specific hardware balance — compute, memory, or storage — for particular business and workload requirements.

### Decoding the Naming Convention

Instance classification labels follow a strict naming architecture that communicates the exact capabilities of the underlying compute node:

$$\text{Instance Profiling Vector: } \mathbf{m5.2xlarge}$$

* **`m` (Instance Family / Class ID):** Defines the underlying resource priority and optimization layer (e.g., General Purpose).
* **`5` (Generation Level Index):** Tracks hardware iteration history. Higher indexes feature newer, faster physical CPU chips, modern hypervisor systems, and improved efficiency profiles.
* **`2xlarge` (Instance Capacity Size):** Represents the specific resource footprint (vCPU counts, RAM capacity, and internal network speeds) allocated relative to others in that family.

### Specialized Hardware Families

#### 1. General Purpose

* **Core Profile:** Maintains an even, balanced distribution of computing power, system memory density, and network throughput profiles.
* **Primary Workloads:** Ideal for low-to-medium traffic web applications, source code repository hosts, integration testing sandboxes, and balanced backend enterprise workflows.
* **Active Families:** `Mac`, `T4g`, `T3`, `T3a`, `T2`, `M6g`, `M5`, `M5a`, `M5n`, `M5zn`, `M4`, `A1`.

#### 2. Compute Optimized

* **Core Profile:** Built for calculation-heavy systems that need high-performance, raw single-core or multi-core clock speeds from their processors.
* **Primary Workloads:** Used for large high-performance computing (HPC) environments, heavy batch data processing, real-time media transcoding engines, high-performance web endpoints, scientific simulation algorithms, and dedicated multiplayer game servers.
* **Active Families:** `C6g`, `C6gn`, `C5`, `C5a`, `C5n`, `C4`.

#### 3. Memory Optimized

* **Core Profile:** Designed to process massive enterprise datasets directly inside fast, volatile random-access memory (RAM) channels without data caching bottlenecks.
* **Primary Workloads:** Powering enterprise relational database structures, web-scale key-value caching clusters (such as Redis or Memcached), distributed in-memory data warehouses, and real-time big data analytical engines.
* **Active Families:** `R6g`, `R5`, `R5a`, `R5b`, `R5n`, `R4`, `X1e`, `X1`, `High Memory`, `z1d`.

#### 4. Storage Optimized

* **Core Profile:** Engineered for data-heavy platforms that need sustained, sequential, low-latency high-IOPS read and write access to massive live datasets stored locally on the server.
* **Primary Workloads:** Used for high-frequency Online Transaction Processing (OLTP) systems, transactional NoSQL database setups, high-volume data warehousing appliances, and large-scale distributed file systems.
* **Active Families:** `I3`, `I8n`, `D2`, `D3`, `D3en`, `H1`.

### Structural Hardware Cross-Section Table

This baseline specification matrix shows how computing resources scale across different instance profiles:

| Instance Identity | vCPU Layer | Memory Density (GiB) | Local Storage Configuration | Network Throughput | Dedicated EBS Bandwidth |
| --- | --- | --- | --- | --- | --- |
| **`t2.micro`** | 1 Core | 1 GiB | Network EBS Only | Low to Moderate Link | Shared Bus Architecture |
| **`t2.xlarge`** | 4 Cores | 16 GiB | Network EBS Only | Moderate Link | Shared Bus Architecture |
| **`c5d.4xlarge`** | 16 Cores | 32 GiB | 1 x 400 GiB Local NVMe SSD | Up to 10 Gbps Link | 4,750 Mbps Baseline |
| **`m5.8xlarge`** | 32 Cores | 128 GiB | Network EBS Only | 10 Gbps Provisioned | 6,800 Mbps Baseline |
| **`r5.16xlarge`** | 64 Cores | 512 GiB | Network EBS Only | 20 Gbps Provisioned | 13,600 Mbps Baseline |

---

## 🛡️ 4. Network Perimeter Security: Security Groups

A **Security Group** serves as the initial, non-bypassable virtual network firewall protecting an EC2 instance. It operates at the AWS software-defined networking (SDN) layer, completely outside the guest operating system.

```
       [ Public Internet Requests ] ──► ( Port 80 / 443 Allowed ) ──► [ EC2 Host RAM ]
                                    ──► ( Port 22 / Unauthorized ) ──► [ DROPPED BY DEFAULT ]
                                                                             │
                                              Instance never handles packet ─┘

```
![Security Groups Diagram](assets/Security%20Groups%20Diagram.png)

### Architecture & Stateful Evaluation Engine

* **Decoupled Protection Layer:** Security groups live outside the guest operating system. If an unauthorized packet hits the network interface, **the security group drops the request before the guest OS or application handles it**, saving valuable CPU cycles.
* **Allow-Only Policy Logic:** Security groups use **allow rules only**. You cannot configure explicit deny blocks. If an incoming or outgoing connection does not match an explicit allow rule, it is dropped by default.
* **Stateful Flow Tracking:** The filtering engine is completely stateful. When a packet is allowed through the inbound rule gate, the return outbound traffic is tracked and automatically permitted, bypassing any outbound rule constraints.
* **Structural Defaults:** Out-of-the-box, all incoming external traffic is completely blocked, while all outbound traffic is fully allowed.

### Production Network Rule Schema Matrix

The following table shows how an enterprise production architecture might configure its security rules:

| Rule Type | Network Protocol | Port Allocation | Authorized Network Source (CIDR) | Operational Context |
| --- | --- | --- | --- | --- |
| **`HTTP`** | `TCP` | 80 | `0.0.0.0/0` (Global Public IPv4) | Handling unencrypted web application calls |
| **`HTTPS`** | `TCP` | 443 | `0.0.0.0/0` (Global Public IPv4) | Production encrypted TLS application entry |
| **`SSH`** | `TCP` | 22 | `197.34.112.55/32` (Admin Bastion IP) | Secure engineer remote control access |
| **`Custom TCP`** | `TCP` | 4567 | `sg-0123abc456def7890` (Load Balancer SG) | Private inner-tier app data routing |

### The Power of Security Group Referencing

Instead of configuring static IP addresses that change during auto-scaling events, AWS security groups can reference *other security groups* as valid traffic sources.

This lets you build clean rules like: *"Only allow incoming traffic on port 5432 (PostgreSQL) if the request originates from an EC2 instance carrying the `app-backend` security group identifier."* This keeps your network perimeter secure without needing to track individual server IP addresses.

![The Power of Security Group Referencing](assets/The%20Power%20of%20Security%20Group%20Referencing.png)

---

## 🖧 5. Standard Core Enterprise Network Ports

To safely configure your Security Group rule arrays, you should memorize these foundational application network ports:

```
   ┌─── Port 22   ──► SSH / SFTP (Secure System Terminal Execution)
   ├─── Port 21   ──► FTP (Cleartext File Transfers)
   ├─── Port 80   ──► HTTP (Standard Web Server Access)
   ├─── Port 443  ──► HTTPS (Secure TLS Encrypted Inbound Streams)
   └─── Port 3389 ──► RDP (Remote Desktop Protocol for Windows)

```

| Port Value | Protocol Engine | Architectural Purpose |
| --- | --- | --- |
| **`22`** | `SSH` / `SFTP` | Secure administrative shell command execution on Linux hosts, and encrypted file transfers. |
| **`21`** | `FTP` | Base unencrypted legacy File Transfer Protocol operations. |
| **`80`** | `HTTP` | Unencrypted web data packet delivery over cleartext lines. |
| **`443`** | `HTTPS` | Production-standard web application traffic protected by TLS/SSL encryption. |
| **`3389`** | `RDP` | Graphical User Interface remote management protocol for Windows instance hosts. |

---

## 🌐 6. Native Browser Access: EC2 Instance Connect

Downloading, sharing, and managing static key files (`.pem` or `.ppk`) across multiple systems introduces a risk of identity exposure. **EC2 Instance Connect** provides a secure, keyless alternative for remote server management.

```
[ AWS Management Console ] ──► [ Temporary Public Key Injected via IMDS ] ──► [ EC2 Instance ]
                                        (Valid for 60 seconds)

```

* **No Static Cryptographic Keys Required:** EC2 Instance Connect allows you to open a secure SSH shell terminal directly inside your web browser via the AWS Management Console, eliminating the need to store or rotate local private key files.
* **The Single-Use Public Key Injection Process:** When a terminal connection is requested, the AWS console securely pushes a temporary public key directly to the instance's RAM via the internal EC2 metadata plane. This key stays valid for **60 seconds** and expires automatically after authorization.
* **Prerequisites:** Out-of-the-box, this feature works on modern AMIs like Amazon Linux 2. However, your underlying Security Group must still allow incoming traffic on **Port 22** from the AWS service IP ranges.

---

## 💳 7. Cloud Economics Engine: EC2 Purchasing Options

AWS offers several flexible purchasing models to help you balance system runtime requirements against cloud spending constraints.

```
               ┌────────────────────────────────────────────────────────┐
               │                 EC2 COST MANAGEMENT MODES              │
               └───────────────────────────┬────────────────────────────┘
                                           │
         ┌───────────────────┬─────────────┴─────────────┬───────────────────┐
         ▼                   ▼                           ▼                   ▼
┌─────────────────┐ ┌─────────────────┐         ┌─────────────────┐ ┌─────────────────┐
│    On-Demand    │ │Reserved/Savings │         │  Spot Instances │ │ Dedicated Hosts │
├─────────────────┤ ├─────────────────┤         ├─────────────────┤ ├─────────────────┤
│• On-The-Fly Fix │ │• 1-3 Year Term  │         │• Up to 90% Off  │ │• Isolated Metal │
│• Pay-by-Second  │ │• Up to 72% Off  │         │• Highly Volatile│ │• Custom License │
└─────────────────┘ └─────────────────┘         └─────────────────┘ └─────────────────┘

```

### The Cost Framework Models

#### 1. On-Demand Instances

* **Pricing Model:** Flat hourly rate with zero upfront financial commitment. Linux and Windows are billed by the second after the first minute, while specialized OS platforms are billed by the hour.
* **Use Case:** Best for short-term, unpredictable workloads that cannot handle interruptions. This model offers the highest flexibility but carries the highest baseline cost per hour.

#### 2. Reserved Instances (RIs) & Convertible RIs

* **Pricing Model:** Provides up to a **72% discount** compared to On-Demand rates in exchange for committing to a continuous 1-year or 3-year contract. Payment terms can be configured as No Upfront, Partial Upfront, or All Upfront.
* **Convertible Flexibility:** Convertible Reserved Instances offer up to a **66% discount** while allowing changes to instance families, operating systems, sizes, or scopes over the course of the contract.
* **Use Case:** Best for long-term applications with predictable, steady-state usage, such as production databases.

#### 3. Savings Plans

* **Pricing Model:** Offers the same deep discount structure as standard RIs (up to 72%). Instead of committing to specific hardware configurations, you commit to a fixed spending rate (e.g., `$10/hour`) over a 1 or 3-year term.
* **Use Case:** Best for dynamic setups that need flexible resource resizing across different operating systems, instance sizes, or tenancy modes within a region.

#### 4. Spot Instances

* **Pricing Model:** Allows you to bid on empty, unused AWS hardware capacity, offering discounts up to **90% off** On-Demand pricing.
* **Interruption Risk:** If market demand shifts and the spot price climbs above your defined maximum bid, AWS will reclaim the instance with a **2-minute grace period** warning to cleanly save data and shut down processes.
* **Use Case:** Best for fault-tolerant, stateless batch-processing jobs, distributed big data analysis pipelines, and background microservice workloads. **Never use Spot instances for stateful databases or critical applications.**

#### 5. Dedicated Hosts vs. Dedicated Instances

* **Dedicated Instances:** Your instances run on physical hardware dedicated to your AWS account. However, different instances within your own account may still share the same physical server, and the instance can migrate to new hardware after a stop-and-start cycle.
* **Dedicated Hosts:** You rent an entire physical bare-metal server from AWS, giving you full control and visibility over the sockets, cores, and physical host IDs.
* **Use Case:** Essential for meeting strict compliance standards and satisfying complex, per-core legacy software licensing requirements (Bring Your Own License - BYOL). This is the most expensive computing option available.

#### 6. Capacity Reservations

* **Pricing Model:** Secures dedicated computing capacity within a specific Availability Zone for any duration, with no long-term time commitment.
* **Cost Mechanics:** You are billed at standard On-Demand rates whether you actively run an instance in the reserved slot or leave it empty.
* **Use Case:** Best for short-term, critical events (such as a high-traffic product launch) where you must guarantee that hardware capacity will be available in a specific zone.

---

## ⚡ 8. Deep Dive: Spot Instances & Fleet Orchestration

To run Spot instances cost-effectively at scale, you need to understand their request lifecycles and fleet management strategies.

### Spot Request Lifecycle

Spot requests are split into two operational modes:

* **One-Time Requests:** The request stays active until AWS provisions your instance. If market demand changes and AWS terminates the instance, the request closes automatically and no new instances will be spun up.
* **Persistent Requests:** The request stays active even if market shifts cause your instances to be terminated. If capacity becomes available again and prices drop below your maximum bid, the request will automatically boot a new replacement instance.

> ### 🛑 Crucial Deletion Rule
>
> **Canceling an active Spot Request will not automatically terminate its associated EC2 instances.** To clean up your environment and avoid unexpected costs, you must follow this exact sequence: **First, cancel the Spot Request to prevent new provisions, then manually terminate all active Spot EC2 instances.**

### Spot Fleets

A **Spot Fleet** is a high-level collection of Spot Instances (and optionally On-Demand instances) managed as a single pool to automatically maintain your target compute capacity at the lowest cost.

```
                            ┌───► Pool A (c5.large in us-east-1a)
                            ├───► Pool B (m5.large in us-east-1b)
[ Spot Fleet Controller ] ──┼───► Pool C (t3.large in us-east-1c)
                            └───► Pool D (c5.large in us-east-1d)

```

The fleet controller balances your target resource needs across multiple launch pools using one of four allocation strategies:

| Strategy | Behavior | Risk Profile |
| --- | --- | --- |
| **`lowestPrice`** | Provisions from the single pool with the absolute lowest market price. | High interruption risk if that pool's price spikes. |
| **`diversified`** | Evenly distributes instances across all configured launch pools. | High availability; price spike in one pool only affects a fraction. |
| **`capacityOptimized`** | Targets pools with the largest available unallocated hardware capacity. | Lower interruption risk from high-capacity pools. |
| **`priceCapacityOptimized`** | Identifies highest-capacity pools first for stability, then selects the most cost-effective. | **Recommended** — the gold standard for most production workloads. |

---

## ✅ 9. Production Blueprints & EC2 Best Practices

To design and operate a secure, cost-optimized EC2 environment, integrate these core engineering patterns into your deployment pipelines:

* **Right-Size Instances First:** Start with a general-purpose instance family (`t3` or `m5`) and benchmark actual CPU, memory, and network utilization before committing to a specialized or larger instance type.
* **Automate Bootstrap Configurations:** Use EC2 User Data scripts to eliminate manual post-boot configuration. Store reusable bootstrap scripts in version-controlled repositories to prevent configuration drift.
* **Enforce Least-Privilege Security Groups:** Never open `0.0.0.0/0` on SSH (`Port 22`) or RDP (`Port 3389`) to the public internet. Restrict administrative access to known bastion IP addresses or use EC2 Instance Connect.
* **Leverage Security Group Referencing:** Replace static IP-based rules with Security Group ID references in your inbound rules to maintain clean network perimeters during auto-scaling events.
* **Match Purchasing Model to Workload:** Use On-Demand for short-lived or unpredictable bursts, Reserved Instances or Savings Plans for stable long-running workloads, and Spot Instances for fault-tolerant batch and background jobs.
* **Implement `priceCapacityOptimized` for Spot Fleets:** For workloads requiring Spot Instances at scale, always configure Spot Fleets with the `priceCapacityOptimized` allocation strategy to balance cost savings against interruption risk.
* **Never Embed Credentials in User Data:** Do not hardcode IAM Access Keys or secrets inside bootstrap scripts. Use IAM Roles attached via Instance Profiles to grant EC2 instances temporary, automatically-rotated credentials.