# ☁️ AWS Storage Services — SAA-C03 Study Reference Guide

<div align="center">

![AWS](https://img.shields.io/badge/AWS-SAA--C03-orange?style=for-the-badge&logo=amazonaws)
![Snowball](https://img.shields.io/badge/Snowball-Data%20Migration-blue?style=for-the-badge)
![FSx](https://img.shields.io/badge/FSx-File%20Systems-success?style=for-the-badge)
![Storage](https://img.shields.io/badge/Storage-Gateway%20%7C%20DataSync-yellow?style=for-the-badge)

### 🗂️ Snowball • FSx • Storage Gateway • Transfer Family • DataSync

*Comprehensive SAA-C03 study reference covering AWS hybrid storage, edge computing, managed file systems, and data migration services — with exam traps, architecture diagrams, and high-frequency facts.*

</div>

---

# 📖 Overview

This guide covers the AWS storage services most heavily tested in the SAA-C03 exam:

* **AWS Snowball** — physical data migration at petabyte scale
* **Amazon FSx** — managed third-party high-performance file systems
* **AWS Storage Gateway** — hybrid cloud bridge between on-premises and AWS
* **AWS Transfer Family** — managed FTP/SFTP/FTPS file transfer into S3 and EFS
* **AWS DataSync** — automated large-scale data movement and synchronization

These services cover the **Storage domain** of the SAA-C03 exam and appear frequently in scenario-based questions about data migration, hybrid architectures, and file system selection.

---

# 🗂️ AWS Storage — Cloud Native Quick Reference

Before diving into migration and hybrid services, know the three storage categories:

| Category | Services |
| :--- | :--- |
| **Block Storage** | Amazon EBS, EC2 Instance Store |
| **File Storage** | Amazon EFS, Amazon FSx |
| **Object Storage** | Amazon S3, Amazon S3 Glacier |

---

# ❄️ AWS Snowball

## What is Snowball?

AWS Snowball is a highly secure, ruggedized physical device used to migrate large amounts of data **into and out of AWS** without relying on a network connection. Snowball solves the problem of network-based transfers being too slow, too expensive, or too unstable for large datasets.

> **Rule of Thumb:** If transferring data over your network would take more than **one week**, use Snowball instead.



---

## 📊 Network Transfer Time — Why Snowball Exists

| Data Size | 100 Mbps | 1 Gbps | 10 Gbps |
| :--- | :--- | :--- | :--- |
| 10 TB | 12 days | 30 hours | 3 hours |
| 100 TB | 124 days | 12 days | 30 hours |
| 1 PB | 3 years | 124 days | 12 days |

Network transfer challenges that justify Snowball:
* Limited connectivity or bandwidth at the source location
* High network costs for large outbound transfers
* Shared bandwidth that cannot be fully utilized
* Connection instability causing transfer failures and retries

---

## 🖥️ Snowball Edge Device Specifications

![AWS Snowball](assets/AWS%20Snowball.png)

---

## 🏗️ Snowball Architecture Options

### Option 1 — Direct Upload to S3 (Network)
```
Client ──── Internet ────▶ Amazon S3 Bucket
```
Works for small datasets where network transfer time is acceptable.

### Option 2 — Snowball Migration
```
Client ──▶ Snowball Device ──[Physical Shipment]──▶ AWS Facility ──▶ Amazon S3 Bucket
```
Used when network transfer would take too long or is not feasible.

### Option 3 — Edge Computing
```
Edge Location (truck / ship / mine)
        │
        ▼
  Snowball Edge Device
  ┌─────────────────────┐
  │ EC2 Instances       │  ← Process data locally
  │ Lambda Functions    │  ← Run code at the edge
  └─────────────────────┘
        │
        ▼ (when connectivity available)
   Upload results to AWS
```

**Edge computing use cases:**
* Preprocessing data before upload
* Machine learning inference at remote locations
* Media transcoding in the field
* Data collection in disconnected environments

![Snowball Comparison](assets/Snowball%20comparison%20Diagrams.png)

---

## ⚠️ Snowball → Glacier Architecture

> **Exam Trap:** Snowball **cannot** import data directly into Amazon Glacier.

The correct architecture is:

```
Snowball Device ──▶ Amazon S3 ──▶ S3 Lifecycle Policy ──▶ Amazon Glacier
```

You must first land data in S3, then use a **Lifecycle Policy** to transition objects to Glacier.

![Snowball into Glacier](assets/Snowball%20into%20Glacier.png)

---

# 🗄️ Amazon FSx

## What is Amazon FSx?

Amazon FSx is a fully managed service that launches **third-party high-performance file systems** on AWS. AWS handles provisioning, patching, backups, and availability. FSx is chosen when you need a specific file system protocol that EFS does not support.

**Four FSx types:**
* FSx for Windows File Server
* FSx for Lustre
* FSx for NetApp ONTAP
* FSx for OpenZFS

![Amazon FSx Overview](assets/Amazon%20FSx%20%E2%80%93%20Overview.png)

---

## 🪟 FSx for Windows File Server

### What it is
A fully managed **Windows-native** file system — essentially a managed Windows file share in AWS.

### Key Features
* Supports **SMB protocol** and **Windows NTFS**
* Integrates with **Microsoft Active Directory** for authentication
* Supports **ACLs** and **user quotas**
* Supports **Microsoft DFS Namespaces** for consolidating shares
* Can be **mounted on Linux EC2 instances**

### Performance & Scale
* Scales to tens of GB/s throughput
* Millions of IOPS
* Hundreds of PB of data

### Storage Options
| Tier | Use Case |
| :--- | :--- |
| **SSD** | Latency-sensitive workloads — databases, media processing, analytics |
| **HDD** | Broad workloads — home directories, CMS |

### Availability & Access
* Accessible from **on-premises** via VPN or Direct Connect
* Supports **Multi-AZ** configuration for high availability
* Data backed up **daily to Amazon S3**

> **When to choose FSx for Windows:** Any scenario involving Windows workloads, SMB protocol, Active Directory integration, or NTFS requirements.

---

## ⚡ FSx for Lustre

### What it is
A managed **parallel distributed file system** built for massive-scale, high-performance computing. The name comes from **Linux** + **cluster**.

### Use Cases
* Machine Learning training
* High Performance Computing (HPC)
* Video Processing
* Financial Modeling
* Electronic Design Automation (EDA)

### Performance
* Scales to hundreds of GB/s
* Millions of IOPS
* Sub-millisecond latencies

### Storage Options
| Tier | Best For |
| :--- | :--- |
| **SSD** | Low-latency, IOPS-intensive, small/random file operations |
| **HDD** | Throughput-intensive, large/sequential file operations |

### S3 Integration
* Can treat an S3 bucket as a file system — **reads S3 as if it were local storage**
* Writes output back to S3
* Accessible from on-premises via VPN or Direct Connect

### Deployment Options

| Type | Storage | Replication | Speed | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Scratch** | Temporary | Not replicated | 6x burst, 200 MB/s per TiB | Short-term processing, cost optimization |
| **Persistent** | Long-term | Replicated within same AZ | Standard | Long-term processing, sensitive data |

> **Exam Trap:** Scratch File System data is **NOT replicated** — if the file server fails, data is lost. Use Persistent for production.

---

## 🔵 FSx for NetApp ONTAP

### What it is
Managed **NetApp ONTAP** on AWS — a direct lift-and-shift for existing ONTAP or NAS workloads.

### Protocols Supported
* NFS
* SMB
* iSCSI

### Compatibility
Works with Linux, Windows, macOS, VMware Cloud on AWS, Amazon WorkSpaces, AppStream 2.0, EC2, ECS, and EKS.

### Key Features
* **Auto-scaling storage** — shrinks or grows automatically
* Snapshots and replication
* Compression and deduplication
* **Point-in-time instantaneous cloning** — ideal for testing new workloads without copying data

![Amazon FSx for NetApp ONTAP](assets/Amazon%20FSx%20for%20NetApp%20ONTAP.png)

---

## 🟠 FSx for OpenZFS

### What it is
Managed **OpenZFS** file system on AWS — for workloads currently running on ZFS.

### Protocol Supported
* NFS (v3, v4, v4.1, v4.2)

### Compatibility
Works with Linux, Windows, macOS, VMware Cloud on AWS, Amazon WorkSpaces, AppStream 2.0, EC2, ECS, and EKS.

### Performance
* Up to **1,000,000 IOPS**
* Under **0.5 ms latency**

### Key Features
* Snapshots and compression
* Low cost
* **Point-in-time instantaneous cloning**

![Amazon FSx for OpenZFS](assets/Amazon%20FSx%20for%20OpenZFS.png)

---

## 📊 FSx Comparison Table

| Feature | Windows File Server | Lustre | NetApp ONTAP | OpenZFS |
| :--- | :--- | :--- | :--- | :--- |
| Protocol | SMB, NTFS | POSIX / custom | NFS, SMB, iSCSI | NFS |
| Primary Use | Windows workloads | HPC, ML, analytics | Lift ONTAP to cloud | Lift ZFS to cloud |
| Multi-AZ | ✅ Yes | ❌ No | ✅ Yes | ❌ No |
| S3 Integration | ❌ | ✅ Read/Write | ❌ | ❌ |
| AD Integration | ✅ | ❌ | ✅ | ❌ |
| Auto Cloning | ❌ | ❌ | ✅ | ✅ |
| Max IOPS | Millions | Millions | Millions | 1,000,000 |

---

# 🌉 AWS Storage Gateway

## What is Storage Gateway?

AWS Storage Gateway is a **hybrid cloud service** that acts as a bridge between your **on-premises environment** and **AWS cloud storage**. It allows on-premises applications to seamlessly use AWS storage as if it were local.

**Hybrid Cloud Drivers:**
* Long cloud migrations — data must remain on-premises during transition
* Security or compliance requirements mandating local data residency
* Low-latency access to frequently used data (local cache)
* Disaster recovery and backup to cloud

**Use cases:**
* Disaster recovery
* Backup and restore
* Tiered storage (hot on-premises, cold in cloud)
* On-premises cache for low-latency file access

---

## 🗂️ Storage Gateway Types

### 1 — S3 File Gateway

```
On-Premises App
      │ NFS / SMB
      ▼
 S3 File Gateway ──── HTTPS ────▶ Amazon S3
  (Local Cache)                (Standard, IA, Intelligent-Tiering)
                                        │
                               Lifecycle Policy
                                        │
                                        ▼
                                Amazon Glacier
```

* Configured S3 buckets appear as NFS or SMB file shares on-premises
* Most recently used data is **cached locally** for low-latency access
* Supports S3 Standard, S3 Standard-IA, S3 One Zone-IA, S3 Intelligent-Tiering
* Transition to Glacier via **S3 Lifecycle Policy**
* Each File Gateway uses **IAM roles** for bucket access
* SMB protocol supports **Active Directory** authentication

![File Gateway](assets/File%20Gateway.png)

---

### 2 — Volume Gateway

```
On-Premises App Server
      │ iSCSI
      ▼
Volume Gateway ──── HTTPS ────▶ Amazon S3
  (Local Cache)                (EBS Snapshots)
```

* Exposes **block storage** using **iSCSI protocol** backed by S3
* Backed by **EBS snapshots** — can restore on-premises volumes from cloud backups

![Volume Gateway](assets/Volume%20Gateway.png)

| Type | How It Works |
| :--- | :--- |
| **Cached Volumes** | Data lives in S3; most recently accessed data cached locally for low latency |
| **Stored Volumes** | Entire dataset stays on-premises; scheduled backups sent to S3 |

---

### 3 — Tape Gateway

```
Backup Application
      │ iSCSI VTL
      ▼
Tape Gateway ──── HTTPS ────▶ Amazon S3 (Virtual Tapes)
                                        │
                               Archive Process
                                        ▼
                              Amazon Glacier (Archived Tapes)
```

* Replaces physical tape backup infrastructure with a **Virtual Tape Library (VTL)**
* Compatible with existing tape-based backup software (no application changes needed)
* Virtual tapes stored in **S3**; archived tapes stored in **Glacier / Glacier Deep Archive**

![Tape Gateway](assets/Tape%20Gateway.png)

---

## 📊 Storage Gateway Summary

| Gateway Type | On-Premises Interface | Local Cache | AWS Backend |
| :--- | :--- | :--- | :--- |
| **S3 File Gateway** | NFS / SMB | ✅ | S3 (not Glacier direct) |
| **Volume Gateway** | iSCSI (block) | ✅ | S3 + EBS Snapshots |
| **Tape Gateway** | iSCSI VTL | ✅ | S3 (tapes) + Glacier (archives) |

**Deployment:** All three types can run as a **VM** (VMware, Hyper-V, or KVM) on-premises.

---

## ⚠️ Storage Gateway Exam Traps

> **Trap 1:** S3 File Gateway cannot write directly to Glacier — use a Lifecycle Policy on the S3 bucket to transition objects.

> **Trap 2:** Volume Gateway Cached vs. Stored — Cached keeps data in S3 with a local hot cache. Stored keeps ALL data on-premises with S3 used only for backup.

> **Trap 3:** Tape Gateway does NOT change your backup software — it presents a standard iSCSI VTL interface so existing backup tools work without modification.

---

# 📡 AWS Transfer Family

## What is Transfer Family?

AWS Transfer Family is a **fully managed service** for transferring files **into and out of Amazon S3 or Amazon EFS** using standard FTP protocols — without managing any server infrastructure.

## Supported Protocols

| Protocol | Full Name | Security |
| :--- | :--- | :--- |
| **FTP** | File Transfer Protocol | No encryption |
| **FTPS** | FTP over SSL | Encryption in transit |
| **SFTP** | Secure File Transfer Protocol | Encryption + SSH key auth |

## Key Features
* Fully managed — no server provisioning or patching
* Multi-AZ for high availability
* Scalable to any transfer volume
* Pay per **provisioned endpoint per hour** + **data transferred (GB)**

## Authentication & Integration
* Store and manage user credentials within Transfer Family itself
* Or integrate with existing identity providers:
  * Microsoft Active Directory
  * LDAP
  * Okta
  * Amazon Cognito
  * Custom authentication providers

## Common Use Cases
* Sharing files with external partners using SFTP
* Hosting public datasets
* CRM and ERP data exchange
* Partner integrations requiring FTP-based workflows

> **When to choose Transfer Family:** Any scenario mentioning SFTP, FTPS, or FTP file transfers going into S3 or EFS without managing server infrastructure.

---

# 🔄 AWS DataSync

## What is DataSync?

AWS DataSync is a fully managed service that **automates moving large amounts of data** between on-premises storage, other clouds, and AWS storage services — handling scheduling, monitoring, encryption, and integrity verification automatically.

## DataSync Transfer Scenarios

```
┌─────────────────────────────────────────────────────────────────┐
│                      AWS DataSync                               │
│                                                                 │
│  On-Premises / Other Cloud ────[Agent]────▶ AWS Storage               
│  (NFS, SMB, HDFS, S3 API)                  (S3, EFS, FSx)        │
│                                                                  │
│  AWS Storage ──────────────[No Agent]────▶ AWS Storage          │
│  (S3 ────────────────────────────────────▶ EFS, FSx, etc.)      │
└─────────────────────────────────────────────────────────────────┘
```

![AWS DataSync Transfer](assets/AWS%20DataSync%20Transfer%20between%20AWS%20storage%20services.png)

## Agent Requirement

| Scenario | Agent Required? |
| :--- | :--- |
| On-premises → AWS | ✅ Yes — DataSync agent installed on-premises |
| Other cloud → AWS | ✅ Yes — DataSync agent installed at source |
| AWS → AWS | ❌ No — agent not required |

## Supported Destinations

| Destination | Notes |
| :--- | :--- |
| Amazon S3 | All storage classes including Glacier |
| Amazon EFS | |
| Amazon FSx | Windows, Lustre, NetApp ONTAP, OpenZFS |

## Key Features

| Feature | Detail |
| :--- | :--- |
| **Scheduling** | Hourly, daily, or weekly replication tasks |
| **Performance** | One agent task uses up to 10 Gbps; bandwidth limits configurable |
| **Data Integrity** | File permissions and metadata preserved (NFS POSIX, SMB) |
| **Agent Deployment** | On-premises VM, or pre-installed on AWS Snowcone |

## DataSync vs. Storage Gateway

| Aspect | DataSync | Storage Gateway |
| :--- | :--- | :--- |
| Primary Purpose | Large-scale **migration** or scheduled **sync** | Ongoing **hybrid access** to cloud storage |
| On-Premises Access | One-time or scheduled transfers | Continuous local file/block/tape access |
| Local Cache | ❌ No | ✅ Yes |
| Use Case | Move data to AWS at scale | Extend on-premises storage to AWS |

> **Exam Trap:** DataSync is for **moving** data. Storage Gateway is for **accessing** cloud storage from on-premises as if it were local. They are often confused in scenario questions.

---

# 🧠 High-Frequency Exam Facts

* Snowball **cannot** import directly to Glacier — must go through **S3 → Lifecycle Policy → Glacier**
* FSx for Lustre **Scratch** = temporary, no replication, high burst speed
* FSx for Lustre **Persistent** = replicated within same AZ, for sensitive/long-term data
* FSx for Lustre can **read from and write to S3** as a file system
* FSx for Windows supports **SMB + NTFS + Active Directory + DFS Namespaces**
* Storage Gateway S3 File Gateway **cannot write directly to Glacier** — use Lifecycle Policy
* Volume Gateway **Cached** = data in S3, hot data cached locally
* Volume Gateway **Stored** = all data on-premises, S3 used only for backups
* Tape Gateway presents **iSCSI VTL interface** — existing backup software unchanged
* Transfer Family supports **FTP, FTPS, SFTP** into **S3 or EFS**
* DataSync AWS → AWS transfers **do not require an agent**
* DataSync preserves **file permissions and metadata**
* Use **Snowball** when network transfer would take **more than 1 week**
* FSx for NetApp ONTAP and OpenZFS both support **point-in-time instantaneous cloning**

---

# 📊 Full Storage Services Comparison

| Service | Protocol | Use Case | On-Premises? |
| :--- | :--- | :--- | :--- |
| **FSx for Windows** | SMB, NTFS | Windows workloads, AD integration | Via VPN/DX |
| **FSx for Lustre** | POSIX | HPC, ML, large-scale analytics | Via VPN/DX |
| **FSx for ONTAP** | NFS, SMB, iSCSI | Lift ONTAP/NAS to AWS | Via VPN/DX |
| **FSx for OpenZFS** | NFS | Lift ZFS workloads to AWS | Via VPN/DX |
| **S3 File Gateway** | NFS, SMB | Hybrid S3 access with local cache | ✅ |
| **Volume Gateway** | iSCSI | Block storage with cloud backup | ✅ |
| **Tape Gateway** | iSCSI VTL | Replace physical tapes with cloud | ✅ |
| **Transfer Family** | FTP/FTPS/SFTP | Partner file exchange into S3/EFS | ✅ |
| **DataSync** | NFS, SMB, S3 API | Bulk migration and scheduled sync | ✅ (agent) |
| **Snowball** | Physical device | Petabyte-scale migration, edge computing | ✅ |

---

# 📚 Conclusion

This guide covers the AWS storage migration and hybrid connectivity services most commonly tested in SAA-C03. The key skill the exam tests is **choosing the right service for the scenario**:

* **Massive data to migrate with slow/no network?** → Snowball
* **Windows file shares with Active Directory?** → FSx for Windows
* **HPC or ML workloads needing high throughput?** → FSx for Lustre
* **Lift existing ONTAP/NAS/ZFS?** → FSx for ONTAP or OpenZFS
* **On-premises apps needing cloud storage via NFS/SMB?** → S3 File Gateway
* **On-premises block storage with cloud backup?** → Volume Gateway
* **Replace physical tape backups?** → Tape Gateway
* **SFTP/FTP partner file exchange into S3?** → Transfer Family
* **Automate large-scale data sync to AWS?** → DataSync

---

<div align="center">

### ☁️ AWS SAA-C03 Storage Domain Reference

**Snowball • FSx • Storage Gateway • Transfer Family • DataSync • Hybrid Cloud • Data Migration**

</div>