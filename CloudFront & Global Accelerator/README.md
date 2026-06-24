# 🌐 Amazon CloudFront & AWS Global Accelerator — Complete Reference Guide

<div align="center">

![AWS](https://img.shields.io/badge/AWS-CloudFront%20%26%20Global%20Accelerator-orange?style=for-the-badge&logo=amazonaws)
![CDN](https://img.shields.io/badge/CloudFront-CDN%20%26%20Edge%20Caching-blue?style=for-the-badge)
![Accelerator](https://img.shields.io/badge/Global%20Accelerator-Anycast%20Routing-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-SAA--C03%20Ready-success?style=for-the-badge)

![Amazon CloudFront](assets/Amazon%20CloudFront.jpg)

### ⚡ Global Content Delivery & Low-Latency Network Acceleration Blueprint

*A deep-dive technical reference covering how CDNs work, how edge location selection happens, CloudFront origins, cache invalidation strategies, file versioning, geo restriction, Global Accelerator Anycast routing, and a definitive comparison between both services.*

</div>

---

## 📖 Overview

When a user in Cairo requests a file hosted on an origin server in `us-east-1`, the request travels thousands of kilometers across the public internet — passing through dozens of routers, submarine cables, and internet exchange points, each adding latency. **Amazon CloudFront** solves this by caching content at edge locations physically close to the user. **AWS Global Accelerator** solves it differently — it doesn't cache, but it routes traffic over the private AWS backbone instead of the public internet from the very first hop.

Understanding which service to use, and when, is a high-frequency SAA-C03 topic.

---

## 📡 1. How a CDN Works — The Full Mental Model

A **Content Delivery Network (CDN)** is a globally distributed network of cache servers (called **edge locations** or **Points of Presence**) that store copies of your origin content close to end users.

Without a CDN, every user hits your origin directly:

```
[ User in Cairo ]
       │
       │  Public internet — ~200ms round trip
       │  crossing: Egypt → Mediterranean → Atlantic → US East Coast
       ▼
[ Origin Server: us-east-1 ]
   Responds with the full file every time
```

With a CDN, the first user warms the cache; every subsequent user gets the cached copy from the edge:

```
[ User 1 in Cairo ] ──► [ Edge Location: Cairo ]
                               │
                    CACHE MISS — file not cached yet
                               │
                               ▼
                    [ Origin: us-east-1 ] ──► returns file
                               │
                    Edge Location caches the file
                               │
                    Returns file to User 1 (~20ms)
                    ─────────────────────────────────────
[ User 2 in Cairo ] ──► [ Edge Location: Cairo ]
                               │
                    CACHE HIT — file already cached
                               │
                    Returns file instantly (~5ms)
                    No origin contact needed
```

### What the cache stores

Every cached object is stored with a **TTL (Time To Live)** value — a timer that tells the edge location how long the object is valid. When the TTL expires, the edge evicts the object from cache. The next request for that object is a cache miss, triggering a fresh fetch from the origin.

```
Object: /images/banner.jpg  │  TTL: 86400 seconds (24 hours)
Cached at: 09:00 UTC        │  Expires: 09:00 UTC next day
                            │
09:30 UTC → User requests   → CACHE HIT  (served from edge)
17:00 UTC → User requests   → CACHE HIT  (still within TTL)
09:01 UTC next day          → CACHE MISS (TTL expired — fetched from origin again)
```

---

## 🗺️ 2. How a Client Selects the Closest Edge Location

This is the mechanism that makes CDNs fast. CloudFront uses **DNS-based edge selection** — the process is transparent to the user and happens automatically on every request.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DNS-BASED EDGE SELECTION FLOW                            │
└─────────────────────────────────────────────────────────────────────────────┘

Step 1: User types https://d1234abcd.cloudfront.net/image.jpg in their browser
             │
             ▼
Step 2: Browser sends DNS query to its local Recursive DNS Resolver
        (usually the ISP's DNS server, or 8.8.8.8, or 1.1.1.1)
             │
             ▼
Step 3: Recursive Resolver queries CloudFront's Authoritative DNS
             │
             ▼
Step 4: CloudFront DNS knows the resolver's IP address
        → Uses it to estimate the user's geographic location
        → Identifies the closest healthy edge location
             │
             ▼
Step 5: CloudFront DNS returns the IP address of the closest edge location
        (NOT the origin — the nearest Point of Presence)
             │
             ▼
Step 6: Browser connects directly to that edge location
        → Cache hit → returns immediately
        → Cache miss → edge fetches from origin, caches, returns to user
```

### Why the resolver's IP is used (not the user's IP)

CloudFront's DNS sees the **recursive resolver's** IP address, not the end user's IP. For most users these are geographically close (an ISP's DNS server is typically in the same city). With services like Google DNS (8.8.8.8), geographic accuracy is lower because Google's resolvers are centralized. AWS mitigates this using **EDNS Client Subnet (ECS)** — an extension that allows resolvers to pass the client's subnet to the authoritative DNS, improving edge selection accuracy.

```
┌──────────────────────────────────────────────────────────────┐
│  EDGE SELECTION CRITERIA (in priority order):                │
│                                                              │
│  1. Geographic proximity    (minimizes raw network distance) │
│  2. Network latency         (measured routing efficiency)    │
│  3. Edge location health    (unhealthy edges are excluded)   │
│  4. Edge location capacity  (overloaded edges avoided)       │
└──────────────────────────────────────────────────────────────┘
```

### CloudFront Points of Presence (2024+)

CloudFront operates **hundreds of edge locations** across 90+ cities in 47 countries, plus **regional edge caches** — a mid-tier cache layer between edge locations and origins for objects with lower access frequency.

```
[ User ] → [ Edge Location (nearest city) ]
                      │ cache miss
                      ▼
           [ Regional Edge Cache (mid-tier) ]
                      │ cache miss
                      ▼
           [ Origin (S3 / ALB / EC2) ]
```

Regional edge caches have larger storage than edge locations, so objects that are too infrequently accessed to stay cached at the edge can still be served without going all the way to the origin.

---

## ☁️ 3. Amazon CloudFront — Core Architecture

Amazon CloudFront is AWS's fully managed CDN. It integrates natively with AWS origins (S3, ALB, EC2) and provides built-in DDoS protection, HTTPS termination, geo restriction, and cache control.

![CloudFront at a high level](assets/CloudFront%20at%20a%20high%20level.png)

```
                          ┌─────────────────────────────┐
                         │     CloudFront Distribution │
                         │                             │
                         │  Origin(s):                 │
                         │  - S3 Bucket (OAC)          │
                         │  - VPC Origin (private ALB) │
                         │  - Custom HTTP Origin       │
                         │                             │
                         │  Behaviors:                 │
                         │  /images/* → cache 24h      │
                         │  /api/*    → no cache       │
                         └──────────────┬──────────────┘
                                        │
              ┌─────────────────────────┼───────────────────────┐
              ▼                         ▼                         ▼
   [ Edge: São Paulo ]       [ Edge: Frankfurt ]       [ Edge: Singapore ]
     Cached content            Cached content            Cached content
              ▲                         ▲                         ▲
              │                         │                         │
   [ Users: Brazil ]         [ Users: Europe ]      [ Users: Southeast Asia ]
```

---

## 🎯 4. CloudFront Origins

A **CloudFront origin** is the authoritative source of truth — where CloudFront fetches content on a cache miss.

### Origin Type 1 — Amazon S3 Bucket

The most common CloudFront origin for static content (images, JS, CSS, HTML, video files).

![CloudFront – S3 as an Origin](assets/CloudFront%20%E2%80%93%20S3%20as%20an%20Origin.png)

```
[ User Request: GET /logo.png ]
         │
         ▼
[ CloudFront Edge Location ]
         │ cache miss
         │
         │  Uses Origin Access Control (OAC) — a signed request
         ▼
[ S3 Bucket (private — direct public access blocked) ]
         │
         ▼  Returns object
[ Edge Location caches + serves to user ]
```

**Origin Access Control (OAC):**
* OAC replaces the older Origin Access Identity (OAI).
* CloudFront signs every request to S3 using AWS SigV4 before forwarding it.
* The S3 bucket policy grants access exclusively to the CloudFront distribution's OAC — direct public access to the bucket is blocked entirely.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-private-bucket/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/EDFDVBD6EXAMPLE"
        }
      }
    }
  ]
}
```

**CloudFront + S3 also supports uploads:** You can use a CloudFront distribution as the upload endpoint for `PUT` operations, which CloudFront then forwards to S3 — useful for globally distributed upload endpoints.

---

### Origin Type 2 — VPC Origin (Private Resources)

VPC Origins allow CloudFront to deliver content from resources in **private VPC subnets** without exposing them to the internet.

![CloudFront ALB or EC2 as an origin Using VPC Origins](assets/CloudFront%20%20ALB%20or%20EC2%20as%20an%20origin%20Using%20VPC%20Origins.png)

```
[ User ]
    │
    ▼
[ CloudFront Edge Location ]
    │
    │  Direct private connection — no internet traversal
    ▼
[ VPC Origin ]
    │
    ├──► Private Application Load Balancer
    ├──► Private Network Load Balancer
    └──► Private EC2 Instance

No public IP required on the backend resources
No security group rules for "the entire internet" needed
```

This replaces the older approach (below) where you had to whitelist all CloudFront IP ranges on your security group.

---

### Origin Type 3 — Custom HTTP Origin (Public Backend)

Any public HTTP/HTTPS endpoint can serve as a CloudFront origin — a public ALB, an on-premises server, or a third-party API.

![CloudFront – ALB or EC2 as an origin Using Public Network](assets/CloudFront%20%E2%80%93%20ALB%20or%20EC2%20as%20an%20originUsingPublic%20Network.png)

```
[ CloudFront Edge ] ─── HTTPS ──► [ Public Application Load Balancer ]
                                              │
                                              ▼
                                  [ EC2 instances in private subnet ]
                                  (EC2 does not need to be public)
```

**Security Group Configuration for Public Origins (older approach):**

```
CloudFront Edge Location IPs → Allowed in ALB Security Group
ALB Security Group           → Allowed in EC2 Security Group

Reference: http://d7uri8nf7uska.cloudfront.net/tools/list-cloudfront-ips
(This list changes as CloudFront grows — prefer VPC Origins for new architectures)
```

---

## 🆚 5. CloudFront vs. S3 Cross-Region Replication

These are frequently confused because both involve delivering S3 content globally. They solve different problems:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     CLOUDFRONT                                              │
│                                                                             │
│  Mechanism: Edge caching — one origin, cached copies at hundreds of edges   │
│  Consistency: Eventually consistent — cached version may be stale (TTL)     │
│  Setup: One distribution covers all regions automatically                   │
│  Content type: Best for STATIC content (images, JS, CSS, video, HTML)       │
│  Access pattern: Many users hitting the same file (high cache hit ratio)    │
│  Latency: Near-instant from cache — 1–10ms typical                          │
│  Cost model: Pay for data transfer out from edges + requests                │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                  S3 CROSS-REGION REPLICATION (CRR)                          │
│                                                                             │
│  Mechanism: Full object replication — actual copies exist in each region    │
│  Consistency: Near-real-time replication (seconds, not hours)               │
│  Setup: Must configure each source→destination region pair explicitly       │
│  Content type: Best for DYNAMIC content with low-latency regional needs     │
│  Access pattern: Users need the very latest version — no stale reads        │
│  Replication: Read-only at destination — writes only to source              │
│  Cost model: Pay for storage in each region + replication data transfer     │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Decision rule:** If your content changes infrequently and is accessed by many global users → CloudFront. If your content changes frequently and must be immediately available in specific regions → CRR.

---

## 🌍 6. CloudFront Geo Restriction

CloudFront lets you control access based on the user's country, determined via a third-party Geo-IP database.

```
[ Incoming Request ]
         │
         ▼
[ CloudFront checks requesting country ]
   (determined by client IP → Geo-IP database lookup)
         │
    ┌────┴────────────────────────────────────┐
    ▼                                         ▼
[ ALLOWLIST mode ]                    [ BLOCKLIST mode ]
  "Only serve users in                  "Deny users in
   approved countries"                   banned countries"
  All others → 403 Forbidden            All others → allowed
```

**Primary use case:** Copyright and licensing laws. A streaming service may be licensed to show a film only in certain countries — CloudFront geo restriction enforces this at the CDN layer without modifying your origin logic.

> **Exam note:** The geographic determination is based on the **client's IP address** and a **third-party Geo-IP database** — it is not based on AWS account region.

---

## 🗑️ 7. Cache Invalidation — Forcing Content Refresh

### The Problem: Stale Cache

When you update a file on your origin (e.g., you deploy a new version of `logo.png` to S3), CloudFront edge locations do **not** know the file changed. They continue serving the old cached version until the TTL expires.

```
[ You update logo.png in S3 at 14:00 ]
                │
                │  CloudFront edges are unaware
                ▼
[ Edge Location: has logo.png cached since 09:00, TTL = 24h ]
[ Edge Location: will continue serving the OLD logo.png until 09:00 tomorrow ]
[ Users see the old file for up to 15 more hours ]
```

### The Solution: Cache Invalidation

A **CloudFront Invalidation** is an **explicit command you send to CloudFront** to immediately evict specified objects from all edge location caches — bypassing the TTL entirely.

```
[ You ]
  │
  │  Submit invalidation request to CloudFront API / Console
  │  Specify path(s) to invalidate:
  │    - /index.html           (single file)
  │    - /images/*             (all files under /images/)
  │    - /*                    (entire cache — use sparingly)
  ▼
[ CloudFront Control Plane ]
  │
  │  Propagates invalidation to ALL edge locations worldwide
  │  (takes ~1–5 minutes to complete globally)
  ▼
[ All Edge Locations ]
  Evict the matched objects from cache
  │
  ▼
[ Next user request for /index.html ]
  CACHE MISS → edge fetches fresh copy from origin
  Caches the new version
  Serves the updated file
```

**Invalidation is initiated by the user/operator** — it does not happen automatically when you update the origin. If you update a file on S3 and do nothing in CloudFront, users will see the stale version until the TTL expires. You must actively submit the invalidation request.

### Invalidation Cost

* **First 1,000 invalidation paths per month:** Free.
* **Beyond 1,000 paths:** Charged per path (wildcard `/*` counts as 1 path).

Invalidations with wildcards (`/images/*`) are efficient for cost — one path covers thousands of files.

---

## 🏷️ 8. File Versioning — A Better Strategy Than Invalidation

Cache invalidation works, but it has a cost, takes 1–5 minutes to propagate globally, and requires an active manual step. The **recommended best practice** is to use **file versioning** instead.

### What is File Versioning?

Rather than overwriting the same filename (`logo.png`), you embed a version identifier in the filename itself on every deploy:

```
Old deployment: /assets/logo.png          → cached everywhere
New deployment: /assets/logo.v2.png       → brand new URL, never cached before
             OR /assets/logo.abc123.png   → content-hash in filename
```

### Why Versioning is Better Than Invalidation

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  INVALIDATION                        │  FILE VERSIONING                     │
│  ─────────────────────────────────── │  ──────────────────────────────────  │
│  Old URL stays the same              │  New URL on every deploy             │
│  Must actively invalidate            │  No invalidation needed              │
│  Propagates in 1–5 minutes           │  Instant — new URL = fresh cache     │
│  May cost money (>1000 paths/month)  │  No additional CDN cost              │
│  Users may still see stale briefly   │  Old URL still works (rollback!)     │
│                                      │  Perfect cache-busting guaranteed    │
└─────────────────────────────────────────────────────────────────────────────┘
```

### How Versioning Enables Instant Cache-Busting

```
[ Deploy v2 of your app ]
    │
    ├── Upload logo.v2.png to S3 (new file, new key)
    ├── Update index.html to reference logo.v2.png
    │
    ▼
[ Invalidate /index.html only ] ← only ONE file to invalidate
    │
    ▼
[ Users fetch the new index.html ]
    → index.html references logo.v2.png
    → logo.v2.png has never been cached → fresh fetch from origin
    → logo.v1.png is still cached but no longer referenced
```

You only ever need to invalidate `index.html` (or your entry point) — all other assets are versioned URLs that are automatically cache-busted because they are brand-new paths.

### How to implement versioning in practice

Modern build tools (Webpack, Vite, Next.js, Parcel) do this automatically — they generate content-hashed filenames for every asset on every build. Your `index.html` is the only file that always has the same name and needs invalidation on each deploy.

```
Build output example:
  index.html                    ← always same name, invalidate on deploy
  assets/main.a1b2c3d4.js       ← content hash in name, never needs invalidation
  assets/styles.e5f6a7b8.css    ← content hash in name, never needs invalidation
  assets/logo.9c0d1e2f.png      ← content hash in name, never needs invalidation
```

---

## 🌏 9. Unicast IP vs. Anycast IP — The Routing Foundation

To understand Global Accelerator, you first need to understand these two IP routing models.

![Unicast IP vs Anycast IP](assets/Unicast%20IP%20vs%20Anycast%20IP.png)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  UNICAST IP                                                                  │
│  ──────────────────────────────────────────────────────────────────────────  │
│  One server → One unique IP address                                          │
│                                                                              │
│  Server A (us-east-1)  → IP: 52.1.2.3                                        │
│  Server B (eu-west-1)  → IP: 54.5.6.7                                        │
│  Server C (ap-east-1)  → IP: 13.9.10.11                                      │
│                                                                              │
│  Problem: To use the closest server, the client must know its specific IP.   │
│  If Server A fails, clients must switch to a different IP.                   │
│  DNS TTL delays make failover slow (60s–300s typically).                     │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  ANYCAST IP                                                                  │
│  ──────────────────────────────────────────────────────────────────────────  │
│  Multiple servers → All share the SAME IP address                            │
│                                                                              │
│  Server A (us-east-1)  → IP: 75.2.60.5  ◄─┐                                  │
│  Server B (eu-west-1)  → IP: 75.2.60.5  ◄─┤ All the same IP                  │
│  Server C (ap-east-1)  → IP: 75.2.60.5  ◄─┘                                  │
│                                                                              │
│  The internet routing infrastructure (BGP) automatically routes each         │
│  client's request to the nearest server advertising that IP.                 │
│  Client in Cairo → routed to eu-west-1 (Frankfurt) automatically.            │
│  Client in Brazil → routed to us-east-1 automatically.                       │
│  If one server goes down → BGP re-routes traffic to the next closest.        │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🚀 10. AWS Global Accelerator

AWS Global Accelerator uses **Anycast IPs** to route user traffic into the AWS private backbone network at the nearest edge location, then delivers that traffic to your application over AWS's internal fiber — bypassing the unpredictable public internet for most of the journey.

```
[ User: Cairo ]                      [ User: São Paulo ]
       │                                      │
       │ connects to Anycast IP: 75.2.60.5   │ connects to Anycast IP: 75.2.60.5
       ▼                                      ▼
[ Edge: Frankfurt ] ─────────────────► [ Edge: Miami ]
       │   AWS Private Network                 │   AWS Private Network
       │   (low-latency, consistent)           │   (low-latency, consistent)
       ▼                                      ▼
[ Your Application: us-east-1 ALB ]   [ Your Application: us-east-1 ALB ]
```

Without Global Accelerator, both users travel the entire distance over the public internet. With Global Accelerator, each user enters the AWS network at the nearest edge and travels the rest of the journey on AWS's dedicated, optimized private backbone.

### The 2 Anycast IPs

Global Accelerator provisions exactly **2 static Anycast IPs** per accelerator:

```
Anycast IP 1: 75.2.60.5   ─► advertised from all Global Accelerator edge locations
Anycast IP 2: 99.83.10.21 ─► advertised from all Global Accelerator edge locations

Both IPs always route to the closest healthy endpoint.
If one IP has issues, the other continues working.
```

These IPs **never change** for the lifetime of the accelerator — this is a critical advantage over DNS-based routing, where IP changes require waiting for DNS propagation and TTL expiry.

---

## ⚡ 11. AWS Global Accelerator — Features

### Consistent Performance

* **Intelligent routing:** Traffic always takes the lowest-latency path from the nearest edge to your application endpoint.
* **No client cache issues:** Because Global Accelerator uses static IPs (not DNS names), there is no DNS TTL to wait out when you change your endpoint configuration. The change propagates in seconds.
* **AWS internal network:** Avoids the congestion, packet loss, and routing instability of the public internet.

### Health Checks and Automatic Failover

```
[ Global Accelerator ]
   │
   │  Continuously health-checks endpoints (every 10 or 30 seconds)
   ▼
[ us-east-1 ALB: HEALTHY ]   [ eu-west-1 ALB: HEALTHY ]
   │
   │  Scenario: us-east-1 ALB fails health check
   ▼
[ Global Accelerator detects failure ]
   │
   │  Automatically reroutes ALL traffic to eu-west-1 ALB
   │  Failover completes in under 60 seconds
   ▼
[ Users seamlessly redirected — no DNS change, no client reconfiguration ]
```

### Security

* **Only 2 IPs to whitelist** — customers with strict firewall policies can whitelist just these 2 IPs globally, regardless of how many regions your application runs in.
* **DDoS protection** via AWS Shield Standard (included) — Global Accelerator absorbs DDoS traffic at the edge before it reaches your application.

### Supported Endpoint Types

| Endpoint Type | Public? | Private? |
|---|---|---|
| Elastic IP | ✅ | N/A |
| EC2 Instance | ✅ | ✅ |
| Application Load Balancer | ✅ | ✅ |
| Network Load Balancer | ✅ | ✅ |

---

## 🆚 12. CloudFront vs. Global Accelerator — Definitive Comparison

Both services use the AWS global network and AWS Shield DDoS protection. Both use edge locations. The difference is in *what they do at the edge*:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  CLOUDFRONT                                                                  │
│  ──────────────────────────────────────────────────────────────────────────  │
│  What it does: CACHES content at the edge                                    │
│  Protocol: HTTP and HTTPS only                                               │
│  How it works: Content is SERVED from the edge — origin not hit on cache hit │
│  IP addresses: Dynamic (DNS-based routing via cloudfront.net domain)         │
│  Content types: Images, video, JS, CSS, HTML, API responses                  │
│  Best for: Static content globally; API acceleration; cacheable responses    │
│  Not good for: Non-HTTP protocols (UDP, MQTT, gaming)                        │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  AWS GLOBAL ACCELERATOR                                                      │
│  ──────────────────────────────────────────────────────────────────────────  │
│  What it does: ROUTES traffic through AWS backbone — no caching              │
│  Protocol: TCP and UDP (any application protocol)                            │
│  How it works: Traffic is PROXIED at the edge — always reaches the endpoint  │
│  IP addresses: Static Anycast IPs (never change)                             │
│  Content types: Any TCP/UDP traffic                                          │
│  Best for: Gaming (UDP), IoT (MQTT), VoIP, non-HTTP workloads                │
│  Also good for: HTTP with static IPs; fast failover; deterministic routing   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Side-by-Side Feature Matrix

| Feature | CloudFront | Global Accelerator |
|---|---|---|
| **Protocol** | HTTP / HTTPS | TCP / UDP (any) |
| **Caching** | ✅ Yes — content served at edge | ❌ No — traffic proxied to origin |
| **IP addresses** | Dynamic (DNS-based) | Static Anycast (2 IPs) |
| **DDoS protection** | ✅ Shield + WAF | ✅ Shield |
| **Health checks** | Via Route 53 / origin policies | ✅ Built-in, <60s failover |
| **Static IP requirement** | ❌ Cannot provide static IPs | ✅ 2 fixed Anycast IPs |
| **Gaming / IoT / VoIP** | ❌ HTTP only | ✅ Native support |
| **Cache invalidation** | ✅ Yes | N/A — no caching |
| **Geo restriction** | ✅ Allowlist / Blocklist | ❌ Not available |
| **Origin types** | S3, ALB, EC2, Custom HTTP | EC2, ALB, NLB, Elastic IP |
| **Best use case** | Static & cacheable web content | Non-HTTP or static-IP requirements |

---

## 🕳️ 13. Exam Traps & Scenario Analysis

### 🚨 Trap 1: CloudFront vs. Global Accelerator for Non-HTTP

* **Question pattern:** "A gaming company needs to reduce latency for UDP game traffic from global users to servers in us-east-1."
* **Wrong answer:** Use Amazon CloudFront.
* **Correct answer:** Use **AWS Global Accelerator**.
* **Reasoning:** CloudFront only handles HTTP and HTTPS traffic. Game data travels over UDP — CloudFront cannot proxy UDP. Global Accelerator handles any TCP/UDP protocol.

### 🚨 Trap 2: Static IP Requirement

* **Question pattern:** "A financial services firm has firewall rules that allow traffic only from specific, known IP addresses. They need to expose a global application. What should they use?"
* **Wrong answer:** CloudFront — it provides a dynamic DNS name, not static IPs.
* **Correct answer:** **AWS Global Accelerator** — provides 2 static Anycast IPs that never change.
* **Reasoning:** Customers can whitelist the 2 static Anycast IPs once. CloudFront's IP addresses can change (they are resolved dynamically via DNS and managed by AWS).

### 🚨 Trap 3: CloudFront vs. CRR for Dynamic Content

* **Question pattern:** "A company stores user-specific data in S3 that changes multiple times per day. They need all global regions to see the latest version within seconds."
* **Wrong answer:** Use CloudFront to cache the S3 objects globally.
* **Correct answer:** Use **S3 Cross-Region Replication**.
* **Reasoning:** CloudFront caches for the duration of the TTL — users may see stale data for hours. CRR replicates changes to destination buckets in near-real-time (seconds). CloudFront is for static, long-lived content.

### 🚨 Trap 4: Cache Invalidation vs. Versioning

* **Question pattern:** "A company deploys a new version of their web application daily. They want cache updates to be instant and free. What is the best practice?"
* **Wrong answer:** Submit a CloudFront invalidation for `/*` on every deployment.
* **Correct answer:** Use **file versioning** — include a content hash or version number in filenames; only invalidate `index.html`.
* **Reasoning:** Wildcard invalidation costs money beyond 1,000 paths/month, takes 1–5 minutes to propagate, and is operationally heavy. Versioned filenames provide instant cache-busting at zero CDN cost because they are new, never-cached URLs.

### 🚨 Trap 5: OAC vs. Making the S3 Bucket Public

* **Question pattern:** "A company uses CloudFront to serve private S3 objects. How should they prevent users from accessing S3 directly?"
* **Wrong answer:** Keep the bucket public and rely on CloudFront security groups.
* **Correct answer:** Keep the bucket **private**, use **Origin Access Control (OAC)** on the CloudFront distribution, and attach an S3 bucket policy granting `s3:GetObject` to that OAC only.
* **Reasoning:** A public bucket means anyone can access S3 directly — bypassing CloudFront, geo restriction, signed URLs, and WAF rules. OAC ensures CloudFront is the only permitted access path.

---

## ⚡ 14. High-Frequency Exam Facts

| Fact | Answer |
|---|---|
| CloudFront uses how many PoPs globally? | **Hundreds — 90+ cities in 47+ countries** |
| How clients find the closest edge | **DNS-based geo routing (Anycast DNS responses)** |
| CloudFront DDoS protection | **AWS Shield Standard (included) + WAF (optional)** |
| S3 origin security mechanism | **Origin Access Control (OAC)** |
| How to force cache refresh before TTL expires | **CloudFront Invalidation** |
| Best practice to avoid invalidation cost | **File versioning (content-hashed filenames)** |
| Invalidation free tier | **First 1,000 paths/month** |
| Global Accelerator provides how many static IPs? | **2 Anycast IPs** |
| Global Accelerator failover time | **Less than 60 seconds** |
| Which service caches at the edge? | **CloudFront only — Global Accelerator does not cache** |
| Which supports UDP? | **Global Accelerator only** |
| CloudFront IP addresses: static or dynamic? | **Dynamic — resolved via DNS** |
| Global Accelerator IPs: static or dynamic? | **Static — 2 fixed Anycast IPs** |
| Geo restriction data source | **Third-party Geo-IP database** |
| VPC Origins replaced what? | **Whitelisting CloudFront IPs in security groups** |
| CloudFront + S3 upload direction | **Client → CloudFront Edge → S3 (PUT supported)** |

---

## ✅ 15. Production Best Practices

* **Always use OAC, never public S3 buckets behind CloudFront.** A public S3 bucket bypasses all CloudFront security controls. OAC ensures S3 is only reachable via your distribution.
* **Set appropriate TTLs per content type.** Images and fonts rarely change — use 30-day TTLs. API responses may change often — use short TTLs (60s) or `Cache-Control: no-cache` headers. HTML entry points should use short TTLs and be versioned.
* **Use file versioning over invalidation for all assets except entry points.** Configure your build tool to output content-hashed filenames for all JS, CSS, and image assets. Only `index.html` needs invalidation on each deploy.
* **Enable WAF on your CloudFront distribution for public-facing applications.** AWS WAF integrates natively with CloudFront and provides SQL injection protection, XSS filtering, rate limiting, and custom rules — all evaluated at the edge before traffic reaches your origin.
* **Use CloudFront for S3 static websites, not S3 website endpoints.** S3 website endpoints are HTTP-only and lack HTTPS support. CloudFront provides HTTPS, custom domains, and caching on top.
* **Use Global Accelerator for applications requiring static IPs or non-HTTP protocols.** Do not try to use CloudFront for gaming, IoT, or VoIP. Use Global Accelerator, and configure health checks on all endpoint regions for automatic failover.
* **Enable Regional Edge Caches for infrequently accessed content.** Regional edge caches sit between edge locations and origins and have much higher storage capacity — they prevent objects from being evicted too quickly from small edge location caches.
* **Use CloudFront Signed URLs or Signed Cookies for premium content access control.** These allow time-limited, user-specific access to private CloudFront content — similar to S3 pre-signed URLs but enforced at the CDN layer.