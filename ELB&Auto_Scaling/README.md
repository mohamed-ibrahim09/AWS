# ⚖️ AWS Elastic Load Balancing & Auto Scaling — Complete Reference Guide

<div align="center">

![AWS](https://img.shields.io/badge/AWS-Load%20Balancing-orange?style=for-the-badge&logo=amazonaws)
![ELB](https://img.shields.io/badge/ELB-ALB%20%7C%20NLB%20%7C%20GWLB-blue?style=for-the-badge)
![ASG](https://img.shields.io/badge/ASG-Auto%20Scaling-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-SAA--C03%20Ready-success?style=for-the-badge)

### ⚡ Scalability, High Availability & Traffic Distribution Engineering Blueprint

*A deep-dive technical engineering reference covering vertical and horizontal scaling, all four managed load balancer types, routing strategies, SSL/TLS termination, sticky sessions, cross-zone balancing, connection draining, and Auto Scaling Group lifecycle management.*

</div>

---

## 📖 Overview

This guide covers the full AWS traffic distribution and scaling stack, from scalability fundamentals and Elastic Load Balancing to Auto Scaling Groups, health checks, routing strategies, TLS termination, and production operations. It is designed as both a practical engineering reference and an AWS certification study guide.

> Reorganized for a learning-first flow while preserving the original engineering blueprint style, certification traps, diagrams, command examples, and production decision frameworks.

## 🗺️ Table of Contents

### 🧱 [Part I — Foundations](#part-i-foundations)
1. [Why Load Balancers Exist](#why-load-balancers-exist)
2. [Scalability vs High Availability](#scalability-vs-high-availability)
3. [Elastic Load Balancing Foundations](#elastic-load-balancing-foundations)

### ⚖️ [Part II — Load Balancer Types](#part-ii-load-balancer-types)
4. [Load Balancer Overview](#load-balancer-overview)
5. [Application Load Balancer (ALB)](#application-load-balancer-alb)
6. [Network Load Balancer (NLB)](#network-load-balancer-nlb)
7. [Gateway Load Balancer (GWLB)](#gateway-load-balancer-gwlb)
8. [Classic Load Balancer (CLB)](#classic-load-balancer-clb)
9. [Load Balancer Comparison Matrix](#load-balancer-comparison-matrix)

### 🧭 [Part III — Traffic Routing](#part-iii-traffic-routing)
10. [Target Groups](#target-groups)
11. [ALB Routing Rules](#alb-routing-rules)
12. [Weighted Target Groups](#weighted-target-groups)
13. [NLB → ALB Chaining](#nlb-to-alb-chaining)

### 💓 [Part IV — Health & Availability](#part-iv-health-availability)
14. [Health Checks](#health-checks)
15. [Health Check Parameters](#health-check-parameters)
16. [Health Check Grace Period](#health-check-grace-period)
17. [Slow Start](#slow-start)
18. [Deregistration Delay](#deregistration-delay)

### 🔐 [Part V — Security](#part-v-security)
19. [TLS & SSL](#tls-ssl)
20. [AWS Certificate Manager (ACM)](#aws-certificate-manager-acm)
21. [Server Name Indication (SNI)](#server-name-indication-sni)
22. [TLS Policies](#tls-policies)
23. [Security Group Design](#security-group-design)

### 📈 [Part VI — Auto Scaling](#part-vi-auto-scaling)
24. [AMIs](#amis)
25. [Launch Templates](#launch-templates)
26. [Auto Scaling Groups](#auto-scaling-groups)
27. [Scaling Policies](#scaling-policies)
28. [Instance Refresh](#instance-refresh)
29. [Mixed Instance Fleets](#mixed-instance-fleets)
30. [Warm Pools](#warm-pools)
31. [Lifecycle Hooks](#lifecycle-hooks)
32. [ASG Lifecycle States](#asg-lifecycle-states)

### 📊 [Part VII — Monitoring & Cost](#part-vii-monitoring-cost)
33. [CloudWatch Metrics](#cloudwatch-metrics)
34. [Access Logs](#access-logs)
35. [LCU & NLCU Billing](#lcu-nlcu-billing)
36. [Cross-AZ Costs](#cross-az-costs)

### 🎯 [Part VIII — Decision Making & Exams](#part-viii-decision-making-exams)
37. [Selection Framework](#selection-framework)
38. [Certification Traps](#certification-traps)
39. [Production Best Practices](#production-best-practices)

---

<a id="part-i-foundations"></a>
# 🧱 Part I — Foundations

<a id="elastic-load-balancing-foundations"></a>
## 🧱 1-3. Elastic Load Balancing Foundations

An **Elastic Load Balancer (ELB)** is a managed regional entry point engineered to provide durable, high-performance traffic distribution for a fleet of EC2 instances, containers, or IP endpoints.

<a id="why-load-balancers-exist"></a>
### 🚦 1.0 Why Load Balancers Exist

A single server creates two production problems: it can become overloaded when traffic rises, and it can become a single point of failure when it crashes. A load balancer solves both by giving users one stable entry point while distributing requests across multiple healthy targets.

```text
Without ELB:
[ Users ] ──► [ One EC2 Instance ]
                  │
                  └── Failure or overload affects everyone

With ELB:
[ Users ] ──► [ Load Balancer ] ──► [ EC2-A ] [ EC2-B ] [ EC2-C ]
                                      │ healthy targets receive traffic
```

| Question | Answer |
| --- | --- |
| **What is it?** | A managed AWS traffic entry point that sends requests to registered backend targets. |
| **Why does it exist?** | To remove single-instance bottlenecks and route around unhealthy capacity. |
| **When should I use it?** | Use it when an application needs high availability, horizontal scaling, TLS termination, or traffic routing. |
| **Real-world example** | An e-commerce site runs web servers across three AZs behind an ALB so checkout traffic survives an instance or AZ failure. |
| **Exam tip** | If a question asks for Multi-AZ traffic distribution across EC2 instances, ELB is usually part of the answer. |

```text
[ Client Plane ] ──► (AWS Edge / Internet) ──► [ ELB Node Array ]
 Global Users                                   Physically Isolated per AZ
 (HTTP/TCP/UDP)                                 (Scales independently of targets)

```

### 🏗️ 1.1 The Architecture & Node Placement Model

* **Network Interconnect Layer:** ELB nodes are completely separate from the physical hardware hosting your EC2 instances. They communicate over the VPC data plane, which introduces microsecond-level latency compared to direct instance access.
* **Dynamic Lifecycle Management:** Because they are decoupled from the target lifecycle, you can register and deregister instances from a target group without recreating or disrupting the load balancer.
* **Availability Zone Lock Constraint:** An ELB provisions at least one load balancer node per enabled Availability Zone. A target registered in `us-east-1a` will only receive traffic from the LB node in `us-east-1a` unless Cross-Zone Load Balancing is enabled. To distribute across boundaries seamlessly, enable cross-zone load balancing or register targets equally across all active AZs.

![Cross-Zone Load Balancing](<assets/Cross-Zone Load Balancing.png>)

<a id="scalability-vs-high-availability"></a>
### ⚖️ 1.2 Scalability vs. High Availability — Core Distinction

```text
  SCALABILITY                          HIGH AVAILABILITY
  ─────────────────────────────        ──────────────────────────────────
  Handle MORE load by adapting    vs.  SURVIVE a failure without downtime

  [ Two Dimensions ]
  Vertical  ──► Make one box BIGGER    [ Requirement ]
  Horizontal ──► Add MORE boxes        Minimum 2 Availability Zones

```

| Dimension | Definition | AWS Example |
| --- | --- | --- |
| **Vertical Scalability** | Increase instance size (scale up/down) | `t2.micro` → `t2.large` |
| **Horizontal Scalability** | Increase instance count (scale out/in) | 1 EC2 → 10 EC2 via ASG |
| **High Availability** | Run across ≥2 AZs to survive a data center loss | Multi-AZ ASG + ELB Multi-AZ |

### 1.3 Vertical Scalability

* Increases the raw power of a **single** instance—allocating more RAM, storage, or vCPUs.
* Natural fit for **non-distributed systems** like relational databases (e.g., RDS or ElastiCache support vertical resizing).
* A hard ceiling exists: you cannot scale past the largest available physical instance type provided by the cloud provider hardware.

```text
  t2.nano   ──► t2.micro ──► t2.large ──► ... ──► u-12tb1.metal
  [0.5G RAM]                [8G RAM]              [12.3TB RAM / 448 vCPUs]
  [1 vCPU ]                                        Hardware Limit ───┘

```

### 1.4 Horizontal Scalability (Elasticity)

* Increases the **number of computing instances** rather than upgrading their individual hardware sizing.
* Implies a **distributed system** topology—traffic is programmatically split across multiple processing nodes.
* Cloud-native pattern; trivially achievable via EC2 instances automated via Auto Scaling Groups.

```text
  Before:  [ App Server ]

  After:   [ App Server ] [ App Server ] [ App Server ]
                    ▲             ▲             ▲
                    └─────────────┴─────────────┘
                       Load Balancer distributes traffic

```

### 1.5 High Availability for EC2 — Quick Reference

| Strategy | Mechanism | AWS Service |
| --- | --- | --- |
| **Vertical Scaling** | Resize instance | Stop instance → Change instance type → Start instance |
| **Horizontal Scaling** | Add/remove instances | Auto Scaling Group (ASG) |
| **High Availability** | Run in ≥2 AZs | ASG Multi-AZ placement + ELB Multi-AZ routing |

---

<a id="part-ii-load-balancer-types"></a>
# ⚖️ Part II — Load Balancer Types

## 🧰 4-9. Load Balancer Classification Taxonomy

> ⚠️ **Classic Load Balancer (CLB) Notice:** CLB is a legacy service and should not be selected for new architectures. AWS recommends using Application Load Balancer (ALB), Network Load Balancer (NLB), or Gateway Load Balancer (GWLB) depending on workload requirements. CLB is included here only for certification context, migration planning, and comparison purposes.

<a id="load-balancer-overview"></a>
### 4.1 Load Balancer Overview

AWS offers four Elastic Load Balancing families. Choose the family by the traffic layer you must inspect, the protocols you must support, and the operational requirement hidden in the scenario.

| Type | Simple Definition | Use When | Certification Signal |
| --- | --- | --- | --- |
| **ALB** | Layer 7 HTTP/HTTPS load balancer that understands web request content. | Websites, APIs, containers, microservices, Lambda targets, WAF protection. | Path, host, header, query-string, WebSocket, gRPC, or WAF. |
| **NLB** | Layer 4 TCP/UDP/TLS load balancer optimized for extreme performance and static IPs. | Low latency, fixed IP allowlists, PrivateLink, TCP/UDP workloads. | Static IP, Elastic IP, source IP preservation, millions of requests. |
| **GWLB** | Layer 3 transparent traffic distribution for virtual network appliances. | Firewalls, IDS, IPS, packet inspection, appliance fleets. | Third-party security appliance or GENEVE. |
| **CLB** | Legacy load balancer for older EC2-Classic or early VPC architectures. | Existing legacy workloads only. | Avoid for new designs unless the question is explicitly legacy. |

<a id="application-load-balancer-alb"></a>
### 🌐 5. Application Load Balancer (ALB)

An **Application Load Balancer** operates at Layer 7 and understands HTTP semantics. It can inspect hostnames, paths, headers, query strings, methods, cookies, and source IP conditions before selecting a target group.

* **Why it exists:** Modern applications often need multiple services behind one domain, such as `/api`, `/images`, and `/admin`, each routed to different backend fleets.
* **When to use it:** Choose ALB for websites, REST APIs, gRPC services, WebSocket apps, ECS services, EKS ingress, Lambda targets, microservices, and AWS WAF integration.
* **Real-world example:** `api.example.com/v1/orders` routes to an ECS service, `/static/*` routes to EC2 web nodes, and `X-Tenant: beta` routes 10% of requests to a canary target group.
* **AWS implementation details:** ALB supports instance, IP, and Lambda targets. IP targets are common for ECS `awsvpc` networking and EKS pods. Lambda targets are useful when an HTTP route should invoke serverless code without running instances.
* **Exam tip:** Path-based routing, host-based routing, HTTP header rules, query-string routing, WebSocket, gRPC, Lambda targets, and WAF all point to ALB.

<a id="network-load-balancer-nlb"></a>
### ⚡ 6. Network Load Balancer (NLB)

A **Network Load Balancer** operates at Layer 4 and forwards TCP, UDP, or TLS traffic with very low latency. It does not inspect HTTP paths or headers; it routes based on connection-level information.

* **Why it exists:** Some workloads need raw transport performance, source IP preservation, fixed IP addresses, or protocols that are not HTTP.
* **When to use it:** Choose NLB for gaming, IoT, VoIP, high-throughput TCP APIs, TLS pass-through, fixed partner allowlists, and PrivateLink endpoint services.
* **Real-world example:** A payment partner requires fixed IP allowlisting, so the architecture exposes an internet-facing NLB with Elastic IPs and forwards traffic to internal application targets.
* **AWS implementation details:** NLB provides one static IP per enabled AZ, can use Elastic IPs for internet-facing balancers, preserves client source IP for many target configurations, and is the load balancer type used for AWS PrivateLink endpoint services.
* **Exam tip:** Static IP, Elastic IP, PrivateLink, TCP, UDP, or source IP preservation usually means NLB.

<a id="gateway-load-balancer-gwlb"></a>
### 🛡️ 7. Gateway Load Balancer (GWLB)

A **Gateway Load Balancer** operates at Layer 3 and transparently distributes IP packets across security appliances using the GENEVE protocol.

* **Why it exists:** Enterprises need scalable inspection fleets without manually wiring traffic to individual firewall instances.
* **When to use it:** Choose GWLB for third-party firewalls, IDS, IPS, deep packet inspection, egress inspection, and centralized security VPC architectures.
* **Real-world example:** VPC route tables send outbound traffic to a GWLB endpoint; GWLB forwards packets to a fleet of firewall appliances; inspected traffic returns symmetrically and continues to the internet or destination VPC.
* **AWS implementation details:** GWLB is commonly paired with Gateway Load Balancer Endpoints and route table entries so traffic inspection stays transparent to applications.
* **Exam tip:** Any scenario describing a fleet of third-party firewalls, IDS, IPS, transparent inspection, or GENEVE maps to GWLB.

<a id="classic-load-balancer-clb"></a>
### 🕰️ 8. Classic Load Balancer (CLB)

Classic Load Balancer is the legacy ELB generation. It can balance basic HTTP/HTTPS and TCP/SSL traffic, but lacks modern ALB listener rules, native WAF integration, advanced target types, and the clearer L7/L4 separation used by ALB and NLB.

* **Use it only for:** Legacy environments, migration planning, or certification questions that explicitly mention old architectures.
* **Exam tip:** For new workloads, prefer ALB, NLB, or GWLB. CLB is rarely the best answer unless the question says legacy.


### 6.1 Detailed Structural Performance Matrix

| Feature | Classic Load Balancer (CLB) - Legacy | Application Load Balancer (ALB) | Network Load Balancer (NLB) | Gateway Load Balancer (GWLB) |
| --- | --- | --- | --- | --- |
| **OSI Layer** | Layer 4 & Layer 7 | Layer 7 | Layer 4 | Layer 3 |
| **Supported Protocols** | HTTP, HTTPS, TCP, SSL | HTTP, HTTPS, WebSocket, gRPC | TCP, UDP, TLS | IP Packets (GENEVE) |
| **Path-Based Routing** | ❌ | ✅ | ❌ | ❌ |
| **Host-Based Routing** | ❌ | ✅ | ❌ | ❌ |
| **Static / Elastic IP** | ❌ | ❌ (Dynamic IPs assigned) | ✅ (1 Fixed IP per AZ) | ❌ |
| **SNI (Multi-Cert Support)** | ❌ | ✅ | ✅ | N/A |
| **Target Types** | EC2 Classic Instances | Instance, ECS/IP, Lambda, ALB | Instance, Private IP, ALB | Instance, Private IP |
| **AWS WAF Native Integration** | ❌ | ✅ | ❌ | ❌ |
| **Native Access Logs** | ✅ | ✅ | ✅ | ❌ |


### 6.2 Architectural Deep Dive: NLB Static IP & AWS PrivateLink

* NLB provisions a fixed, non-changing IP address for each mapping Availability Zone, allowing systems administrators to map Elastic IPs directly onto the ingress interfaces.
* Integrates natively with **AWS PrivateLink** architecture (VPC Endpoint Services). This empowers enterprise topologies to expose secure, internal corporate microservice APIs privately across external VPC setups and distinct AWS organizational accounts without transiting the public internet.

---

<a id="load-balancer-comparison-matrix"></a>
## 9. Load Balancer Comparison Matrix

| Architectural Evaluator | Application Load Balancer (ALB) | Network Load Balancer (NLB) | Gateway Load Balancer (GWLB) |
| --- | --- | --- | --- |
| **OSI Layer Operation** | Layer 7 (Application) | Layer 4 (Transport) | Layer 3 (Network Packet Core) |
| **Routing Intelligence** | HTTP content parameters | Sockets, IP addresses, Port numbers | GENEVE protocol encapsulation |
| **Latency Signature** | Low millisecond scale (~sub-10ms) | Ultra-low microsecond scale (~μs) | Low millisecond scale |
| **Static IP Allotment** | No (IP interfaces fluctuate) | Yes (1 Static IP per AZ) | No |
| **Primary Target Workload** | Modern Web Apps / Microservices | High-performance Gaming / IoT / TCP APIs | Fleet management of 3rd-party Firewalls, IPS/IDS appliances |

---

<a id="part-iii-traffic-routing"></a>
# 🧭 Part III — Traffic Routing

<a id="target-groups"></a>
<a id="nlb-to-alb-chaining"></a>
## 🔗 10. Target Groups & NLB → ALB Chaining

A **target group** is the named pool of destinations that receives traffic from a load balancer rule. Targets can be EC2 instances, private IP addresses, Lambda functions for ALB, or another ALB behind an NLB.

| Target Group Concept | Plain-English Explanation | Practical Example |
| --- | --- | --- |
| **Listener** | The front-door port and protocol on the load balancer. | HTTPS listener on port `443`. |
| **Rule** | A condition/action pair evaluated by the listener. | If path is `/api/*`, forward to `tg-api`. |
| **Target group** | The backend pool selected by a rule. | `tg-api` contains ECS task IPs. |
| **Target** | One registered destination inside the group. | EC2 instance, pod IP, Lambda function, or ALB target. |

```text
[ Internet ] ──► [ NLB — Static IP Ingress ] ──► [ ALB Target Group ] ──► [ EC2 Application Fleet ]
                       (Layer 4 Layer)                  (Layer 7 Layer)

```

### 7.1 Cluster Coordination & Requirements

* **NLB → ALB Chaining:** Combines the distinct advantages of both load balancer archetypes. Place an NLB at the absolute network edge to present a fixed, whitelistable Elastic IP address to consumer networks, then forward traffic directly to an Application Load Balancer target group to carry out intricate Layer 7 content-based routing.
* **Cross-VPC Boundaries:** Target groups can evaluate resources by private VPC IP allocation fields. Chained components must reside within identical VPC regions, except when targeting AWS Lambda serverless functions.
* **Health Evaluation Flow:** Health metrics are maintained and performed independently per target group level. The NLB actively gauges the operational state of the ALB endpoints, while the ALB independently tracks individual EC2 server statuses.

---

<a id="alb-routing-rules"></a>
## 🧭 11. Application Load Balancer Routing Engine

The Application Load Balancer parses listener rule configurations sequentially to evaluate downstream structural conditions.

```text
[ HTTPS :443 Ingress Listener ]
               │
               ├── Rule 10: IF Host = api.example.com AND Path = /v1/* ──► [ tg-api-v1 ]
               ├── Rule 20: IF Path = /images/* ──► [ tg-images ]
               ├── Rule 30: IF Header X-Tenant = beta                  ──► [ tg-canary ] (Weighted 10%)
               └── Rule Default: Fallback Processing Catch-All          ──► [ tg-app-default ]

```

### 9.1 Listener Rule Evaluation Conditions

| Condition Rule Metric | Concrete Real-World Example | ALB Native Support |
| --- | --- | --- |
| **Host Header** | `api.example.com` or `admin.internal.net` | ✅ Supported |
| **Path Pattern** | `/orders/*`, `/v2/users/search`, or `*.png` | ✅ Supported |
| **Query String Key/Value** | `?platform=mobile` or `?version=beta` | ✅ Supported |
| **HTTP Header Attributes** | `X-Tenant-Id: company_abc` | ✅ Supported |
| **Source IP Address CIDR** | `192.168.10.0/24` or `10.0.0.0/16` | ✅ Supported |
| **HTTP Request Method** | `POST`, `GET`, `DELETE`, or `PUT` | ✅ Supported |

---

## 11.2 Advanced Request Routing Internals

### 14.1 Incremental Rule Chain Mechanics

* Listener rules are processed in strict priority order (from low numbers to high numbers). The first matching rule applies, and any subsequent rules are ignored.
* The default catch-all rule handles any requests that do not match the explicit conditions above it.
* Each single application listener supports up to 100 rules simultaneously.

<a id="cross-az-costs"></a>
### 14.2 Financial & LCU Billing Realities

* **ALB Billing Architecture:** Calculated based on LCU hours consumed. 1 LCU metrics are bound by: 25 new connections/sec, 3,000 active connections/min, 1 GB of data processed per hour, or 1,000 active rule evaluations/sec.
* **NLB Billing Architecture:** Measured via NLCU hours. 1 NLCU tracks: 800 new connections/sec, 100,000 active connections/min, or 1 GB of data throughput per hour.
* **Cross-AZ Data Transits:** If Cross-Zone Load Balancing is disabled on an NLB, any cross-AZ traffic routing will incur data transfer fees. Conversely, ALB Cross-Zone load balancing is native, always active, and free of data transfer charges.

### 14.3 Cross-Account Sharing & Global Architecture

* **AWS PrivateLink:** Map an internal NLB behind a VPC Endpoint Service to safely share core backend microservices cross-organizationally without dealing with complex VPC peering or internet routing overhead.
* **Route 53 Failover Routing:** Configure multi-region disaster recovery paths by setting active-passive failover targets across separate load balancers located in distinct geographical regions.
* **AWS Global Accelerator:** Leverages AWS's private global fiber-optic network to route traffic to your local regional balancers via anycast static IP addresses, reducing latency for distributed users.

---

<a id="weighted-target-groups"></a>
## ⚖️ 12. ALB Listener Rules & Weighted Target Groups

Weighted Target Groups enable safe application deployments by splitting traffic proportionally between stable and experimental application paths.

```bash
# Production Canary Deployment: Route 90% of traffic to stable, 10% to canary
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:listener/app/my-alb/abc/def \
  --priority 30 \
  --conditions Field=path-pattern,Values='/api/*' \
  --actions Type=forward,ForwardConfig='{"TargetGroups":[{"TargetGroupArn":"arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/tg-stable/111","Weight":90},{"TargetGroupArn":"arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/tg-canary/222","Weight":10}]}'

```

### 17.1 Rule Isolation Architecture

* **Multi-Tenant SaaS Isolation:** Leverage unique listener rule condition arrays to route distinct enterprise business tenants down individual, isolated target group boundaries based on subdomains or custom HTTP header contexts.
* **Canary Deployments:** Control traffic distribution during releases by configuring weighted routing rules (e.g., routing a strict 5% chunk of live API traffic to a newly deployed target group while maintaining a 95% baseline on the stable fleet).

---

<a id="part-iv-health-availability"></a>
# 💓 Part IV — Health & Availability

<a id="health-checks"></a>
<a id="health-check-grace-period"></a>
<a id="slow-start"></a>
## 💓 14-17. Health Checks, Tiering & Lifecycle Management

Health checks are the programmatic mechanism by which a load balancer determines whether a downstream target is eligible to receive production traffic.

```text
  [ ELB ]  ──► HTTP GET /health (port 4567)  ──►  [ EC2 Instance ]
                                                         │
                           HTTP 200 OK ◄─────── Healthy ┤
                           HTTP 5xx   ◄─────── Unhealthy┘ (removed from rotation)

```

### 3.1 Core Blueprint Features

* **Live Target Probing:** Evaluates active targets without downtime, using highly configurable intervals, timeouts, and success thresholds.
* **Cross-Boundary Isolation:** Unhealthy targets are automatically removed from rotation per Availability Zone, preserving global availability across the remaining healthy infrastructure zones.

<a id="health-check-parameters"></a>
### 3.2 Health Check Parameters

| Parameter | Application Load Balancer (ALB) | Network Load Balancer (NLB) | Gateway Load Balancer (GWLB) |
| --- | --- | --- | --- |
| **Protocol** | HTTP, HTTPS | TCP, HTTP, HTTPS | GENEVE |
| **Port** | traffic-port / explicit override | traffic-port / explicit override | 80 / explicit override |
| **Healthy threshold** | 2–10 consecutive successes | 2–10 consecutive successes | 2–10 consecutive successes |
| **Unhealthy threshold** | 2–10 consecutive failures | 2–10 consecutive failures | 2–10 consecutive failures |
| **Interval** | 5–300 seconds | 10–30 seconds | 10 seconds |
| **Timeout** | 2–120 seconds | 10 seconds | 5 seconds |
| **Success codes** | 200–399 (user configurable) | N/A (TCP Handshake connection) | N/A |

> ⚠️ **Exam Rule:** Only a matching, successfully returned status code marks an instance as healthy. For HTTP/HTTPS, the default is strictly `200 OK`, but can be modified to accept ranges (e.g., `200-399`).

### 3.2.1 Health Check Concepts in Plain English

| Concept | Plain-English Explanation | Real-World Example | Common Mistake |
| --- | --- | --- | --- |
| **Healthy threshold** | Number of successful checks required before a target returns to service. | Two passing `/health` responses mark a recovered instance healthy. | Setting it too high delays recovery. |
| **Unhealthy threshold** | Number of failed checks required before traffic stops. | Three failed responses remove a broken instance from rotation. | Setting it too low can remove targets during short deploy blips. |
| **Interval** | How often the load balancer checks each target. | Check `/health` every 15 seconds. | Very frequent checks can create noisy application logs. |
| **Timeout** | How long the balancer waits for a response. | A target must answer within 5 seconds. | Timeout longer than interval creates confusing failure behavior. |
| **Success codes** | HTTP codes treated as healthy. | Accept `200-399` for an app that redirects `/health`. | Returning `500` while dependencies are down but still reporting healthy. |

### 3.2.2 Grace Period vs Slow Start vs Deregistration Delay

| Feature | Applies To | Purpose | Example |
| --- | --- | --- | --- |
| **Health Check Grace Period** | Auto Scaling Group | Prevents ASG from terminating a new instance before the app finishes booting. | Give a Java service 300 seconds before ASG health evaluation. |
| **Slow Start** | Target Group | Gradually increases traffic to a newly healthy target. | Warm a cache-heavy node over 120 seconds. |
| **Deregistration Delay** | Target Group | Lets in-flight requests finish before removing a target. | Keep long API calls alive during scale-in or deployment. |

> ⚠️ **Exam Tip:** Grace period protects new instances from premature ASG replacement, slow start protects new targets from immediate traffic spikes, and deregistration delay protects existing requests during removal.

### 3.3 Advanced Optimization Frameworks

#### 1. ASG Health Check Grace Period

* **Mechanics:** Delays ASG EC2 health evaluation checks for a designated window post-launch, allowing application runtimes to fully bootstrap.
* **Operational Advantage:** Prevents premature termination loops of slow-booting monolithic applications.
* **Profile:** Match exact AMI boot time + application cold-start warm-up latency. The default value is 300 seconds.

#### 2. Target Group Slow-Start

* **Mechanics:** Gradually ramps up the volume of production traffic forwarded to newly registered backend targets over a tunable 30 to 900-second window.
* **Performance Profile:** Prevents cold-start request overload on cache-heavy setups or JIT compilation backends.

#### 3. Health Check Override & Anomaly Detection

* **Mechanics:** Leverage CloudWatch anomaly detection algorithms on the `UnHealthyHostCount` metric to alert systems engineers or trigger runbooks before full AZ failure occurs.
* **Data Protection:** Protects availability bounds by orchestrating rollback mechanisms via an ASG *Instance Refresh*.

---

<a id="deregistration-delay"></a>
## 🧹 18. Deregistration Delay & Connection Draining

The **DeregistrationDelay** attribute is a critical timeout flag that dictates what happens to in-flight requests when a target is actively removed from service.

```text
            [ Target Deregistration Event ]
                           │
       ┌───────────────────┴───────────────────┐
       ▼                                       ▼
[ Connection Draining: ON ]             [ Connection Draining: OFF ]
Timeout: 300s (configurable)            Timeout: 0s
ACTION: Complete in-flight requests      ACTION: Connections terminated

```

### 2.1 Production Configuration Behavior

* **ALB / NLB Target Groups:** By default, `deregistration_delay.timeout_seconds` is set to **300 seconds**—the load balancer waits up to 5 minutes for active, in-flight requests to complete before severing the socket connection.
* **CLB:** The Classic equivalent is known as **Connection Draining**, which is disabled by default. When active, it can be configured between 1 and 3600 seconds.
* **Orchestration Customization:** Modify via Console, CLI, or IaC to closely match your application's request duration profiles. Short REST APIs can be lowered to 30–60s, whereas long-running reporting engines or file uploads should be sustained at 300s+.

```text
┌───────────────┬──────────────────────┬────────────┬─────────────┬──────────────────────────┐
│ Target Group  │ Target ID            │ Port       │ Health      │ Deregistration Delay     │
├───────────────┼──────────────────────┼────────────┼─────────────┼──────────────────────────┤
│ tg-app-alb    │ i-01a2b3c4d5e6f7890  │ 80         │ healthy     │ 60s                      │
├───────────────┼──────────────────────┼────────────┼─────────────┼──────────────────────────┤
│ tg-api-nlb    │ 10.0.12.45           │ 443        │ healthy     │ 30s                      │
└───────────────┴──────────────────────┴────────────┴─────────────┴──────────────────────────┘

```

---

<a id="part-v-security"></a>
# 🔐 Part V — Security

## 🔐 19-23. Data Security & Cryptographic Compliance

<a id="tls-ssl"></a>
### 8.1 Encryption Mechanics

* **TLS Ingress Termination:** The ALB/NLB accepts and terminates inbound client TLS connections at the edge load balancer layer, loading X.509 cryptographic certificates directly from AWS Certificate Manager (ACM). Internally, traffic can navigate over unencrypted HTTP protocol within the private security perimeter of the corporate VPC to decrease instance CPU overhead.
* **End-to-End TLS Enforcement:** For highly regulated compliance environments (HIPAA, PCI-DSS), re-encryption protocols can be configured. The load balancer terminates public client TLS, inspects traffic rules, and initializes a secondary secure TLS session to the backend instances using internal self-signed certificates.
* **Server Name Indication (SNI):** Allows a single ingress load balancer interface to host multiple distinct TLS certificates simultaneously. The balancer parses the inbound client TLS handshake hostname parameter to select the corresponding certificate matching the domain requested.

<a id="aws-certificate-manager-acm"></a>
<a id="server-name-indication-sni"></a>
### 8.2 Step-by-Step Blueprint: ACM Certificate Provisioning & ALB SNI

```bash
# 1. Request a public, wildcard TLS certificate via AWS ACM
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names "*.example.com" \
  --validation-method DNS

# 2. Spin up a secure production HTTPS Listener on an active ALB
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/abc \
  --protocol HTTPS --port 443 \
  --certificates CertificateArn=arn:aws:acm:us-east-1:123456789012:certificate/abc123 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/app-tg/xyz

# 3. Append an alternate multi-tenant domain certificate via SNI maps
aws elbv2 add-listener-certificates \
  --listener-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:listener/app/my-alb/abc/def \
  --certificates CertificateArn=arn:aws:acm:us-east-1:123456789012:certificate/def456

```

<a id="tls-policies"></a>
### 8.3 TLS Security Policies

| Security Policy Name | Supported TLS Versions | Cipher Suite Constraints | Primary Core Use Case |
| --- | --- | --- | --- |
| `ELBSecurityPolicy-TLS13-1-2-2021-06` | TLS 1.2, TLS 1.3 | Strict Forward Secrecy Only | High-security modern web apps |
| `ELBSecurityPolicy-FS-1-2-Res-2020-10` | TLS 1.2 Only | Forward Secrecy Restricted | Enterprise regulatory compliance |
| `ELBSecurityPolicy-2016-08` | TLS 1.0, 1.1, 1.2 | Broad, inclusive cipher mix | Legacy browser support (Avoid) |

<a id="security-group-design"></a>
### 8.4 Security Group Lockdown Blueprint

To enforce strict perimeter isolation, EC2 computing layers should remain completely blind to direct public internet traffic:

1. **ALB Ingress Security Group:** Accept traffic strictly from `0.0.0.0/0` (or consumer corporate networks) on port `80` and `443`. Route outbound traffic specifically via an explicit rule targeting the internal VPC CIDR or backend target group security group.
2. **EC2 Host Instance Security Group:** Configure the inbound network rule to allow traffic strictly on application ports (e.g., `80`, `8080`, `4567`) where the **Source ID** equals the unique security group identifier (`sg-xxxxxx`) of the ingress Load Balancer.

### 8.5 WAF, Shield & Least Privilege Design

* **AWS WAF with ALB:** Attach WAF web ACLs to ALBs to filter SQL injection, cross-site scripting, bad bots, IP reputation lists, and rate-based attacks before traffic reaches targets.
* **AWS Shield Standard:** Automatically protects AWS edge and ELB resources from common network and transport-layer DDoS attacks at no extra cost.
* **AWS Shield Advanced:** Adds enhanced DDoS detection, cost protection, and access to the AWS DDoS Response Team for high-risk production workloads.
* **Least privilege security groups:** Public users should reach only the load balancer. Backend instances should accept traffic only from the ALB security group or trusted internal sources.
* **Certificate lifecycle:** Prefer ACM-managed public certificates for automatic renewal. Avoid manually installed certificates on every EC2 instance unless end-to-end TLS compliance requires it.

> ⚠️ **Exam Tip:** WAF protects at Layer 7 and integrates with ALB. Shield protects against DDoS. Security groups enforce network access boundaries.

---

<a id="part-vi-auto-scaling"></a>
# 📈 Part VI — Auto Scaling

<a id="amis"></a>
<a id="launch-templates"></a>
## 🧬 24-25. Amazon Machine Images & Launch Templates

An **Amazon Machine Image (AMI)** coupled with a **Launch Template** provides an immutable, version-controlled compute blueprint for deploying your auto-scaling instance fleets.

```text
  [ Golden AMI Build ] ──► (Versioned) ──► [ Launch Template v12 ]
         │                                            │
         └─────────────────────┬──────────────────────┘
                               ▼
                    [ ASG Instance Refresh ]

```

### 4.1 Image Provisioning Classifications

| Type | Use Case | Persistence / Maintenance Profile |
| --- | --- | --- |
| **AWS Marketplace AMI** | Base Operating System / Off-the-shelf software | Publicly distributed, vendor-maintained |
| **Custom Golden AMI** | Pre-baked application dependencies, binaries, agent configurations | Private, immutable, version-controlled by internal CI/CD |
| **Launch Template** | Core instance configuration plane | Versioned configurations, declarative parameters |

### 4.2 The Launch Template Component Plane

A Launch Template defines the structural layout of every EC2 instance provisioned inside the ASG bounds:

* **AMI ID:** The precise base operating system layer or golden image.
* **Instance Type:** Computing class profile defining vCPU, RAM, and network capabilities.
* **User Data:** Base64-encoded shell script executed one time during boot phase initialization.
* **Security Groups:** Statefully managed host network perimeter firewall rules.
* **IAM Instance Profile:** Associated execution role credentials mapping AWS resource access.
* **Block Device Mappings:** Elastic Block Store (EBS) volume schemas including sizes, types, and `DeleteOnTermination` true/false flags.
* **Metadata Options:** Strict enforcement configurations for IMDSv2 security hardening.

> ⚠️ **Exam Tip:** Launch Templates are preferred over Launch Configurations because they are versioned and support modern EC2 features such as mixed instances, Spot options, T2/T3 unlimited settings, and advanced metadata options.

### 4.3 The Enterprise Golden Image Pipeline

1. Compile and bake the application image using automated tooling (e.g., Packer or AWS EC2 Image Builder) to install all operating system dependencies upfront.
2. Publish a new version of the AWS Launch Template referencing the newly produced AMI ID.
3. Execute an ASG `Instance Refresh` complete with checkpoint evaluation rolling rollbacks.
4. Legacy computing nodes gracefully drain connections via the *Deregistration Delay* settings, while newly minted nodes step into production traffic streams smoothly using *Slow-Start*.

```bash
# 1. Append a new Launch Template version mapping a fresh Golden AMI
aws ec2 create-launch-template-version \
  --launch-template-id lt-0123456789abcdef0 \
  --version-description "app-v2.4" \
  --source-version 12 \
  --launch-template-data '{"ImageId":"ami-0newgolden123"}'

# 2. Trigger an automated ASG rolling Instance Refresh
aws autoscaling start-instance-refresh \
  --auto-scaling-group-name app-asg \
  --preferences '{"MinHealthyPercentage":90,"InstanceWarmup":120}'

```

---

<a id="auto-scaling-groups"></a>
## 📈 26. Auto Scaling Group Core Architecture

An **Auto Scaling Group (ASG)** represents a fleet orchestration boundary engineered to automatically maintain a dynamic or static capacity allocation of instances distributed evenly across multiple physical Availability Zones.

![Auto Scaling Group in AWS](<assets/Auto Scaling Group in AWS.png>)

```text
       [ ASG Control Plane ]
                 │
  ┌──────────────┼──────────────┐
  ▼              ▼              ▼
us-east-1a     us-east-1b     us-east-1c
 [EC2] [EC2]    [EC2] [EC2]    [EC2] [EC2]

```

### 5.1 Production Fleet Capacity Profiles

| Scaling Dimension | Fleet Orchestration Mechanism | Typical Target Value |
| --- | --- | --- |
| **Minimum Capacity** | Floor boundary—the fleet size will never scale below this value under any operational trigger. | 2 |
| **Desired Capacity** | Current operational baseline target—ASG maintains this total instance volume continuously. | 4 |
| **Maximum Capacity** | Ceiling boundary—the scaling loop will never add instances beyond this hard count limit. | 20 |
| **AZ Balance** | Automated balancing logic—enforces symmetrical instance allocation counts across mapped subnets. | Enabled / Enforced |
| **Health Check Type** | Evaluates instance state via EC2 hypervisor checks or direct downstream ELB endpoint feedback. | ELB (Recommended for web applications) |

### 5.2 ASG Troubleshooting Signals

| Signal | What It Shows | Common Root Cause |
| --- | --- | --- |
| **Scaling activities** | Historical launches, terminations, failures, and policy decisions. | Invalid AMI, missing IAM profile, subnet capacity, failed health checks. |
| **Group metrics** | Desired, minimum, maximum, pending, in-service, and terminating capacity. | Scaling policy too aggressive or max capacity too low. |
| **Suspended processes** | ASG processes manually paused by operators. | Launch, terminate, health check, or AZ rebalance accidentally suspended. |
| **ELB health check failures** | Application-level health failures. | `/health` endpoint returns 500, app boot takes longer than grace period, security group blocks LB traffic. |

> ⚠️ **Exam Tip:** If EC2 status checks pass but the application returns errors, configure the ASG health check type as `ELB` so the group replaces application-broken instances.

---

<a id="scaling-policies"></a>
<a id="instance-refresh"></a>
## 🎚️ 27-28. ASG Scaling Modification & Elastic Capacity

Auto Scaling Groups scale your compute footprint dynamically through automated policies that react to real-time workloads.

```bash
# 1. Inspect and describe current active scaling policy boundaries
aws autoscaling describe-policies --auto-scaling-group-name app-asg

# 2. Establish a Target Tracking policy maintaining an exact ALB Request Count baseline
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name app-asg \
  --policy-name alb-request-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{"PredefinedMetricSpecification":{"PredefinedMetricType":"ALBRequestCountPerTarget","ResourceLabel":"app/my-alb/abc/targetgroup/app-tg/xyz"},"TargetValue":1000.0}'

```

### 12.1 Dynamic Scaling Classifications

* **Target Tracking Scaling:** The standard deployment choice. Tracks a specified metric baseline (e.g., maintain average fleet CPU allocation at exactly 50%, or sustain `ALBRequestCountPerTarget` at 1,000 requests per node). The scaling loop calculates and handles scaling sizes automatically.
* **Step Scaling:** Dynamically scales infrastructure by evaluating the specific magnitude of a CloudWatch alarm breach. For instance, if CPU utilization jumps $> 70\%$, provision $+2$ instances; if it spikes severely $> 90\%$, provision $+5$ instances immediately.
* **Scheduled Scaling:** Executes capacity shifts based on time schedules (e.g., cron configurations). Increases desired instance counts to 15 every Friday at 4:00 PM ahead of weekend retail peaks, and drops down to 2 every Monday morning.
* **Predictive Scaling:** Machine learning-driven optimization model. Historically evaluates your infrastructure's metrics over a rolling two-week window to forecast daily patterns and pre-provision instance counts hours ahead of anticipated traffic spikes.

| Scaling Policy | Best Use Case | Common Mistake |
| --- | --- | --- |
| **Target Tracking** | Keep CPU, request count, or custom metric near a desired target. | Setting a target that is too high, causing late scale-out. |
| **Step Scaling** | Apply different scale-out sizes based on alarm severity. | Creating overlapping alarm thresholds. |
| **Scheduled Scaling** | Known business events, batch windows, and predictable traffic peaks. | Forgetting timezone and daylight-saving behavior. |
| **Predictive Scaling** | Repeating traffic patterns such as daily retail or office-hour workloads. | Expecting it to handle sudden one-off spikes alone. |

### 12.2 Production Migration Blueprint: Rolling Deployments via Instance Refresh

```bash
# 1. Update and commit a new Launch Template layout configuration version
aws ec2 create-launch-template-version --launch-template-id lt-abc --version-description "blue-green-v2"

# 2. Launch an automated Instance Refresh enforcing healthy checkpoint steps
aws autoscaling start-instance-refresh \
  --auto-scaling-group-name app-asg \
  --preferences '{"MinHealthyPercentage":90,"InstanceWarmup":120,"CheckpointPercentages":[50],"CheckpointDelay":600}'

```

### 12.3 Operational Limitations & Cooldown Guardrails

* **Default Cooldown Period:** Pauses subsequent scaling actions for a set timeframe (default is 300 seconds) after a scaling event completes, allowing infrastructure metrics to stabilize before adding or removing more instances.
* **Target Tracking Auto-Management:** Target Tracking calculations automatically modulate internal scale-in cooldown metrics. Manual cooldown overwrites should be avoided here.
* **AMI Pre-Baking Strategy:** Pre-bake configuration dependencies, dependencies, and code assets directly into the Golden AMI to minimize boot-up times, reduce scaling latency, and optimize your systems' responsiveness to traffic spikes.

### 12.4 Blue/Green and Canary Deployment Patterns

* **Blue/Green with Target Groups:** Run the blue version and green version in separate target groups, validate green health, then shift listener rules or weights to cut traffic over.
* **Canary with Weighted Target Groups:** Send a small traffic slice, such as 5% or 10%, to a new version while the stable target group still receives most traffic.
* **Instance Refresh Rollback:** Use checkpoints and CloudWatch alarms so an ASG refresh pauses or rolls back when health or latency degrades.
* **Practical guardrail:** Pair deployments with deregistration delay and slow start so old targets finish active requests and new targets warm up gradually.

---

<a id="mixed-instance-fleets"></a>
## 🏎️ 29. ASG Fleet Architectures with Mixed Instances

```yaml
# Production Infrastructure-as-Code Blueprint: Mixed Instances Policy
AutoScalingGroup:
  Type: AWS::AutoScaling::AutoScalingGroup
  Properties:
    MixedInstancesPolicy:
      LaunchTemplate:
        LaunchTemplateSpecification:
          LaunchTemplateId: !Ref AppLaunchTemplate
          Version: !GetAtt AppLaunchTemplate.LatestVersionNumber
      InstancesDistribution:
        OnDemandBaseCapacity: 2
        OnDemandPercentageAboveBaseCapacity: 50
        SpotAllocationStrategy: capacity-optimized
    MinSize: 2
    MaxSize: 20
    VPCZoneIdentifier: [subnet-a, subnet-b, subnet-c]
    TargetGroupARNs: [!Ref AppTargetGroup]

```

### 13.1 Fleet Topology Comparison Matrix

| Fleet Topology Mode | Cost Profile | Availability Guarantee | Ideal Operational Use Case |
| --- | --- | --- | --- |
| **On-Demand Only** | Highest baseline expenses | Maximum architecture survival | Critical core master applications, stateful workloads, baseline databases |
| **Spot Only** | Lowest costs (up to 90% discount) | Interruptible (2-minute warning evictions possible) | Batch processing engines, stateless workers, fault-tolerant rendering pools |
| **Mixed Instances Fleet** | Optimized cost boundaries | Balanced high availability | Standard web application fleets, production API endpoints |
| **Attribute-Based Selection** | Highly flexible pricing matrices | Variable based on constraints | Graviton architecture mixed structures, dynamic testing matrices |

### 13.2 Spot Instance Interruption Strategy

Spot Instances can reduce compute cost dramatically, but AWS can reclaim capacity with a two-minute interruption notice. Production ASGs should diversify across instance families, sizes, and Availability Zones, and use `capacity-optimized` allocation for interruption-sensitive workloads.

| Strategy | Why It Matters |
| --- | --- |
| **On-Demand base capacity** | Keeps a reliable minimum fleet online even when Spot markets shift. |
| **Multiple instance types** | Improves the chance that ASG can find replacement capacity. |
| **Capacity-optimized Spot** | Favors Spot pools with better availability instead of only lowest price. |
| **Graceful shutdown hook** | Uses lifecycle hooks or interruption handlers to drain work before termination. |

> ⚠️ **Exam Tip:** Spot is excellent for fault-tolerant and stateless workloads. Avoid Spot-only designs for critical stateful systems that cannot tolerate interruption.

---

<a id="asg-lifecycle-states"></a>
## 🔄 30-32. ASG Instance Lifecycle State Machine

Instances within an Auto Scaling Group transition through a sequence of operational states as they spin up, serve traffic, or spin down.

```text
[Pending] ──► [Pending:Wait] ──► [InService] ──► [Terminating] ──► [Terminating:Wait] ──► [Terminated]
                                    ▲  │
                       [Standby] ◄──┘  └──► [Warm Pool (Stopped/Warmed)]

```

### 15.1 Comprehensive Instance State Matrix

| Lifecycle State | Actively Receives Traffic | Counts Toward Group Capacity | Billing Active Status |
| --- | --- | --- | --- |
| **Pending** | ❌ No | ✅ Yes | ✅ Yes (EC2 Instance launched) |
| **Pending:Wait** | ❌ No (Lifecycle Hook pause) | ✅ Yes | ✅ Yes |
| **InService** | ✅ Yes (Registered with ELB) | ✅ Yes | ✅ Yes |
| **Standby** | ❌ No (Pulled for debugging) | ❌ No | ✅ Yes |
| **Terminating** | ❌ No (Connection draining active) | ✅ Yes | ✅ Yes |
| **Terminating:Wait** | ❌ No (Lifecycle Hook pause) | ✅ Yes | ✅ Yes |
| **Terminated** | ❌ No | ❌ No | ❌ No (Resource released) |

<a id="warm-pools"></a>
<a id="lifecycle-hooks"></a>
### 15.2 Under the Hood of ASG Lifecycle Customizations

* **Lifecycle Hooks:** Pause an instance during its `Pending` or `Terminating` phase to run automated operational scripts (e.g., downloading system updates, configuring log shippers, or backing up state via Amazon SNS, SQS, or AWS Lambda). Holds the state for up to 3,600 seconds.
* **Warm Pools:** Maintains a reserve of pre-initialized, booted instances in a `Stopped` or `Warmed` state. This configuration bypasses the cold-start boot sequence, allowing instances to scale out into production up to 80% faster.
* **Standby State Utility:** Allows operators to temporarily isolate a specific instance from traffic for troubleshooting or live patching without terminating the node or disrupting ASG capacity counts.

### 15.3 Termination Policies

Termination policies control which instance an ASG removes during scale-in. The default behavior balances Availability Zones first, then considers factors such as launch template age, billing, and proximity to the next billing hour for older purchasing models.

Use custom termination policies when you need predictable scale-in behavior, such as terminating oldest instances first during an AMI refresh or protecting newer capacity during a rollout.

---

<a id="part-vii-monitoring-cost"></a>
# 📊 Part VII — Monitoring & Cost

## 📡 33-36. ELB Observability, Logging & Billing Architecture

<a id="access-logs"></a>
### 18.1 Compliance Logging Access

```bash
# Enable continuous ALB Access Logging directly into a secure S3 bucket location
aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/abc \
  --attributes Key=access_logs.s3.enabled,Value=true \
             Key=access_logs.s3.bucket,Value=my-alb-logs \
             Key=access_logs.s3.prefix,Value=prod

```

<a id="cloudwatch-metrics"></a>
### 18.2 Critical CloudWatch Metrics Monitoring Core

| CloudWatch Metric Name | AWS Metric Namespace | Metric Primary Core Use Case |
| --- | --- | --- |
| `RequestCount` | `AWS/ApplicationELB` | Tracks request volume through the ALB. |
| `TargetResponseTime` | `AWS/ApplicationELB` | Measures application latency profiles at the P95/P99 scale. |
| `HealthyHostCount` | `AWS/ApplicationELB` | Monitors the number of healthy backend nodes available per target group. |
| `UnHealthyHostCount` | `AWS/ApplicationELB` | Detects target loss before customer-facing failures escalate. |
| `HTTPCode_ELB_4XX_Count` | `AWS/ApplicationELB` | Tracks client-side errors generated at the load balancer layer. |
| `HTTPCode_ELB_5XX_Count` | `AWS/ApplicationELB` | Tracks load-balancer-side failures, often caused by no healthy targets or connection errors. |
| `HTTPCode_Target_5XX_Count` | `AWS/ApplicationELB` | Tracks server errors thrown by application targets. |
| `ActiveConnectionCount` | `AWS/NetworkELB` | Measures concurrent open socket metrics on Layer 4 network balancers. |
| `ProcessedBytes` | `AWS/NetworkELB` | Tracks raw network throughput and bandwidth consumption. |

### 18.3 Auto Scaling Metrics & Dashboards

| ASG Signal | Why It Matters |
| --- | --- |
| **CPUUtilization** | Common target tracking metric for CPU-bound services. |
| **NetworkIn / NetworkOut** | Better signal for network-heavy services where CPU stays low. |
| **GroupDesiredCapacity** | Shows what the ASG is trying to maintain. |
| **GroupInServiceInstances** | Shows usable capacity currently serving traffic. |
| **GroupPendingInstances** | Reveals slow launches or capacity shortages. |
| **Scaling Activities** | Explains why the group launched or terminated instances. |

A production dashboard should place traffic, latency, error rate, healthy hosts, desired capacity, in-service capacity, and scaling activities on the same screen. This lets operators correlate "traffic rose" with "ASG scaled out" and "latency recovered."

### 18.4 Logging Strategy

* **ELB access logs:** Store request-level records in S3 for audit, incident response, and traffic analysis.
* **CloudWatch Logs:** Store application logs, bootstrap logs, and deployment logs from instances or containers.
* **Correlation IDs:** Propagate request IDs through the load balancer and application tiers so a single customer request can be traced.
* **Retention policy:** Apply lifecycle policies to S3 logs and CloudWatch log groups to avoid uncontrolled storage cost.

---

## 💸 33-36. ALB Performance & Storage Lifecycle Tiering

<a id="lcu-nlcu-billing"></a>
### 10.1 Performance Operational Modes

Capacity allocation on Application Load Balancers is scaled automatically based on demand and metered programmatically through **Load Balancer Capacity Units (LCU)**. An LCU measures balancer utilization across four structural vectors (billing is calculated against whichever dimension reaches the highest threshold peak):

* **New Connections:** Number of newly initialized connections per second (up to 25 per LCU).
* **Active Connections:** Concurrent open connections tracked per minute (up to 3,000 per LCU).
* **Processed Bytes:** Volumetric throughput data processed per hour (up to 1 GB per LCU for ALB).
* **Rule Evaluations:** The product of the number of active rules multiplied by the request rate (up to 1,000 evaluations per second per LCU).

For NLB, **Network Load Balancer Capacity Units (NLCU)** measure new connections or flows, active connections or flows, and processed bytes. The most expensive dimension is usually high throughput, high connection churn, or cross-AZ traffic patterns.

| Cost Lever | Optimization Strategy |
| --- | --- |
| **LCU / NLCU usage** | Reduce unnecessary listener rules, avoid connection churn, and right-size request routing patterns. |
| **Cross-AZ transfer** | Keep target counts balanced across AZs and understand NLB cross-zone data transfer impact. |
| **Idle load balancers** | Delete unused load balancers, stale listeners, and abandoned target groups after migrations. |
| **Spot instances** | Use mixed fleets for stateless and fault-tolerant workloads. |
| **Rightsizing** | Match instance family and target tracking metrics to the workload bottleneck. |

> ⚠️ **Exam Tip:** ALB billing uses LCU dimensions; NLB billing uses NLCU dimensions. Cross-AZ transfer costs are especially important in NLB architectures.

### 10.2 Throughput Allocation Strategies

* **Pre-Warming Support:** For flash-sale configurations, broadcast media events, or massive coordinated load testing, open an AWS Support case to manually "pre-warm" the ingress load balancer array to absorb massive incoming traffic spikes without initial packet drop.
* **Target Group Slow-Start Protection:** Safeguards fragile backend nodes from immediate cascade failures by ramping up connection limits incrementally over a predefined window.

### 10.3 Automated Target Group Lifecycle Tiering

* **Sticky Sessions (Session Affinity):** Forces a client's sequential requests to route to an identical backend instance via cookies. Use *duration-based cookies* generated by the ELB engine (`AWSALB`) or *application-based cookies* customized directly by backend application microservices.
* **Deregistration Delay Tuning:** Automatically matches your application stack's real-world execution timeout behaviors. If processing complex reports takes 4 minutes, ensure the deregistration delay is sustained at $\ge 240$ seconds to keep connections active during rolling deployments.

![Sticky Sessions](<assets/Sticky Sessions.png>)

---

<a id="part-viii-decision-making-exams"></a>
# 🎯 Part VIII — Decision Making & Exams

<a id="selection-framework"></a>
## 🎯 37. Load Balancer Selection Decision Framework

### 37.1 Service Selection Cheat Sheet

| Requirement | Best-Fit Service | Why |
| --- | --- | --- |
| Route by path, host, header, or query string | **ALB** | It understands HTTP at Layer 7. |
| Fixed IP address or Elastic IP | **NLB** | It provides static IPs per AZ. |
| Third-party firewall or IDS/IPS fleet | **GWLB** | It transparently distributes packets to appliances. |
| DNS-level regional failover | **Route 53** | It answers DNS queries based on routing policy and health checks. |
| Anycast static IPs and global traffic acceleration | **Global Accelerator** | It sends users to regional endpoints over the AWS global network. |
| CDN caching and edge delivery | **CloudFront** | It caches and serves content from edge locations. |
| Layer 7 attack filtering | **AWS WAF on ALB or CloudFront** | It evaluates HTTP request patterns before backend services. |

```text
                        [ Select Load Balancer Base ]
                                     │
          ┌──────────────────────────┴──────────────────────────┐
          ▼                                                     ▼
 [ HTTP/HTTPS Routing? ]                             [ TCP/UDP / Extreme Perf? ]
  Are you routing by URL/host?                        Need static IP?
          │                                                     │
  ┌───────┴───────┐                                     ┌───────┴───────┐
  ▼ YES           ▼ NO                                  ▼ YES           ▼ NO
Use ALB      Check NLB/GWLB                            Use NLB      Evaluate ALB
             3rd-party security appliance? ──► Use GWLB

```

---

<a id="certification-traps"></a>
## 🚨 38. AWS Certification Exam Traps & Scenario Analysis

### 🔴 Trap 1: ALB vs. NLB — Which Layer?

* **Question Pattern:** Your system requires routing architecture derived from evaluated HTTP cookies, URL path variables, and custom headers.
* **Wrong Answer:** Choose Network Load Balancer due to its high-performance throughput metrics.
* **Correct Answer:** Use an **Application Load Balancer (ALB)**.
* **Reasoning:** NLBs operate strictly on Layer 4 transport protocols and cannot inspect application layer payload structures like URLs or cookies.

### 🔴 Trap 2: Static IP Firewall Whitelisting

* **Question Pattern:** An external B2B partner's security firewall requires a fixed, single whitelistable IP address to transmit secure payroll data.
* **Wrong Answer:** Provision an ALB and point the client to its dynamic DNS endpoint name.
* **Correct Answer:** Deploy a **Network Load Balancer (NLB)** and assign a static Elastic IP to its interface.
* **Reasoning:** ALBs dynamically scale their underlying IP architecture, resulting in changing ingress IP addresses. Only NLBs offer fixed, permanent IP address allocation.

### 🔴 Trap 3: Third-Party Security Appliance Insertion

* **Question Pattern:** Internal compliance mandates that all ingress and egress VPC traffic must pass through a third-party virtual firewall appliance for deep packet inspection.
* **Wrong Answer:** Place an ALB in front of the application layer and attach AWS WAF rules.
* **Correct Answer:** Implement a **Gateway Load Balancer (GWLB)** to transparently route packets through a fleet of security appliances.
* **Reasoning:** GWLBs act as a transparent "bump-in-the-wire" network gateway, routing raw IP packets through virtual security appliances over the GENEVE protocol.

### 🔴 Trap 4: Multi-Domain SSL Hosting Costs

* **Question Pattern:** A cost-conscious architecture needs to serve secure traffic across three separate web domains (`app.a.com`, `app.b.com`, `app.c.com`) using a single load balancer interface.
* **Wrong Answer:** Spin up a legacy Classic Load Balancer and upload all three SSL keys.
* **Correct Answer:** Provision an **Application Load Balancer (ALB)** and leverage Server Name Indication (SNI) to load all three certificates onto a single HTTPS listener.
* **Reasoning:** Legacy CLBs support only one SSL certificate per listener, which would require deploying three separate load balancers. ALBs support SNI out of the box, allowing multiple certs on a single balancer.

### 🔴 Trap 5: User Logouts During Scaling Events

* **Question Pattern:** Users report being randomly logged out of their active sessions when the system scales down during low-traffic windows.
* **Wrong Answer:** Increase the minimum capacity limit on your Auto Scaling Group configuration.
* **Correct Answer:** Enable **Sticky Sessions** on the ALB target group, or externalize user sessions to Amazon ElastiCache or DynamoDB.
* **Reasoning:** When the ASG scales in, instances are terminated along with any local, in-memory user sessions. Externalizing session state ensures stateless, high-availability horizontal scaling.

### 🔴 Trap 6: Connection Drops During Code Deployments

* **Question Pattern:** In-flight user API calls drop mid-request and return connection errors during rolling code updates or automated target terminations.
* **Wrong Answer:** Increase the polling frequency of your target group health check configuration.
* **Correct Answer:** Enable and optimize the **Deregistration Delay / Connection Draining** timeout parameter.
* **Reasoning:** Without a deregistration delay, the load balancer abruptly drops active connections when an instance is removed. Enabling it allows the balancer to drain traffic gracefully, letting active requests finish.

### 🔴 Trap 7: Unexpected Data Transfer Cost Spikes on NLB

* **Question Pattern:** An engineering team notices a significant spike in intra-VPC data transfer costs after deploying a high-throughput Network Load Balancer.
* **Wrong Answer:** Reconfigure the subnet layout to route traffic over the public internet gateway.
* **Correct Answer:** Symmetrically balance instance counts across all active subnets, or evaluate the cost impact of enabling **Cross-Zone Load Balancing** on the NLB.
* **Reasoning:** Cross-zone load balancing is disabled by default on NLBs. Enabling it allows cross-AZ traffic distribution, but it incurs data transfer charges across AZ boundaries.

### 🔴 Trap 8: Silent Application Failures

* **Question Pattern:** An application loop crashes, causing web nodes to return `HTTP 500` server errors. However, the instances pass basic EC2 status checks, so the ASG fails to replace them.
* **Wrong Answer:** Shorten the health check grace period configuration on the Auto Scaling Group.
* **Correct Answer:** Change the Auto Scaling Group's health check type configuration from **EC2** to **ELB**.
* **Reasoning:** The default EC2 health check only monitors hypervisor status and hardware availability. Selecting the ELB health check type allows the ASG to monitor actual application layer health endpoints (`/health`).

---

## 🧭 37.2 Final Load Balancer Selection Workflow

> Navigation shortcuts: ALB routing features → §9, Load balancer comparison → §6.1, NLB static IPs & PrivateLink → §6.3, Architectural summary → §11, NLB→ALB chaining → §7.

```text
                          [ What is your use case? ]
                                      │
          ┌───────────────────────────┼───────────────────────────┐
          ▼                           ▼                           ▼
  HTTP/HTTPS routing?          Need static IP?          3rd-party appliance?
  Path/host/query?             Millions RPS?            Firewall/IDS/IPS?
          │                           │                           │
          ▼                           ▼                           ▼
   Use ALB (v2)               Use NLB (v2)              Use GWLB (2020)
                                       │
                               Need L7 routing too?
                                       │
                               NLB → ALB chaining
                              (NLB for static IP +
                               ALB for HTTP routing)

```

### 37.3 Route 53, Global Accelerator & Multi-Region Patterns

* **Route 53 failover:** DNS returns a primary ALB while health checks pass, then returns a secondary region when the primary endpoint fails. This is simple and cost-effective, but DNS caching can delay full client cutover.
* **Route 53 latency routing:** DNS returns the region with the lowest measured latency for the requester. Use it for active-active regional designs where each region can serve traffic independently.
* **Global Accelerator:** Provides static anycast IPs and routes clients to healthy regional endpoints over the AWS global network. Use it when you need fixed global IPs, fast failover, or lower latency for TCP/UDP workloads.
* **Disaster recovery:** Active-passive designs reduce steady-state cost but require tested failover. Active-active designs improve recovery time but require data replication, conflict handling, and regional capacity planning.

> ⚠️ **Exam Tip:** Route 53 is DNS routing, Global Accelerator is network acceleration with static anycast IPs, CloudFront is content delivery, and ELB distributes traffic inside a region.

---

<a id="production-best-practices"></a>
## ✅ 39. Production Best Practices

* **Deprecate Classic Load Balancers:** Always choose current-generation options (ALB, NLB, GWLB) over legacy CLBs to leverage advanced routing capabilities, security policy structures, and improved pricing profiles.
* **Lock Down Host Security Groups:** Symmetrically lock down backend EC2 security profiles to strictly accept inbound traffic from your load balancer's security group identifier (`sg-xxxxxx`), blocking direct public internet access.
* **De-couple Session State:** Avoid relying solely on sticky sessions. Externalize session data to distributed, high-speed storage layers like Amazon ElastiCache (Redis) or Amazon DynamoDB to enable true stateless scaling.
* **Automate Certificate Lifecycle Management:** Utilize AWS Certificate Manager (ACM) to provision and automatically renew TLS certificates, minimizing the risk of outages from expired credentials.
* **Configure Target Group Slow-Starts:** Always activate target group slow-start configurations when handling JVM or JIT-compiled application architectures to protect fresh nodes from being overwhelmed by traffic before they warm up.
* **Pre-Bake Computing Golden AMIs:** Avoid running heavy software installations inside instance User Data scripts at boot time. Pre-bake all dependencies into a Golden AMI to minimize scale-out times during high-traffic events.
* **Default to Target Tracking Scaling Policies:** Leverage Target Tracking as your default scaling policy mode. It functions as a self-tuning mechanism that adjusts fleet size smoothly without requiring complex step scaling alarm thresholds.
* **Audit Application Endpoint Health Paths:** Regularly monitor your application health check paths (`/health`). Ensure the underlying health check script verifies downstream dependencies, such as database connectivity, before returning an `HTTP 200 OK` status.




