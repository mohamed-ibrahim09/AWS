# 🪣 Amazon S3 — Complete Reference Guide

<div align="center">

![AWS](https://img.shields.io/badge/AWS-Amazon%20S3-orange?style=for-the-badge&logo=amazonaws)
![Storage](https://img.shields.io/badge/Storage-Object%20Storage-blue?style=for-the-badge)
![S3](https://img.shields.io/badge/S3-Infinitely%20Scalable-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Comprehensive-success?style=for-the-badge)

### ⚡ Amazon Simple Storage Service — Engineering Blueprint

*A deep-dive technical reference covering S3 buckets, objects, security models, versioning, replication, all storage classes, static website hosting, and exam-critical traps for SAA-C03.*

</div>

---

## 📖 Overview

This document covers Amazon S3 from first principles through to production-grade storage architecture. It is structured to build understanding layer by layer — starting with what S3 is and how it stores data, then how to secure it, then how to configure advanced features like versioning and replication, and finally all storage classes with a side-by-side comparison.

---

## 🌍 1. What is Amazon S3?

**Amazon Simple Storage Service (S3)** is AWS's core object storage service, marketed as **"infinitely scaling"** storage. It is one of the fundamental building blocks of AWS — nearly every AWS architecture touches S3 in some way.

```
  You have a file  →  Upload it to S3  →  It lives forever at a unique URL
  You have a video →  Store in S3      →  Stream it globally via CloudFront
  You need a DB backup → Dump to S3   →  Replicate it across regions automatically
```

> **Why "backbone of AWS"?** Almost every AWS service integrates with S3 — Athena queries S3, Lambda reads from S3, EMR processes S3 data, CloudTrail logs land in S3. If you understand S3 deeply, everything else in AWS becomes easier to reason about.

### Real-World Scale Examples

| Company | S3 Usage |
| --- | --- |
| **Nasdaq** | Stores 7 years of financial data in S3 Glacier |
| **Sysco** | Runs analytics on S3 data to extract business insights |

---

## 📋 2. S3 Use Cases

S3 is not just "file storage" — it covers a wide range of architectural patterns:

```
  ┌─────────────────────────────────────────────────────────┐
  │                    S3 USE CASES                          │
  ├──────────────────────┬──────────────────────────────────┤
  │  Backup & Storage    │  Primary backup target            │
  │  Disaster Recovery   │  Cross-region data safety net     │
  │  Archive             │  Long-term cold storage           │
  │  Hybrid Cloud        │  Bridge between on-prem and AWS   │
  │  App Hosting         │  Assets, configs, build artifacts │
  │  Media Hosting       │  Images, video, audio at scale    │
  │  Data Lakes          │  Big data analytics foundation    │
  │  Software Delivery   │  Distribute builds / packages     │
  │  Static Websites     │  Serve HTML/CSS/JS directly       │
  └──────────────────────┴──────────────────────────────────┘
```

---

## 🪣 3. Buckets — S3's Container Unit

A **bucket** is the top-level container in S3. Think of it as a directory, but with global scope rules.

```
  AWS Cloud
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │   S3 (Global Service View)                               │
  │   ┌────────────────────────────────────────────────┐     │
  │   │   Bucket: my-company-assets  (us-east-1)       │     │
  │   │   ├── images/logo.png                          │     │
  │   │   ├── videos/intro.mp4                         │     │
  │   │   └── docs/readme.txt                          │     │
  │   └────────────────────────────────────────────────┘     │
  │   ┌────────────────────────────────────────────────┐     │
  │   │   Bucket: my-backups  (eu-west-1)              │     │
  │   │   └── db-snapshot-2024.sql                     │     │
  │   └────────────────────────────────────────────────┘     │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

### Key Bucket Facts

| Property | Detail |
| --- | --- |
| **Region-scoped** | Buckets are created in a specific AWS region, even though the S3 console looks global |
| **Namespace: Global** | Bucket names are globally unique across **all AWS accounts and all regions** |
| **Max Buckets** | 100 per account by default (can request increase) |

### Bucket Naming Rules

```
  ✅ ALLOWED                          ❌ NOT ALLOWED
  ─────────────────────────────────   ─────────────────────────────────────
  my-company-bucket                   My_Company_Bucket   (uppercase/underscore)
  projectassets2024                   192.168.1.1         (IP address format)
  logs-prod-eu                        xn--mybucket        (starts with xn--)
  devops-artifacts                    my-bucket-s3alias   (ends with -s3alias)
  
  Rules:
  • 3–63 characters long
  • Lowercase letters, numbers, and hyphens only
  • Must start with a lowercase letter or number
  • Must NOT start with: xn--
  • Must NOT end with: -s3alias
  • Must NOT be formatted as an IP address
```

---

## 📦 4. Objects — What You Actually Store

An **object** is any file (or data) you store inside a bucket. Every object is uniquely identified by its **key**.

### The Key — Full Object Path

```
  s3://my-bucket/my_folder1/another_folder/my_file.txt
  │              │                          │
  │              │                          └── Object Name
  │              └───────────────────────────── Prefix (simulated folder path)
  └──────────────────────────────────────────── Bucket Name

  KEY = prefix + object name
      = "my_folder1/another_folder/" + "my_file.txt"
      = "my_folder1/another_folder/my_file.txt"
```

> **Critical concept:** S3 has **no real directory structure**. It is a flat key-value store. The slash (`/`) in a key is just a character — the S3 console renders folders visually, but they don't exist as actual objects. This matters for prefix-based IAM policies and lifecycle rules.

### Object Properties

| Property | Detail |
| --- | --- |
| **Max Object Size** | **5 TB** (5,000 GB) |
| **Max Single PUT Upload** | **5 GB** — above this, you MUST use Multi-Part Upload |
| **Multi-Part Upload** | Required for objects > 5 GB; recommended for objects > 100 MB for speed/reliability |
| **Metadata** | Key/value pairs (text) — system-generated or user-defined |
| **Tags** | Unicode key/value pairs, up to 10 per object — useful for security and lifecycle rules |
| **Version ID** | Assigned when S3 Versioning is enabled on the bucket |

---

## 🔒 5. S3 Security Model

S3 has two layers of access control: **user-based** (IAM) and **resource-based** (bucket/object policies and ACLs).

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                   S3 ACCESS DECISION LOGIC                        │
  │                                                                  │
  │   IAM Principal attempts to access S3 object                     │
  │                       │                                          │
  │                       ▼                                          │
  │        ┌──────────────────────────────┐                          │
  │        │  Is there an EXPLICIT DENY?  │──── YES ──► ❌ DENIED    │
  │        └──────────────────────────────┘                          │
  │                       │ NO                                       │
  │                       ▼                                          │
  │   ┌─────────────────────────────────────────────────┐            │
  │   │  IAM Policy ALLOWS  OR  Resource Policy ALLOWS? │            │
  │   └─────────────────────────────────────────────────┘            │
  │          │ YES                          │ NO                      │
  │          ▼                              ▼                         │
  │      ✅ ALLOWED                     ❌ DENIED                    │
  └──────────────────────────────────────────────────────────────────┘
```

### Access Control Methods

| Method | Type | Scope | Notes |
| --- | --- | --- | --- |
| **IAM Policies** | User-Based | Per IAM user/role/group | Controls which S3 API calls the identity can make |
| **Bucket Policies** | Resource-Based | Entire bucket | JSON-based; supports cross-account access; most commonly used |
| **Object ACL** | Resource-Based | Per object | Fine-grained; can be disabled |
| **Bucket ACL** | Resource-Based | Entire bucket | Less common; can be disabled |
| **Encryption** | Data Protection | Per object | SSE-S3, SSE-KMS, SSE-C, or client-side |

### Bucket Policies (JSON)

Bucket Policies are the primary mechanism for controlling access at scale. They are JSON-based and follow the same IAM policy structure.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::examplebucket/*"]
    }
  ]
}
```

| Field | Purpose |
| --- | --- |
| `Effect` | `Allow` or `Deny` |
| `Principal` | Who this applies to — `"*"` means everyone (public), or specify an account/role ARN |
| `Action` | The S3 API calls being allowed or denied (e.g., `s3:GetObject`, `s3:PutObject`) |
| `Resource` | The bucket or objects this applies to — use `/*` to include all objects |

### Common Bucket Policy Use Cases

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                  BUCKET POLICY SCENARIOS                          │
  ├──────────────────────────────────────────────────────────────────┤
  │                                                                  │
  │  1. PUBLIC ACCESS (Static Website / CDN origin)                  │
  │     Principal: "*"  |  Action: s3:GetObject  |  Effect: Allow    │
  │                                                                  │
  │  2. FORCE ENCRYPTION AT UPLOAD                                   │
  │     Deny PutObject unless header                                 │
  │     x-amz-server-side-encryption is present                      │
  │                                                                  │
  │  3. CROSS-ACCOUNT ACCESS                                         │
  │     Principal: "arn:aws:iam::OTHER_ACCOUNT_ID:root"              │
  │     Action: s3:GetObject | Effect: Allow                         │
  │                                                                  │
  └──────────────────────────────────────────────────────────────────┘
```

### Access Scenarios — Visual Reference

```
  SCENARIO 1: Anonymous Public Access
  ─────────────────────────────────────
  Internet User ──► S3 Bucket
                    (Bucket Policy: Principal=*, Effect=Allow)

  SCENARIO 2: IAM User Access
  ─────────────────────────────────────
  IAM User ──► IAM Policy grants s3:* ──► S3 Bucket

  SCENARIO 3: EC2 Instance Access (Best Practice)
  ─────────────────────────────────────
  EC2 Instance ──► Instance Role (IAM Role) ──► S3 Bucket
  ✅ NEVER put credentials on EC2 — always use IAM Roles

  SCENARIO 4: Cross-Account Access
  ─────────────────────────────────────
  IAM User (Account B) ──► Bucket Policy (Account A allows Account B) ──► S3 Bucket
```

### Block Public Access Settings

These settings act as a final override — they block public access regardless of what bucket policies or ACLs say.

```
  ┌──────────────────────────────────────────────────────────────────┐
  │             BLOCK PUBLIC ACCESS SETTINGS                          │
  ├──────────────────────────────────────────────────────────────────┤
  │  ✅ Block public access via new ACLs                             │
  │  ✅ Block public access via any ACLs                             │
  │  ✅ Block public access via new bucket/access point policies     │
  │  ✅ Block public and cross-account access via any public policy  │
  ├──────────────────────────────────────────────────────────────────┤
  │  PURPOSE: Prevent accidental company data leaks                  │
  │  SCOPE: Can be set at the account level (applies to all buckets) │
  │  RULE: If the bucket should NEVER be public → leave ALL ON       │
  └──────────────────────────────────────────────────────────────────┘
```

> **Exam note:** Block Public Access settings override bucket policies. Even with a `Principal: "*"` Allow policy, if Block Public Access is on, the request is denied.

---

## 🌐 6. Static Website Hosting

S3 can serve static websites directly to the internet — no server, no EC2, no backend required.

```
  Browser ──► http://bucket-name.s3-website-us-east-1.amazonaws.com ──► S3 Bucket
                                                                         (HTML/CSS/JS)
```

### URL Formats (Region-Dependent)

```
  Format 1:  http://<bucket-name>.s3-website-<region>.amazonaws.com
  Format 2:  http://<bucket-name>.s3-website.<region>.amazonaws.com

  Example:   http://my-portfolio.s3-website-us-east-1.amazonaws.com
```

### Requirements for Static Hosting

| Requirement | Setting |
| --- | --- |
| Static Website Hosting | Must be **enabled** in the bucket properties |
| Index Document | Must specify (e.g., `index.html`) |
| Error Document | Should specify (e.g., `error.html`) |
| Bucket Policy | Must allow `s3:GetObject` for `Principal: "*"` |
| Block Public Access | Must be **disabled** — otherwise you get `403 Forbidden` |

> **403 Forbidden on a static website?** The bucket policy does not allow public reads. Make sure Block Public Access is off AND a public bucket policy with `s3:GetObject` is attached.

---

## 🔁 7. Versioning

Versioning allows S3 to keep multiple versions of an object — every overwrite or delete creates a new version instead of permanently destroying data.

```
  Without Versioning:
  ──────────────────
  Upload v1 of logo.png  →  [logo.png : v1]
  Upload v2 of logo.png  →  [logo.png : v2]   ← v1 is GONE FOREVER

  With Versioning Enabled:
  ──────────────────────────
  Upload v1 of logo.png  →  [logo.png : v1]
  Upload v2 of logo.png  →  [logo.png : v1] [logo.png : v2]
  Delete logo.png        →  [logo.png : v1] [logo.png : v2] [logo.png : delete marker]
  Restore: delete the delete marker → v2 is live again ✅
```

### Versioning Key Facts

| Fact | Detail |
| --- | --- |
| **Enabled at** | Bucket level — applies to all objects in the bucket |
| **Pre-versioning objects** | Files uploaded before versioning was enabled get version `null` |
| **Suspending versioning** | Does NOT delete existing versions — just stops creating new ones |
| **Delete behavior** | A regular delete adds a **delete marker** — the object is not truly removed |
| **Permanent delete** | Must delete a specific version ID to permanently remove a version |

### Why Use Versioning?

```
  ✅ Protection against accidental deletes  →  Restore from any prior version
  ✅ Rollback on bad deployments            →  Revert a config file to last known good
  ✅ Audit trail                            →  See every version of every object
  ✅ Required for S3 Replication            →  Cross-Region Replication needs versioning
```

---

## 🔄 8. Replication — CRR & SRR

S3 Replication automatically copies objects from a **source bucket** to a **destination bucket**, either in the same or different region.

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    S3 REPLICATION TYPES                           │
  ├───────────────────────────┬──────────────────────────────────────┤
  │  CRR (Cross-Region)       │  SRR (Same-Region)                   │
  ├───────────────────────────┼──────────────────────────────────────┤
  │  Source → different       │  Source → same region                │
  │  AWS region               │  (different bucket)                  │
  ├───────────────────────────┼──────────────────────────────────────┤
  │  Use cases:               │  Use cases:                          │
  │  • Compliance             │  • Log aggregation                   │
  │  • Lower latency          │  • Prod → Test live replication      │
  │  • Cross-account backup   │  • Data sovereignty (stay in region) │
  └───────────────────────────┴──────────────────────────────────────┘
```

### Replication Requirements

| Requirement | Detail |
| --- | --- |
| **Versioning** | Must be enabled on **both** source and destination buckets |
| **IAM Permissions** | S3 must be granted an IAM role with permission to replicate |
| **Copying mode** | Asynchronous — there is a small delay, not real-time |
| **Accounts** | Buckets can be in different AWS accounts |

### Replication Behavior Rules

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    REPLICATION RULES                              │
  ├──────────────────────────────────────────────────────────────────┤
  │                                                                  │
  │  NEW OBJECTS: Replicated automatically after replication         │
  │  is enabled.                                                     │
  │                                                                  │
  │  EXISTING OBJECTS: NOT replicated by default.                    │
  │  → Use S3 Batch Replication for existing objects                 │
  │  → Also handles objects that previously failed replication       │
  │                                                                  │
  │  DELETE MARKERS: Optional — you choose whether to replicate      │
  │  them to the destination.                                        │
  │                                                                  │
  │  DELETE BY VERSION ID: NOT replicated. This protects you        │
  │  from malicious deletes propagating to your backup bucket.       │
  │                                                                  │
  │  CHAINING: NOT supported.                                        │
  │  Bucket 1 → Bucket 2 → Bucket 3                                  │
  │  Objects in Bucket 1 are NOT replicated to Bucket 3.            │
  │  Only Bucket 2 gets them.                                        │
  │                                                                  │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 📦 9. S3 Storage Classes

S3 offers multiple storage classes, each optimized for a different data access pattern. You can assign a class at upload time or transition objects automatically using **Lifecycle Rules**.

```
  ┌─────────────────────────────────────────────────────────────────────────────┐
  │                       S3 STORAGE CLASS SPECTRUM                              │
  │                                                                             │
  │  ACCESS FREQUENCY:   Frequent ◄──────────────────────────────► Rare        │
  │                                                                             │
  │  Standard ──► Intelligent-Tiering ──► Standard-IA ──► One Zone-IA          │
  │                                                                             │
  │                   ──► Glacier Instant ──► Glacier Flexible ──► Deep Archive │
  │                                                                             │
  │  COST:             High ◄──────────────────────────────────────► Low        │
  │  RETRIEVAL SPEED:  Fast ◄──────────────────────────────────────► Slow       │
  └─────────────────────────────────────────────────────────────────────────────┘
```

### Durability & Availability (Applies to All Classes)

| Metric | Value |
| --- | --- |
| **Durability** | **99.999999999% (11 nines)** — same for all storage classes |
| **What it means** | If you store 10,000,000 objects, you can expect to lose 1 object every **10,000 years** |
| **Availability** | Varies by class — measures how readily accessible the service is |

---

### 9.1 S3 Standard — General Purpose

The default. Use for anything accessed regularly.

```
  Availability:    99.99%
  AZs:             ≥ 3
  Min Duration:    None
  Retrieval Fee:   None
  Latency:         Milliseconds
```

**Use cases:** Big Data analytics, mobile apps, gaming, content distribution, active data

---

### 9.2 S3 Standard-IA (Infrequent Access)

For data accessed less than once a month but needed quickly when accessed.

```
  Availability:    99.9%
  AZs:             ≥ 3
  Min Duration:    30 days
  Min Object Size: 128 KB
  Retrieval Fee:   Yes (per GB)
  Latency:         Milliseconds
```

**Use cases:** Disaster recovery, backups that are rarely read

---

### 9.3 S3 One Zone-IA

Same as Standard-IA but stored in **a single Availability Zone** — cheaper but less resilient.

```
  Availability:    99.5%   ← LOWER because single AZ
  AZs:             1       ← Data is LOST if the AZ is destroyed
  Durability:      99.9999999999% (within that one AZ)
  Min Duration:    30 days
  Min Object Size: 128 KB
  Retrieval Fee:   Yes (per GB)
```

**Use cases:** Secondary backup copies of on-premises data, data that can be recreated if lost

> **Exam warning:** One Zone-IA data is permanently lost if the AZ is destroyed. Never use this for data you cannot recreate.

---

### 9.4 S3 Glacier Instant Retrieval

Glacier pricing with millisecond access — for data accessed roughly once per quarter.

```
  Availability:    99.9%
  AZs:             ≥ 3
  Min Duration:    90 days
  Min Object Size: 128 KB
  Retrieval:       Milliseconds (like S3 Standard)
  Retrieval Fee:   Yes (per GB)
```

**Use cases:** Medical images, news media assets — accessed rarely but must be available instantly when needed

---

### 9.5 S3 Glacier Flexible Retrieval

The classic Glacier. Designed for archives where you can wait hours for retrieval.

```
  Availability:    99.99%
  AZs:             ≥ 3
  Min Duration:    90 days
  Min Object Size: 40 KB
  Retrieval Tiers:
    ├── Expedited:  1–5 minutes
    ├── Standard:   3–5 hours
    └── Bulk:       5–12 hours  (FREE)
```

**Use cases:** Long-term backups, compliance archives, audit logs

---

### 9.6 S3 Glacier Deep Archive

Lowest cost storage in all of AWS. For data that must be kept but will almost never be accessed.

```
  Availability:    99.99%
  AZs:             ≥ 3
  Min Duration:    180 days   ← Longest minimum
  Min Object Size: 40 KB
  Retrieval Tiers:
    ├── Standard:  12 hours
    └── Bulk:      48 hours
  Cost:            ~$0.00099/GB/month  ← Cheapest in AWS
```

**Use cases:** Regulatory compliance (Nasdaq 7-year retention), legal records, historical archives

---

### 9.7 S3 Intelligent-Tiering

Automatically moves objects between tiers based on actual usage patterns. No retrieval charges — you pay a small monitoring fee instead.

```
  ┌──────────────────────────────────────────────────────────────────┐
  │               INTELLIGENT-TIERING TIERS                           │
  ├──────────────────────────────────────────────────────────────────┤
  │  Tier                        │ Trigger            │ Automatic?   │
  ├──────────────────────────────┼────────────────────┼──────────────┤
  │  Frequent Access             │ Default (active)   │ ✅ Yes       │
  │  Infrequent Access           │ Not accessed 30d   │ ✅ Yes       │
  │  Archive Instant Access      │ Not accessed 90d   │ ✅ Yes       │
  │  Archive Access              │ 90–700+ days       │ ⚙️  Optional │
  │  Deep Archive Access         │ 180–700+ days      │ ⚙️  Optional │
  └──────────────────────────────┴────────────────────┴──────────────┘

  Monitoring fee: $0.0025 per 1,000 objects/month
  Retrieval fee:  NONE ← Key differentiator
```

**Use cases:** Unknown or unpredictable access patterns, mixed workloads

---

### 9.8 S3 Express One Zone ⚡ (High-Performance Class)

A specialized class for latency-sensitive, high-throughput workloads. Newer addition to the S3 family.

```
  Storage Type:    Directory Bucket (single AZ)
  Throughput:      100,000s of requests per second
  Latency:         Single-digit milliseconds
  Performance:     Up to 10x faster than S3 Standard
  Cost Savings:    Up to 50% lower costs vs Standard
  Durability:      99.999999999% (11 nines)
  Availability:    99.95%
```

**Best integrated with:** SageMaker Model Training, Amazon Athena, EMR, AWS Glue

**Use cases:** AI/ML training data, financial modeling, media processing, HPC, latency-sensitive apps

---

## 📊 10. Storage Class Comparison

### Availability & Architecture

| | Standard | Intelligent-Tiering | Standard-IA | One Zone-IA | Glacier Instant | Glacier Flexible | Deep Archive |
|---|---|---|---|---|---|---|---|
| **Availability** | 99.99% | 99.9% | 99.9% | 99.5% | 99.9% | 99.99% | 99.99% |
| **Availability SLA** | 99.9% | 99% | 99% | 99% | 99% | 99.9% | 99.9% |
| **AZs** | ≥ 3 | ≥ 3 | ≥ 3 | **1** | ≥ 3 | ≥ 3 | ≥ 3 |
| **Min Duration** | None | None | 30 days | 30 days | 90 days | 90 days | **180 days** |
| **Min Object Size** | None | None | 128 KB | 128 KB | 128 KB | 40 KB | 40 KB |
| **Retrieval Fee** | None | None | Per GB | Per GB | Per GB | Per GB | Per GB |

### Pricing (us-east-1)

| | Standard | Intelligent-Tiering | Standard-IA | One Zone-IA | Glacier Instant | Glacier Flexible | Deep Archive |
|---|---|---|---|---|---|---|---|
| **Storage ($/GB/mo)** | $0.023 | $0.0025–$0.023 | $0.0125 | $0.01 | $0.004 | $0.0036 | **$0.00099** |
| **GET (per 1k req)** | $0.0004 | $0.0004 | $0.001 | $0.001 | $0.01 | $0.0004 | $0.0004 |
| **POST (per 1k req)** | $0.005 | $0.005 | $0.01 | $0.01 | $0.02 | $0.03 | $0.05 |
| **Retrieval Time** | Instant | Instant | Instant | Instant | Instant | 1min–12hr | 12–48hr |
| **Monitoring (per 1k)** | N/A | $0.0025 | N/A | N/A | N/A | N/A | N/A |

---

## 🚨 11. Exam Traps & Common Mistakes

### 🚨 Trap 1: S3 is Global but Buckets are Regional

* **Question Pattern:** You create an S3 bucket in the AWS console without selecting a region. Where is it created?
* **Wrong Answer:** S3 is a global service, so the bucket has no region.
* **Correct Answer:** S3 has a **global namespace** (bucket names are globally unique), but buckets are **created in a specific AWS region**. The console may look global but every bucket lives in exactly one region.
* **Reasoning:** Data sovereignty, latency, and replication all depend on region. When asked about where data is stored, the answer is always the specific region of the bucket.

---

### 🚨 Trap 2: No Real Directories in S3

* **Question Pattern:** You have a key `s3://my-bucket/logs/2024/app.log`. What is the directory structure?
* **Wrong Answer:** There is a folder `logs` containing a folder `2024` containing `app.log`.
* **Correct Answer:** There are **no directories**. The key is `logs/2024/app.log` — a flat string. The slashes are just characters in the key name, not actual folder separators.
* **Reasoning:** This matters for IAM resource ARNs, lifecycle rules, and prefix-based queries. Always think in terms of **key prefixes**, not folder paths.

---

### 🚨 Trap 3: Multi-Part Upload Threshold

* **Question Pattern:** You need to upload a 6 GB file to S3. What upload method must you use?
* **Wrong Answer:** Standard PUT upload.
* **Correct Answer:** **Multi-Part Upload** is required for objects larger than 5 GB. A single PUT can only handle up to 5 GB.
* **Reasoning:** Max single PUT = 5 GB. Max object size = 5 TB. Multi-Part Upload is required in between. AWS also recommends Multi-Part for anything over 100 MB for performance and retry resilience.

---

### 🚨 Trap 4: Versioning Null Objects

* **Question Pattern:** Versioning is enabled on a bucket that already contains files. What is the Version ID of the existing files?
* **Wrong Answer:** They get assigned version 1 automatically.
* **Correct Answer:** Pre-existing objects get a Version ID of **`null`**. Only objects uploaded or modified after versioning is enabled get real version IDs.
* **Reasoning:** S3 cannot retroactively version data it didn't track. The null version is a real, retrievable version — you can delete it specifically using its null version ID.

---

### 🚨 Trap 5: Suspending Versioning Does NOT Delete Versions

* **Question Pattern:** You suspend versioning on a bucket to stop accumulating version history. What happens to existing versions?
* **Wrong Answer:** Existing versions are deleted.
* **Correct Answer:** Suspending versioning **does not delete** any existing versions. They remain in the bucket and continue to incur storage costs.
* **Reasoning:** To actually remove old versions, you must explicitly delete them or configure a Lifecycle Rule to expire non-current versions.

---

### 🚨 Trap 6: Block Public Access Overrides Bucket Policy

* **Question Pattern:** You add a bucket policy allowing `Principal: "*"` but the website still returns 403. Why?
* **Wrong Answer:** The bucket policy syntax is wrong.
* **Correct Answer:** **Block Public Access is still enabled.** Even a correctly written public bucket policy is overridden by Block Public Access settings.
* **Reasoning:** Block Public Access is a safety net that overrides everything. For static hosting, Block Public Access must be turned off AND the bucket policy must allow public reads.

---

### 🚨 Trap 7: EC2 Access to S3 — Never Use Access Keys

* **Question Pattern:** An EC2 instance needs to read from S3. What is the correct approach?
* **Wrong Answer:** Store AWS access keys in the EC2 instance's environment variables or `~/.aws/credentials`.
* **Correct Answer:** Attach an **IAM Role** to the EC2 instance. The role should have the necessary S3 permissions.
* **Reasoning:** Hardcoded credentials on EC2 are a critical security vulnerability. IAM Roles provide temporary credentials that rotate automatically. This is always the correct answer for EC2-to-AWS-service access.

---

### 🚨 Trap 8: CRR Replication — Only New Objects

* **Question Pattern:** You enable Cross-Region Replication on a bucket that already has 1 million objects. Are those objects replicated?
* **Wrong Answer:** Yes, replication automatically copies all existing objects.
* **Correct Answer:** **No.** Replication only applies to objects uploaded **after** replication is enabled. Existing objects are NOT replicated.
* **Reasoning:** To replicate existing objects, you must run an **S3 Batch Replication** job explicitly.

---

### 🚨 Trap 9: Replication Delete Behavior

* **Question Pattern:** A delete operation is performed on an object in a source bucket with CRR enabled. Is the delete replicated?
* **Wrong Answer:** Yes, deletes are always replicated.
* **Correct Answer:** It depends. **Delete markers** can optionally be replicated (configurable). **Version ID deletes** (permanent deletes of a specific version) are **never** replicated — this protects against malicious deletes propagating to your backup.
* **Reasoning:** AWS deliberately decouples delete propagation to prevent an attacker who deletes your source from also wiping your backup.

---

### 🚨 Trap 10: Glacier Deep Archive Minimum Duration

* **Question Pattern:** You store a 1 KB file in Glacier Deep Archive for 30 days then delete it. How many days are you billed for?
* **Wrong Answer:** 30 days.
* **Correct Answer:** **180 days** — the minimum storage duration for Glacier Deep Archive.
* **Reasoning:** Glacier classes enforce minimum storage durations. If you delete before the minimum, you are billed for the remaining minimum duration. This catches people who try to temporarily archive data cheaply.

---

### 🚨 Trap 11: One Zone-IA Data Loss

* **Question Pattern:** You store secondary backup copies in S3 One Zone-IA. The AZ experiences a catastrophic failure. What happens to your data?
* **Wrong Answer:** S3's 11-nines durability protects it.
* **Correct Answer:** The data is **permanently lost**. One Zone-IA is only durable within its single AZ. If that AZ is destroyed, there is no copy.
* **Reasoning:** The 11-nines durability of One Zone-IA applies within the AZ, not across AZs. Only use this class for data you can recreate or that is genuinely a secondary backup.

---

## ✅ 12. Production Best Practices

### Bucket Configuration
* **Always enable versioning on critical buckets** — the storage cost is minimal compared to the recovery value when someone accidentally deletes a file or uploads corrupted data.
* **Leave Block Public Access ON by default** — only disable it for intentionally public buckets (static websites, public assets). The default state should be private.
* **Use meaningful, globally-unique bucket naming conventions** — adopt a pattern like `<company>-<environment>-<purpose>-<region>` (e.g., `acme-prod-assets-us-east-1`) to avoid naming conflicts and make buckets self-documenting.

### Security
* **Never attach access keys to EC2 for S3 access** — always use IAM Roles. This applies to Lambda, ECS tasks, and any compute resource that needs S3 access.
* **Use bucket policies for cross-account access, never share IAM users** — bucket policies with specific account/role ARNs in the Principal field are the correct and auditable pattern.
* **Force encryption at upload via bucket policy** — deny `s3:PutObject` requests that don't include `x-amz-server-side-encryption` header. This ensures no plaintext objects are ever uploaded.
* **Enable S3 Access Logging** — audit logs of every request to your bucket, stored in a separate logging bucket.

### Cost Optimization
* **Use S3 Lifecycle Rules** — automatically transition objects to cheaper storage classes as they age. A typical pattern: Standard (0–30 days) → Standard-IA (30–90 days) → Glacier Flexible (90–365 days) → Deep Archive (365+ days).
* **Clean up non-current versions** — versioning is invaluable but accumulates storage costs. Set a lifecycle rule to expire non-current versions after 30–90 days.
* **Match storage class to access pattern** — over-using S3 Standard for data that's accessed once a year wastes money. Use Intelligent-Tiering if patterns are unknown.

### Replication
* **Always enable versioning before setting up replication** — replication cannot function without versioning enabled on both source and destination.
* **Use Batch Replication for pre-existing data** — don't assume existing objects will replicate automatically after enabling CRR/SRR.
* **Test restore procedures regularly** — enabling CRR and never testing recovery is not a backup strategy. Periodically test that you can retrieve objects from the destination bucket.

---

## 🔥 13. High-Frequency Exam Facts

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                  SAA-C03 S3 EXAM FACTS                           │
  ├──────────────────────────────────────────────────────────────────┤
  │  CORE                                                            │
  │  ✅ S3 is a global service but buckets are REGIONAL              │
  │  ✅ Bucket names are GLOBALLY unique (all accounts, all regions) │
  │  ✅ S3 has no real directories — only keys with slashes          │
  │  ✅ Max object size = 5 TB                                       │
  │  ✅ Max single PUT upload = 5 GB                                 │
  │  ✅ Multi-Part Upload REQUIRED for objects > 5 GB                │
  │  ✅ Durability = 99.999999999% (11 nines) for ALL classes        │
  ├──────────────────────────────────────────────────────────────────┤
  │  SECURITY                                                        │
  │  ✅ Explicit DENY always wins regardless of Allow policies       │
  │  ✅ Block Public Access overrides bucket policies                │
  │  ✅ EC2 → S3: use IAM Role, NEVER access keys                   │
  │  ✅ Cross-account → S3: use Bucket Policy with account ARN       │
  ├──────────────────────────────────────────────────────────────────┤
  │  VERSIONING                                                      │
  │  ✅ Pre-existing objects get version ID = null                   │
  │  ✅ Suspending versioning does NOT delete existing versions      │
  │  ✅ Delete = adds a delete marker (not permanent removal)        │
  │  ✅ Versioning is REQUIRED for replication to work               │
  ├──────────────────────────────────────────────────────────────────┤
  │  REPLICATION                                                     │
  │  ✅ Only NEW objects are replicated after enabling               │
  │  ✅ Existing objects need S3 Batch Replication                   │
  │  ✅ Version ID deletes are NEVER replicated                      │
  │  ✅ Delete markers are optionally replicated                     │
  │  ✅ No replication chaining: 1→2→3, objects from 1 skip 3       │
  ├──────────────────────────────────────────────────────────────────┤
  │  STORAGE CLASSES                                                 │
  │  ✅ One Zone-IA: data lost if AZ is destroyed                    │
  │  ✅ Glacier Deep Archive: cheapest, min 180 days                 │
  │  ✅ Intelligent-Tiering: NO retrieval fees                       │
  │  ✅ Glacier Instant: millisecond retrieval at Glacier pricing    │
  │  ✅ Min durations: IA=30d, Glacier=90d, Deep Archive=180d       │
  │  ✅ Min billable size: IA=128KB, Glacier Flexible/Deep=40KB     │
  └──────────────────────────────────────────────────────────────────┘
```