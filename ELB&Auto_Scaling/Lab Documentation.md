# ☁️ AWS Elastic Load Balancing & Auto Scaling Lab

<div align="center">

![AWS](https://img.shields.io/badge/AWS-ELB-orange?style=for-the-badge&logo=amazonaws)
![AutoScaling](https://img.shields.io/badge/Auto%20Scaling-Dynamic%20Capacity-blue?style=for-the-badge)
![CloudWatch](https://img.shields.io/badge/CloudWatch-Monitoring-success?style=for-the-badge)
![EC2](https://img.shields.io/badge/EC2-High%20Availability-yellow?style=for-the-badge)

### ⚖️ Amazon ELB & Auto Scaling Hands-On High Availability Lab

*A comprehensive hands-on lab focused on Elastic Load Balancing, EC2 Auto Scaling, AMI creation, CloudWatch alarms, and dynamic infrastructure scaling using AWS cloud infrastructure.*

</div>

---

# 📖 Overview

This lab focuses on building a highly available and automatically scalable web application infrastructure using Amazon Elastic Load Balancing and EC2 Auto Scaling.

The lab demonstrates how cloud engineers distribute incoming traffic across multiple EC2 instances, automatically scale capacity in response to real-time demand, and monitor infrastructure health using Amazon CloudWatch.

The environment simulates a real-world production scenario where organizations require resilient application infrastructure capable of:

* Distributing traffic evenly across multiple instances and Availability Zones
* Scaling compute capacity automatically based on CPU utilization
* Maintaining application availability during demand spikes
* Reducing costs by removing excess capacity during low-traffic periods
* Recovering automatically from instance failures without manual intervention

This lab introduces foundational scalability concepts required for:

* Cloud Engineering
* DevOps Engineering
* Site Reliability Engineering (SRE)
* Solutions Architecture
* Performance Engineering

---

# 🎯 Lab Objectives

By completing this lab, you will successfully:

* **Create an Amazon Machine Image (AMI)**

  * Capture a running EC2 instance as a reusable golden image for Auto Scaling.

* **Create an Application Load Balancer**

  * Distribute incoming HTTP traffic across multiple EC2 instances and Availability Zones.

* **Create a Launch Template**

  * Define the instance configuration used by the Auto Scaling group.

* **Create an Auto Scaling Group**

  * Automatically launch and terminate EC2 instances based on demand.

* **Test Auto Scaling Under Load**

  * Generate CPU load and observe automatic instance provisioning.

* **Monitor with Amazon CloudWatch**

  * Create and observe alarms tied to scaling policies.

---

# 🏢 Business Scenario

**NovaMart** is a fast-growing Egyptian e-commerce company that sells electronics across Egypt and the wider MENA region. Their platform experiences highly unpredictable traffic — quiet during weekday mornings and severely overloaded during promotional events, flash sales, and national holidays.

Their current infrastructure runs all web traffic through a single EC2 instance. This has resulted in serious business problems:

* During a major Ramadan sale, the single web server became completely unresponsive under load — causing an estimated **EGP 2.3 million in lost sales** in under 3 hours
* On quiet days, the same oversized instance runs at 4% CPU utilization — wasting money around the clock
* When the instance fails, the entire platform goes offline until an engineer manually relaunches it
* There is no automated mechanism to detect unhealthy instances and remove them from serving traffic
* Scaling currently requires a manual ticket to the infrastructure team with a 2-hour response SLA

The engineering team has been tasked with redesigning the infrastructure using AWS native services:

* Deploy an **Application Load Balancer** to distribute traffic across multiple instances
* Use **Auto Scaling** to launch new instances when CPU exceeds 60% and terminate them when load drops
* Build a **CloudWatch alarm-driven scaling policy** with a minimum of 2 and maximum of 6 instances
* Ensure the platform stays online automatically even if individual instances fail
* Reduce idle infrastructure cost by scaling down automatically after traffic subsides

This lab simulates that real-world redesign. You will build the entire scalable architecture from a single running instance — exactly as a cloud engineer would in a production migration.

---

# 🛠️ AWS Services Utilized

| Service | Purpose |
| :--- | :--- |
| **💻 Amazon EC2** | Hosts the web application instances behind the load balancer. |
| **🖼️ Amazon Machine Image (AMI)** | Captures the web server configuration as a reusable image for Auto Scaling. |
| **⚖️ Elastic Load Balancing (ALB)** | Distributes incoming HTTP traffic across healthy EC2 instances. |
| **📋 Launch Template** | Defines the instance type, AMI, security group, and settings for Auto Scaling. |
| **📈 EC2 Auto Scaling** | Automatically adds or removes instances based on CloudWatch metrics. |
| **📊 Amazon CloudWatch** | Monitors CPU utilization and triggers scaling alarms. |
| **🌐 Amazon VPC** | Provides network isolation using public and private subnets across two AZs. |

---

# 🧠 Core Concepts

---

## ⚖️ Elastic Load Balancing (ELB)

Elastic Load Balancing automatically distributes incoming application traffic across multiple EC2 instances, containers, and other targets.

The **Application Load Balancer (ALB)** operates at Layer 7 (HTTP/HTTPS) and supports:

* Content-based routing (path, host, header)
* Health checks on individual targets
* Cross-zone load balancing across Availability Zones
* Integration with Auto Scaling target groups

Key benefits for production workloads:

* No single point of failure at the traffic distribution layer
* Automatically stops sending traffic to unhealthy instances
* Scales its own capacity to handle any level of incoming traffic

---

## 📈 EC2 Auto Scaling

Auto Scaling monitors your applications and automatically adjusts capacity to maintain steady, predictable performance at the lowest possible cost.

Auto Scaling uses three core components:

| Component | Purpose |
| :--- | :--- |
| Launch Template | Defines what type of instance to launch (AMI, instance type, security group) |
| Auto Scaling Group | Defines how many instances to run and the scaling boundaries |
| Scaling Policy | Defines when to scale based on CloudWatch metrics |

---

## 📊 Target Tracking Scaling Policy

The lab uses a **Target Tracking** policy set to `60% Average CPU Utilization`.

| Condition | Action |
| :--- | :--- |
| CPU > 60% for 3 minutes | Auto Scaling adds instances (scale out) |
| CPU < 60% sustained | Auto Scaling removes instances (scale in) |
| Instance count < 2 | Never scale in below minimum |
| Instance count > 6 | Never scale out above maximum |

This maintains performance during spikes while preventing over-provisioning during quiet periods.

---

# 🏗️ Architecture Overview

### Starting State

```
┌──────────────────────────────────────┐
│              Internet                │
└──────────────┬───────────────────────┘
               │
               ▼
    ┌─────────────────────┐
    │    Web Server 1     │  ← Single point of failure
    │    (EC2 Instance)   │
    └─────────────────────┘
```

![Starting Architecture](assets/lab%20the%20starting%20architicture.png)

### Final State After Lab

```
┌──────────────────────────────────────────────────────────┐
│                        Internet                          │
└─────────────────────────┬────────────────────────────────┘
                          │
                          ▼
          ┌───────────────────────────────┐
          │   Application Load Balancer   │
          │         (LabELB)              │
          │   Public Subnet 1 & 2 (AZ)   │
          └──────────┬──────┬────────────┘
                     │      │
          ┌──────────▼──┐  ┌▼─────────────┐
          │ Lab Instance│  │ Lab Instance  │  ← Min 2, Max 6
          │ Private AZ-1│  │ Private AZ-2  │  ← Auto Scaling
          └─────────────┘  └──────────────┘
                     ↑ CloudWatch Alarm drives scaling ↑
```

![Final Architecture](assets/lab%20final%20architichure.png)

---

# 🖼️ Task 1 — Create an AMI for Auto Scaling

Before creating the Auto Scaling group, a reusable **Amazon Machine Image (AMI)** was created from the existing running web server. This captures the entire disk state — operating system, application code, configuration — so every new instance launched by Auto Scaling starts with an identical, pre-configured environment.

## Why Create an AMI?

Without an AMI, every new EC2 instance would start as a blank OS and require manual configuration. The AMI eliminates this by providing a **golden image** that Auto Scaling uses to launch fully configured instances automatically.

## AMI Configuration

Navigate to:

```text
AWS Console → EC2 → Instances → Select Web Server 1
→ Actions → Image and Templates → Create Image
```

The following settings were configured:

| Setting | Value |
| :--- | :--- |
| Image Name | `WebServerAMI` |
| Image Description | `Lab AMI for Web Server` |

After confirming, AWS generated an AMI ID that was used in the Launch Template created in Task 3.

> **Note:** The instance does not need to be stopped to create an AMI. AWS takes a snapshot of the root EBS volume while the instance continues running.

![Creating AMI](assets/lab%20creating%20AMI.png)

---

# ⚖️ Task 2 — Create a Load Balancer

An **Application Load Balancer** was created to receive all incoming HTTP traffic and distribute it across healthy EC2 instances registered in the target group.

## Step 1 — Create a Target Group

A Target Group defines the set of instances that will receive forwarded traffic from the load balancer.

Navigate to:

```text
AWS Console → EC2 → Target Groups → Create Target Group
```

| Setting | Value |
| :--- | :--- |
| Target Type | `Instances` |
| Target Group Name | `LabGroup` |
| VPC | `Lab VPC` |

No instances were registered at this stage — Auto Scaling will register new instances automatically when they are launched.

## Step 2 — Create the Application Load Balancer

Navigate to:

```text
AWS Console → EC2 → Load Balancers → Create Load Balancer → Application Load Balancer
```

| Setting | Value |
| :--- | :--- |
| Load Balancer Name | `LabELB` |
| VPC | `Lab VPC` |
| Subnets | `Public Subnet 1` (AZ-1) and `Public Subnet 2` (AZ-2) |
| Security Group | `Web Security Group` |
| Listener | HTTP on port `80` → Forward to `LabGroup` |

## Load Balancer Network Mapping

| Availability Zone | Subnet |
| :--- | :--- |
| First AZ | `Public Subnet 1` |
| Second AZ | `Public Subnet 2` |

Selecting both public subnets across two Availability Zones ensures the load balancer itself has no single point of failure — if one AZ experiences issues, the load balancer continues routing traffic through the other.

![Network Mapping in LB](assets/lab%20network%20mapping%20in%20LB%201.png)

> **Architecture Note:** The Load Balancer resides in **public subnets** to receive internet traffic, while the EC2 instances reside in **private subnets** — inaccessible directly from the internet. All traffic must pass through the ALB.

![Security Group in LB](assets/lab%20SG%20in%20LB%202.png)

The load balancer entered a **Provisioning** state. Task 3 proceeded in parallel.

---

# 📋 Task 3 — Create a Launch Template and Auto Scaling Group

## Step 1 — Create the Launch Template

A Launch Template defines the blueprint for every EC2 instance that Auto Scaling launches.

Navigate to:

```text
AWS Console → EC2 → Launch Templates → Create Launch Template
```

| Setting | Value |
| :--- | :--- |
| Launch Template Name | `LabConfig` |
| AMI | `WebServerAMI` (created in Task 1) |
| Instance Type | `t2.micro` |
| Key Pair | `vockey` |
| Security Group | `Web Security Group` |
| Detailed CloudWatch Monitoring | `Enabled` |

![Creating Launch Template 1](assets/lab%20creating%20luanch%20templete%201.png)
![Creating Launch Template 2](assets/lab%20creating%20luanch%20templete%202.png)

> **Why Enable Detailed Monitoring?** Standard CloudWatch metrics are reported every 5 minutes. Enabling detailed monitoring reduces this to **1-minute intervals**, allowing Auto Scaling to detect and react to load spikes significantly faster.

## Step 2 — Create the Auto Scaling Group

From the `LabConfig` launch template, the Auto Scaling group was created using the following configuration across 6 steps:

### Step 1 — Basic Settings

| Setting | Value |
| :--- | :--- |
| Auto Scaling Group Name | `Lab Auto Scaling Group` |
| Launch Template | `LabConfig` |

### Step 2 — Network

| Setting | Value |
| :--- | :--- |
| VPC | `Lab VPC` |
| Subnets | `Private Subnet 1` and `Private Subnet 2` |

Instances launch into **private subnets** — they are not directly reachable from the internet.

### Step 3 — Load Balancer Integration

| Setting | Value |
| :--- | :--- |
| Load Balancer | Attach to existing — `LabGroup` target group |
| CloudWatch Group Metrics | `Enabled` (1-minute collection intervals) |

### Step 4 — Group Size & Scaling Policy

| Setting | Value |
| :--- | :--- |
| Desired Capacity | `2` |
| Minimum Capacity | `2` |
| Maximum Capacity | `6` |
| Scaling Policy | `Target Tracking` |
| Policy Name | `LabScalingPolicy` |
| Metric | `Average CPU Utilization` |
| Target Value | `60%` |

![Creating ASG 1](assets/lab%20creating%20ASG%201.png)
![Creating ASG 2](assets/lab%20creating%20ASG%202.png)

### Step 5 — Instance Tags

Tags applied to the Auto Scaling group propagate automatically to every instance it launches:

| Key | Value |
| :--- | :--- |
| `Name` | `Lab Instance` |

After creation, the Auto Scaling group immediately began launching **2 instances** to satisfy the Desired Capacity of 2.

---

# ✅ Task 4 — Verify Load Balancing is Working

## Verify Auto Scaling Launched Instances

Navigate to:

```text
AWS Console → EC2 → Instances
```

Two new instances named `Lab Instance` appeared — both launched automatically by the Auto Scaling group.

![Added Instances from ASG](assets/lab%20added%20instances%20from%20ASG.png)

## Verify Target Group Health

Navigate to:

```text
AWS Console → EC2 → Target Groups → LabGroup → Targets tab
```

Both `Lab Instance` targets were listed. After a short wait, both transitioned to:

```text
Status: healthy
```

![Target Group Health Check](assets/lab%20target%20group%20health%20check.png)

A `healthy` status means the Application Load Balancer's health check successfully received an HTTP response from the instance. The load balancer will only forward traffic to healthy targets.

## Access the Application via Load Balancer

The DNS name was copied from:

```text
AWS Console → EC2 → Load Balancers → LabELB → Details → DNS Name
```

The DNS name follows this format:

```text
LabELB-1998580470.us-west-2.elb.amazonaws.com
```

Opening this DNS name in a browser displayed the running web application — confirming that:

* The load balancer received the HTTP request
* Forwarded it to one of the healthy EC2 instances
* The instance processed and returned the response

![Testing the Instances](assets/lab%20testing%20the%20instances.png)

---

# 🔥 Task 5 — Test Auto Scaling Under Load

## CloudWatch Alarms

Navigate to:

```text
AWS Console → CloudWatch → All Alarms
```

Two alarms were automatically created by the Auto Scaling group:

| Alarm | Condition | Action |
| :--- | :--- | :--- |
| `AlarmHigh` | Average CPU > 60% for 3 minutes | Scale out — add instances |
| `AlarmLow` | Average CPU < threshold | Scale in — remove instances |

Both alarms started in `OK` state with minimal CPU activity.

![CloudWatch Tracking](assets/lab%20cloud%20watch%20tracking%201.png)

## Generating Load

The web application includes a **Load Test** feature. Selecting **Load Test** beside the AWS logo triggered intensive computation across all instances in the Auto Scaling group simultaneously.

## Observing Scale-Out

Returning to CloudWatch, within approximately 5 minutes:

* `AlarmLow` remained `OK`
* `AlarmHigh` transitioned to `In alarm`

The CloudWatch chart showed CPU climbing above the 60% threshold line.

![CPU at 100%](assets/lab%20testing%20the%20cpu%20100%25.png)

Once `AlarmHigh` remained in alarm for 3 consecutive minutes, Auto Scaling triggered the `LabScalingPolicy` and launched additional instances.

Navigate to:

```text
AWS Console → EC2 → Instances
```

More than two `Lab Instance` instances were now running — automatically provisioned by Auto Scaling in response to real demand.

![Instances After Load](assets/lab%20creating%20instances%20after%20load.png)

## Scale-Out Summary

```
Load Test Started
      │
      ▼
CPU rises across all instances
      │
      ▼
AlarmHigh triggers (CPU > 60% for 3 min)
      │
      ▼
LabScalingPolicy executes → New instances launched
      │
      ▼
Load Balancer registers new instances as healthy
      │
      ▼
Traffic distributed across all running instances
```

![After Alarm Low is OK](assets/lab%20after%20alarm%20low%20is%20ok.png)

---

# 🗑️ Task 6 — Terminate Web Server 1

The original `Web Server 1` instance served its purpose — it was used to create `WebServerAMI`. It is no longer required because:

* All production traffic is now handled by Auto Scaling instances behind the load balancer
* The AMI already captured everything needed to replicate the web server configuration
* Leaving it running would incur unnecessary cost

Navigate to:

```text
AWS Console → EC2 → Instances → Select Web Server 1
→ Instance State → Terminate Instance → Confirm
```

`Web Server 1` was permanently terminated.

> **Important:** Auto Scaling instances are not affected by manually terminating `Web Server 1`. If an Auto Scaling instance is terminated, Auto Scaling automatically detects the reduction in capacity and launches a replacement instance to restore the Desired Count.

---

# 🧪 Lab Validation

The scalable infrastructure was successfully validated through:

* Creating an AMI from a running EC2 web server
* Creating a Target Group for load balancer routing
* Provisioning an Application Load Balancer across two public subnets
* Creating a Launch Template using the captured AMI
* Building an Auto Scaling group with min 2 and max 6 instances
* Attaching the Auto Scaling group to the load balancer target group
* Verifying both instances passed health checks
* Accessing the application through the load balancer DNS name
* Triggering CPU load to activate the `AlarmHigh` CloudWatch alarm
* Observing automatic instance scale-out in response to the alarm
* Terminating the original Web Server 1 after migration to Auto Scaling

![Lab Grades](assets/lab%20grades%20form%20the%20lab.png)

---

# 🏆 Key Learning Outcomes

This lab demonstrates several foundational AWS scalability and availability concepts:

* Understanding the difference between vertical and horizontal scaling
* Creating reusable golden AMIs for consistent instance launches
* Designing multi-AZ load balancing with Application Load Balancer
* Configuring launch templates for Auto Scaling group integration
* Implementing target tracking scaling policies based on CPU utilization
* Using CloudWatch alarms to drive automatic infrastructure changes
* Understanding health checks and their role in traffic routing
* Building infrastructure that self-heals after instance failures

---

# 📊 Auto Scaling Behavior Summary

| Scenario | Auto Scaling Response |
| :--- | :--- |
| Group created | Launches instances to match Desired Capacity (2) |
| CPU > 60% sustained | Launches new instances up to Maximum (6) |
| CPU drops below threshold | Terminates excess instances down to Minimum (2) |
| Instance fails health check | Terminates unhealthy instance, launches replacement |
| AZ outage | Launches replacement instances in healthy AZ |

---

# 📊 ELB vs. No Load Balancer

| Capability | Single Instance | Application Load Balancer + Auto Scaling |
| :--- | :--- | :--- |
| Traffic Distribution | None | Across all healthy instances |
| Instance Failure Handling | Full outage | Automatic rerouting to healthy instances |
| Demand Spikes | Overload and crash | Auto scale out to handle load |
| Low Traffic Periods | Idle waste | Auto scale in to reduce cost |
| Multi-AZ Support | None | Built-in cross-AZ routing |
| Health Monitoring | Manual | Automated health checks every interval |

---

# 📚 Conclusion

This lab provides practical experience building highly available, fault-tolerant, and cost-optimized infrastructure using Amazon ELB and EC2 Auto Scaling.

The workflow closely mirrors real-world enterprise cloud operations where engineers must:

* Eliminate single points of failure in web application infrastructure
* Build systems that respond dynamically to real-world demand
* Reduce manual scaling operations through automation
* Distribute traffic intelligently across healthy compute resources
* Monitor and react to infrastructure performance automatically

Mastering ELB and Auto Scaling concepts forms a critical foundation for careers in:

* Cloud Engineering
* DevOps Engineering
* Site Reliability Engineering (SRE)
* Solutions Architecture
* Platform Engineering
* Cloud Cost Optimization

---

<div align="center">

### ☁️ Built with AWS Cloud Infrastructure

**Elastic Load Balancing • EC2 Auto Scaling • CloudWatch • AMI • Launch Templates • High Availability • Dynamic Scaling**

</div>