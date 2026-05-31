# 💾 EC2 Storage – Exam Notes

## ☁️ What is EBS?

Amazon EBS (Elastic Block Store) is a **network-attached, persistent block storage** device for a single EC2 instance.

EBS volumes are completely separate from the host hardware — they communicate over a dedicated internal AWS network fabric.

---

## 🔒 EBS Core Characteristics

### 📌 Characteristics

* 🌐 **Network-Attached** — communicates over an internal AWS network bus.
* 💿 **Persistent** — data survives instance termination by default.
* 🔒 **AZ-Locked** — bound strictly to the Availability Zone it was created in.
* 🔄 **Detachable** — can be unmounted from one instance and reattached to another in the same AZ.
* 🔑 **Encrypted** — natively via AES-256 + AWS KMS.

> 💡 An EBS volume in `us-east-1a` **cannot** be directly attached to an instance in `us-east-1b`. Use snapshot-and-restore to migrate across AZ boundaries.

---

## ⚙️ Delete on Termination Flag

The `DeleteOnTermination` attribute controls what happens to a volume when its instance is destroyed.

### 📌 Default Behaviors

* 🟢 **Root Volume (`/dev/xvda`)** — `DeleteOnTermination` is **ENABLED** by default → deleted on termination.
* 🔴 **Extra Data Volumes** — `DeleteOnTermination` is **DISABLED** by default → preserved on termination.

### ⚠️ Important Rule

❌ You can override this flag via the Console, CLI, or IaC — but always verify it before terminating production instances.

---

## 📸 EBS Snapshots

EBS Snapshots are **point-in-time, incremental backups** stored durably in Amazon S3.

### 📌 Characteristics

* ✅ Can be taken from **live, running volumes** (best practice: freeze writes first).
* 🌍 **Regional resources** — can be copied across regions or AZs.
* 💰 **Incremental billing** — you pay only for changed blocks since the last snapshot.
* 🔐 Snapshots from encrypted volumes are **automatically encrypted**.

### 📌 Advanced Snapshot Features

* 🗃️ **Snapshot Archive** — up to **75% cheaper** for rarely-accessed snapshots. Restore takes **24–72 hours**.
* 🗑️ **Recycle Bin** — catches accidental deletions. Recovery window from **1 day to 1 year**.
* ⚡ **Fast Snapshot Restore (FSR)** — eliminates first-access latency penalty. Costs more.

---

## 🗂️ Amazon Machine Images (AMIs)

An AMI is a **pre-configured template** used to launch identical EC2 instances.

### 📌 AMI Component Plane

* 📋 **Metadata Manifest** — Architecture, virtualization type (HVM), OS kernel.
* 💾 **Block Device Mapping** — Maps EBS snapshots to root and secondary volumes.
* 🔐 **Launch Permissions** — Public / Private / Explicitly Shared with specific accounts.

### 📌 AMI Sources

* 🟢 **Public AMIs** — AWS-provided vanilla base images (Amazon Linux 2, Ubuntu, Windows).
* 🔵 **Private AMIs** — Custom images built and maintained in your own account.
* 🟣 **Marketplace AMIs** — Third-party pre-built application stacks.

💡 Use **EC2 Image Builder** to automate **Golden AMI** pipelines — pre-hardened, patched, and replicated across regions.

---

## 🚀 EC2 Instance Store (Ephemeral Storage)

Instance Store provides **physical NVMe/SSD drives** directly wired to the host motherboard — not network-attached.

### 📌 Characteristics

* ⚡ **Extreme Performance** — millions of IOPS, sub-millisecond latency.
* 💥 **Ephemeral** — data is permanently wiped on stop, termination, or hardware failure.
* ✅ **Survives only** standard OS reboots.
* 🛠️ **Your responsibility** — no automated snapshots or replication from AWS.

### ⚠️ Critical Rule

❌ **Never store stateful, critical, or persistent data on an Instance Store volume.**

### 📌 Ideal Workloads

* 🗄️ High-speed database cache scratch space.
* 📁 Temporary file buffers and session stores.
* 🔄 Stateless replication nodes.

---

## 🎚️ EBS Volume Types

### 📌 Two Hardware Categories

* 💠 **SSD** — Transactional, IOPS-intensive. Can be used as boot volumes.
* 🟤 **HDD** — Throughput-heavy, sequential data. **Cannot be boot volumes.**

### 📌 SSD Volume Types

| Type | Max IOPS | Max Throughput | Key Fact |
| --- | --- | --- | --- |
| **`gp3`** | 16,000 | 1,000 MiB/s | ⭐ **Default choice.** Performance decoupled from size. |
| **`gp2`** | 16,000 | 250 MiB/s | 3 IOPS per GiB. Over-provision disk just to hit IOPS. |
| **`io2 Block Express`** | **256,000** | 4,000 MiB/s | Highest performance. Supports Multi-Attach. 99.999% durability. |
| **`io1`** | 64,000 | 1,000 MiB/s | Legacy high-performance DBs. |

### 📌 HDD Volume Types

| Type | Max IOPS | Max Throughput | Key Fact |
| --- | --- | --- | --- |
| **`st1`** | 500 | 500 MiB/s | Big data, logs, data warehousing. |
| **`sc1`** | 250 | 250 MiB/s | 🟤 **Lowest cost.** Cold archives, infrequent access. |

### ⚠️ Critical Rule

❌ **`st1` and `sc1` cannot be used as boot volumes.**

---

## 🔗 EBS Multi-Attach

Multi-Attach allows a **single `io1` or `io2` volume** to be attached to multiple EC2 instances simultaneously.

### 📌 Characteristics

* ✅ Supports up to **16 instances** at the same time.
* 🔒 Restricted to a **single Availability Zone**.
* ⚠️ Requires a **cluster-aware filesystem** (GFS2, OCFS2) — standard EXT4/XFS will corrupt data.

---

## 🔐 EBS Encryption

EBS encryption uses **AES-256** managed by **AWS KMS**.

### 📌 What Gets Encrypted

* 💿 Data at rest on the physical drive.
* 🌐 Data in flight between EC2 and the volume (internal network bus).
* 📸 All snapshots from encrypted volumes.
* 🔄 All new volumes created from encrypted snapshots.

### 📌 Encrypting an Unencrypted Volume

1. 📸 Create a **snapshot** of the unencrypted volume.
2. 🔑 **Copy the snapshot** with KMS encryption enabled.
3. 💿 **Create a new volume** from the encrypted snapshot.
4. 🔄 **Swap** the old volume for the new encrypted one.

💡 There is **no direct in-place encryption** for existing unencrypted volumes.

---

## 📂 Amazon EFS (Elastic File System)

Amazon EFS is a fully managed, **multi-tenant Network File System (NFS)** using the POSIX standard.

### 📌 Characteristics

* 🌍 **Multi-AZ** — mounts across **thousands of concurrent EC2 instances** region-wide.
* 📈 **Elastic Autoscaling** — grows and shrinks automatically. No pre-provisioning needed.
* 🐧 **Linux-only** — POSIX-compliant. **Does not support Windows AMIs.**
* 💰 **Pay per use** — billed only for exact data stored.
* 🔌 **Protocol** — uses **NFSv4.1** over **TCP Port 2049**.

### ⚠️ Critical Rule

❌ **EFS does not support Windows instances.** Use **Amazon FSx for Windows** instead.

---

## ⚙️ EFS Performance & Storage Tiers

### 📌 Performance Modes

* 🟢 **General Purpose (Default)** — Low latency. Best for web servers, CMS, standard apps.
* 🔵 **Max I/O** — Higher latency, massive aggregate throughput. Best for big data, ML, media.

### 📌 Throughput Modes

* 📊 **Bursting** — Scales with storage size. 1 TB → 50 MiB/s baseline, 100 MiB/s burst.
* ⚡ **Provisioned** — Fixed throughput regardless of storage (e.g., 1 GiB/s).
* 🔄 **Elastic** — Auto-balances for unpredictable loads. Up to 3 GiB/s reads, 1 GiB/s writes.

### 📌 Storage Lifecycle Tiers

* 🟢 **Standard** — Frequently accessed active data.
* 🔵 **Infrequent Access (EFS-IA)** — Lower cost. Small retrieval fee on read.
* 🟤 **Archive** — Cold data, few accesses per year. Up to **50% cheaper** than IA.

---

## 🗃️ EFS Access Points

An **EFS Access Point** is an application-specific gateway for managing user identity and directory isolation within a shared filesystem.

### 📌 Characteristics

* 👤 **POSIX User Enforcement** — Overrides client identity with a fixed `UID`/`GID` profile.
* 🔒 **Directory Root Jailing** — Jails clients into a specific subdirectory as their root (`/`).

💡 Use Access Points to isolate multiple tenant applications sharing a single EFS volume.

---

## 🛡️ EFS Security Layers

### 📌 Security Components

* 🔒 **Network** — Security Groups must allow **TCP Port 2049 (NFSv4)**.
* 🎭 **IAM Resource Policies** — Control `elasticfilesystem:ClientMount` actions.
* 💿 **At-Rest Encryption** — AES-256 via custom KMS master keys.
* 🌐 **In-Transit Encryption** — TLS 1.2 via the **EFS mount helper utility**.

---

## 🏎️ RAID Architectures with EBS

### 📌 RAID Comparison

| Strategy | Fault Tolerance | Primary Goal |
| --- | --- | --- |
| **RAID 0** | ❌ Zero — one failure corrupts all. | Maximize IOPS & Throughput beyond single volume limits. |
| **RAID 1** | ✅ High — survives N-1 mirror loss. | Protect non-replicated data against drive failure. |
| **RAID 10** | ✅ High — survives one mirror set failure. | Enterprise DB backends needing both speed and redundancy. |

### ⚠️ RAID Exam Traps

* 🚨 **Always enable EBS-Optimized** on instances running RAID arrays, or the shared network interface becomes the bottleneck.
* 🚨 **Freeze writes globally** before snapshotting a RAID array — AWS snapshots are per-volume and can capture inconsistent state.

---

## 💾 EC2 Hibernate

### 📌 Power State Comparison

| State | Compute Charges | RAM | EBS | Boot |
| --- | --- | --- | --- | --- |
| **Running** | ✅ Yes | Active | Read-Write | Immediate |
| **Stopped** | 🚫 No | Purged | Preserved | Cold boot |
| **Hibernated** | 🚫 No | **Preserved on disk** | Preserved | ⚡ Ultra-fast warm resume |
| **Terminated** | 🚫 No | Purged | Deleted (if flagged) | Non-restorable |

### 📌 Hibernate Constraints

* 🔐 Root EBS volume **must be encrypted**.
* 🧠 Instance RAM must be **strictly less than 150 GB**.
* ⏱️ Maximum continuous hibernation: **60 days**.
* 🚫 Not supported on bare-metal instance sizes.
* 💿 Root volume must be **EBS** (not Instance Store).

### ⚠️ Critical Rule

❌ **Never hibernate an instance with an unencrypted root volume.** RAM state — including secrets and session tokens — is written to disk in full.

---

## ⚖️ EBS vs. EFS vs. Instance Store

| Factor | EBS | EFS | Instance Store |
| --- | --- | --- | --- |
| **Persistence** | ✅ Permanent | ✅ Permanent | ❌ Ephemeral |
| **Multi-Host Access** | 🚫 Single (Multi-Attach max 16, same AZ) | ✅ Thousands, multi-AZ | 🚫 Single host |
| **AZ Scope** | Single AZ | Multi-AZ | Single AZ |
| **Boot Volume** | ✅ Yes | ❌ No | ❌ No |
| **OS Support** | All OS | Linux only | All OS |
| **Performance** | High | High | 🚀 Extreme |
| **Scaling** | Manual | ✅ Elastic auto | Fixed to instance type |

---

## 🔥 High-Frequency Exam Facts

* 💾 EBS is **AZ-locked** — cannot attach across AZ boundaries.
* 🔄 Use **snapshot + restore** to move EBS data across AZs or regions.
* ⚙️ `DeleteOnTermination` is **enabled** for root volumes, **disabled** for extra volumes by default.
* 📸 EBS Snapshots are **incremental** — billed only for changed blocks.
* 🏛️ Snapshot Archive saves up to **75%** — restoration takes **24–72 hours**.
* ⭐ **`gp3` is the default choice** — always prefer it over `gp2` for decoupled performance.
* 🏆 **`io2 Block Express`** = highest IOPS (256,000), highest durability (99.999%), supports Multi-Attach.
* 🟤 **`st1` and `sc1` cannot be boot volumes.**
* 🔗 **EBS Multi-Attach** = max 16 instances, same AZ, requires cluster-aware filesystem (GFS2/OCFS2).
* 🐧 **EFS is Linux-only** — use FSx for Windows.
* 🌍 EFS supports **thousands of concurrent mounts across multiple AZs**.
* 🔌 EFS uses **NFSv4.1 on TCP Port 2049**.
* 📈 EFS is **pay-per-use** — no pre-provisioning needed.
* 💥 Instance Store data is wiped on **stop, termination, or hardware failure** — survives OS reboot only.
* 🛡️ EBS Encryption covers **at-rest, in-flight, snapshots, and child volumes**.
* 🔐 Default AWS managed key (`aws/ebs`) **cannot be shared** across accounts — use a Custom CMK.
* 💾 **Hibernate requires encrypted root EBS + RAM < 150 GB + max 60-day limit**.
* 🏎️ RAID 0 = max performance, zero fault tolerance. Enable **EBS-Optimized** on the host instance.
* 🗃️ EFS Access Points = POSIX identity override + directory root jailing for multi-tenant isolation.
* 📊 Use **Amazon Data Lifecycle Manager (DLM)** to automate EBS snapshot policies.