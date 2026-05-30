# 💻 EC2 – Exam Notes

## ☁️ What is EC2?

EC2 (Elastic Compute Cloud) is a Regional AWS Service that provides virtual servers in the cloud.

EC2 implements **Infrastructure as a Service (IaaS)** — you manage the OS and above, AWS manages the physical hardware.

---

## 🖥️ EC2 Instance Configuration

When launching an EC2 instance, you configure six core subsystems.

### 📌 Configurable Subsystems

* 💿 **OS** — via Amazon Machine Image (AMI): Linux, Windows, macOS.
* ⚙️ **Compute** — vCPU count, generation, and clock speed.
* 🧠 **Memory** — RAM density measured in GiB.
* 🌐 **Network** — link speed, bandwidth, and IP assignment.
* 💾 **Storage** — EBS (network-attached) or Instance Store (local NVMe).
* 🛡️ **Firewall** — Security Groups control inbound/outbound traffic.

💡 EC2 gives you full control over the virtual hardware plane.

---

## 🏷️ Instance Naming Convention

Instance type names follow a strict format that encodes the hardware profile.

### 📌 Format

```
m5.2xlarge
│ │  └── Size      → Resource footprint (vCPUs, RAM, network)
│ └───── Generation → Higher = newer hardware & better efficiency
└─────── Family     → Optimization category (e.g., General Purpose)
```

---

## 🗂️ Instance Families

### 📌 Four Main Families

* 🟢 **General Purpose** — Balanced CPU, RAM, and network. Best for web apps and dev environments.
  * Families: `T2`, `T3`, `M5`, `M6g`
* 🔵 **Compute Optimized** — High-performance CPUs. Best for HPC, batch jobs, game servers.
  * Families: `C4`, `C5`, `C6g`
* 🟣 **Memory Optimized** — Large RAM pools. Best for in-memory databases, Redis, Memcached.
  * Families: `R4`, `R5`, `X1`, `z1d`
* 🟠 **Storage Optimized** — High IOPS local storage. Best for OLTP, NoSQL, data warehouses.
  * Families: `I3`, `D2`, `D3`, `H1`

---

## ⚡ EC2 User Data (Bootstrap)

User Data is a script that runs **once** automatically on first boot.

### 📌 Characteristics

* 🚀 Runs at instance launch only — not on reboot or stop/start.
* 👑 Executes as the `root` user automatically.
* 🤖 Used to automate post-launch configuration.

### 🛠️ Common Use Cases

* 📦 Install packages (Docker, Nginx, Python, Git).
* 🔄 Run system updates and security patches.
* 🔧 Pull configuration files or application code.

💡 User Data eliminates manual post-boot configuration and prevents configuration drift.

---

## 🛡️ Security Groups

Security Groups act as a **virtual stateful firewall** for EC2 instances.

### 📌 Characteristics

* ✅ **Allow rules only** — you cannot write explicit Deny rules.
* 🔄 **Stateful** — return traffic is automatically allowed.
* 🚫 **Default Inbound** — all traffic blocked by default.
* ✅ **Default Outbound** — all traffic allowed by default.
* 🔗 Can reference other Security Groups instead of IPs.
* 🌐 Operate at the AWS SDN layer — outside the guest OS.

### 🚨 Important Rule

❌ If a packet does not match any Allow rule, it is dropped silently.

---

## 🖧 Common Network Ports

Ports you must know for the exam.

### 📌 Port Reference

* 🔒 **Port 22** — SSH / SFTP (Linux remote access)
* 📁 **Port 21** — FTP (unencrypted file transfer)
* 🌐 **Port 80** — HTTP (unencrypted web traffic)
* 🔐 **Port 443** — HTTPS (TLS-encrypted web traffic)
* 🖥️ **Port 3389** — RDP (Windows remote desktop)

⚠️ Never open Port 22 or Port 3389 to `0.0.0.0/0` in production.

---

## 🌐 EC2 Instance Connect

EC2 Instance Connect allows browser-based SSH without managing key files.

### 📌 Characteristics

* 🚫 No `.pem` or `.ppk` key file required.
* 🔑 AWS injects a **temporary public key** into the instance via the metadata plane.
* ⏱️ The temporary key is valid for **60 seconds** only.
* ✅ Requires **Port 22** open in the Security Group.
* 💿 Works out-of-the-box on Amazon Linux 2 and modern AMIs.

---

## 💳 EC2 Purchasing Options

AWS offers multiple pricing models to match workload patterns and cost targets.

### 📌 Purchasing Models

* 🟢 **On-Demand**
  * Pay per second (Linux/Windows) or per hour (other OS).
  * No upfront commitment.
  * Highest cost — best for short-term or unpredictable workloads.

* 🔵 **Reserved Instances (RIs)**
  * 1-year or 3-year commitment.
  * Up to **72% discount** vs On-Demand.
  * Convertible RIs offer up to **66% discount** with flexibility to change family/OS/size.
  * Best for steady-state, long-running workloads.

* 🟣 **Savings Plans**
  * Commit to a fixed spend rate (e.g., `$10/hr`) for 1 or 3 years.
  * Up to **72% discount** — more flexible than RIs.
  * Works across instance families, sizes, and OS types.

* 🟠 **Spot Instances**
  * Bid on unused AWS capacity.
  * Up to **90% discount** vs On-Demand.
  * ⚠️ Can be **interrupted** with a 2-minute warning.
  * Best for fault-tolerant, stateless batch jobs.
  * 🚫 Never use for databases or critical stateful workloads.

* ⚫ **Dedicated Instances**
  * Runs on hardware dedicated to your account.
  * May share the physical host with other instances in your account.

* 🔴 **Dedicated Hosts**
  * Rent the entire physical bare-metal server.
  * Full visibility over sockets, cores, and host IDs.
  * Required for BYOL (Bring Your Own License) compliance.
  * Most expensive option.

* ⬛ **Capacity Reservations**
  * Reserve capacity in a specific AZ for any duration.
  * No time commitment — billed at On-Demand rates even if unused.
  * Best for guaranteed capacity during critical events.

---

## 🎯 Spot Instances — Deep Dive

### 📌 Request Types

* 🔂 **One-Time** — Request closes if the instance is interrupted.
* 🔄 **Persistent** — Request stays open; new instance is launched when capacity returns.

### 🛑 Critical Deletion Rule

> To fully terminate a Spot environment:
> 1. **Cancel the Spot Request first.**
> 2. **Then terminate the Spot Instances.**
>
> ❌ Canceling the request alone does NOT terminate running instances.

---

## 🚢 Spot Fleets

A Spot Fleet manages a pool of Spot (and optionally On-Demand) instances to meet a target capacity at lowest cost.

### 📌 Allocation Strategies

* 💸 **lowestPrice** — Pick the cheapest pool. High interruption risk.
* 🔀 **diversified** — Spread across all pools. Best availability.
* 💪 **capacityOptimized** — Pick the pool with most available capacity. Lower interruption risk.
* ⭐ **priceCapacityOptimized** — Balance capacity and price. **Recommended for most workloads.**

---

## 🔥 High-Frequency Exam Facts

* 💻 EC2 is a **Regional** service, not Global.
* ☁️ EC2 is **IaaS** — you manage the OS and above.
* 💿 AMIs define the OS and base configuration.
* ⚡ User Data runs **once** at first launch, as `root`.
* 🛡️ Security Groups are **stateful** and use **Allow rules only**.
* 🚫 Default inbound = **all blocked**. Default outbound = **all allowed**.
* 🔗 Security Groups can reference **other Security Groups** (not just IPs).
* 🔒 Port 22 = SSH, Port 3389 = RDP, Port 80 = HTTP, Port 443 = HTTPS.
* 🌐 EC2 Instance Connect = browser SSH, **no key file**, 60-second temp key.
* 💸 On-Demand = highest cost, no commitment.
* 💰 Reserved Instances = up to **72% off**, 1 or 3-year term.
* 🎯 Spot Instances = up to **90% off**, can be **interrupted**.
* 🔴 Dedicated Hosts = **most expensive**, required for **BYOL**.
* ⭐ Best Spot Fleet strategy = **`priceCapacityOptimized`**.
* 🛑 Cancel Spot Request **before** terminating Spot Instances.
* 🚫 Never use Spot for **stateful or critical workloads**.
* 🔑 EC2 should use **IAM Roles** (not hardcoded Access Keys) for AWS API access.