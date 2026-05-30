# ☁️ AWS EC2 Management & Monitoring Lab

<div align="center">

![AWS](https://img.shields.io/badge/AWS-EC2-orange?style=for-the-badge\&logo=amazonaws)
![Cloud](https://img.shields.io/badge/Cloud-Infrastructure-blue?style=for-the-badge)
![EC2](https://img.shields.io/badge/EC2-Web%20Server-success?style=for-the-badge)
![Monitoring](https://img.shields.io/badge/Monitoring-CloudWatch-yellow?style=for-the-badge)

### 💻 Amazon EC2 Hands-On Management & Monitoring Lab

*A comprehensive hands-on lab focused on launching, securing, monitoring, resizing, and protecting Amazon EC2 instances using AWS cloud infrastructure services.*

</div>

---

# 📖 Overview

This lab provides practical experience with managing virtual servers using Amazon Web Services and specifically Amazon EC2.

The lab simulates a real-world cloud deployment workflow where a system administrator or cloud engineer deploys a Linux-based web server, configures network security, enables protection mechanisms, monitors server health, scales resources, and explores AWS service quotas.

The environment demonstrates how organizations operate production-ready cloud infrastructure using AWS services and best practices.

![](assets/Lab%20overview%20and%20objectives.png)

This lab introduces essential cloud concepts required for:

* Cloud Engineering
* DevOps Engineering
* System Administration
* Solutions Architecture
* Cloud Security

---

# 🎯 Lab Objectives

By completing this lab, you will successfully:

* **Launch an Amazon EC2 Instance**

  * Deploy a Linux-based virtual server in AWS.

* **Configure Security Groups**

  * Control inbound network access using firewall rules.

* **Deploy a Web Server Automatically**

  * Use User Data scripts for automated provisioning.

* **Monitor EC2 Health & Performance**

  * Explore instance monitoring and CloudWatch metrics.

* **Enable Termination & Stop Protection**

  * Prevent accidental shutdown or deletion of production resources.

* **Resize an EC2 Instance**

  * Scale compute resources by changing instance types.

* **Resize an EBS Volume**

  * Increase storage capacity dynamically.

* **Understand EC2 Service Quotas**

  * Explore AWS regional resource limits.

---

# 🏢 Business Scenario

A company is migrating its internal web applications to the cloud using AWS infrastructure. The organization requires a secure and scalable environment capable of hosting publicly accessible applications while maintaining operational safety and monitoring capabilities.

The company needs to:

* Launch Linux web servers rapidly
* Restrict network access using firewall rules
* Automatically configure servers during deployment
* Prevent accidental deletion or shutdown of critical systems
* Scale infrastructure as demand increases
* Monitor system health and availability
* Understand AWS account resource limitations

To achieve this, the organization uses:

* Amazon EC2
* Amazon EBS
* Security Groups
* CloudWatch Monitoring
* Service Quotas
* User Data Automation

---

# 🛠️ AWS Services Utilized

| Service                            | Purpose                                                      |
| :--------------------------------- | :----------------------------------------------------------- |
| **💻 Amazon EC2**                  | Launches and manages virtual machine instances.              |
| **💾 Amazon EBS**                  | Provides persistent block storage volumes for EC2 instances. |
| **🛡️ Security Groups**            | Controls inbound and outbound traffic using firewall rules.  |
| **📊 Amazon CloudWatch**           | Monitors instance performance and metrics.                   |
| **📦 Amazon Machine Images (AMI)** | Provides operating system templates for EC2 deployment.      |
| **📈 Service Quotas**              | Displays AWS account resource limits and quotas.             |

---

# 💻 EC2 Instance Deployment

An EC2 instance was launched to host a Linux-based web server.

![](assets/Configuring%20the%20instance.png)

| Configuration     | Value             |
| :---------------- | :---------------- |
| **Instance Name** | `Web Server`      |
| **AMI**           | Amazon Linux 2023 |
| **Instance Type** | `t2.micro`        |
| **Key Pair**      | `vockey`          |
| **Storage**       | 8 GiB EBS Volume  |
| **Public IP**     | Enabled           |

![](assets/Instance%20after%20running.png)

# 🧠 Core EC2 Concepts

---

## 💻 Amazon EC2

Amazon EC2 is a cloud computing service that allows users to launch virtual servers on demand.

EC2 instances provide:

* Virtual CPUs
* Memory
* Storage
* Networking
* Operating systems

without requiring physical hardware management.

---

## 📦 Amazon Machine Image (AMI)

The lab uses:

```text id="mf8i3o"
Amazon Linux 2023
```

An AMI acts as a deployment template containing:

* Operating system
* Boot configuration
* Required software packages

This allows rapid and consistent server provisioning.

---

## ⚡ Instance Type

The deployed instance type:

```text id="g9p0qe"
t2.micro
```

Provides:

| Resource | Value |
| :------- | :---- |
| vCPU     | 1     |
| Memory   | 1 GiB |

The lab later upgrades the instance to:

```text id="r2wo6y"
t2.small
```

which provides additional memory and improved performance.

---

# 🔑 Key Pair Configuration

The instance uses the following key pair:

```text id="jlwmk9"
vockey
```

Key pairs provide secure SSH authentication to EC2 instances using public-key cryptography.

---

# 🌐 Networking & Security

---

## 🛡️ Security Group Configuration

A Security Group acts as a virtual firewall controlling traffic entering and leaving the EC2 instance.

The lab creates:

| Setting                 | Value                            |
| :---------------------- | :------------------------------- |
| **Security Group Name** | `Web Server security group`      |
| **Description**         | Security group for my web server |

---

## 📥 HTTP Access Rule

Initially, the security group contained no inbound rules, resulting in failed public access.

![](assets/Faild%20access%20due%20to%20SG%20inbound%20rules.png)

To allow web traffic, the following rule was added to open port 80:

| Type | Source        |
| :--- | :------------ |
| HTTP | Anywhere-IPv4 |

This opens:

```text id="h3rx7z"
Port 80
```

for inbound HTTP requests.

---

# ⚙️ User Data Automation Script

During launch, the EC2 instance automatically executes a bootstrap script using User Data.

![](assets/script%20in%20user%20data.png)

The script performs:

* Apache installation
* Service startup
* Automatic boot configuration
* Website deployment

---

## 🧾 User Data Script

```bash
#!/bin/bash

# Install Apache Web Server
dnf install -y httpd

# Enable Apache on Boot
systemctl enable httpd

# Start Apache Service
systemctl start httpd

# Create Simple Web Page
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
```

---

# 🌐 Web Server Validation

After modifying the Security Group, the hosted application became publicly accessible through the EC2 Public IPv4 Address.

The browser successfully displayed the default web page:

```text id="jlwmh8"
Hello From Your Web Server!
```

![](assets/Result%20from%20public%20IP.png)

This confirmed:

* Apache installation succeeded
* EC2 networking functioned correctly
* Security Group rules permitted HTTP traffic

---

# 📊 EC2 Monitoring & Troubleshooting

The lab explored monitoring capabilities using Amazon CloudWatch.

---

## ✅ Status Checks

AWS automatically performs two health checks:

| Check                 | Purpose                                  |
| :-------------------- | :--------------------------------------- |
| System Reachability   | Verifies AWS infrastructure health       |
| Instance Reachability | Verifies operating system responsiveness |

The instance successfully passed:

```text id="jlwmw4"
2/2 checks passed
```

---

## 📈 CloudWatch Metrics

The Monitoring tab provides performance metrics such as:

* CPU utilization
* Network traffic
* Disk operations
* Status checks

CloudWatch enables operational visibility into running infrastructure.

---

## 📜 System Logs

The lab accessed the EC2 System Log to review boot-time console output.

![](assets/Accessing%20System%20logs.png)

The logs confirmed successful execution of the User Data script and Apache installation process.

---

## 🖥️ Instance Screenshot

The lab also demonstrated the EC2 Screenshot feature, which allows administrators to capture the virtual console display for troubleshooting inaccessible systems.

![](assets/Screenshot%20of%20the%20instance.png)

---

# 🛡️ EC2 Protection Mechanisms

---

## ❌ Termination Protection

Termination Protection was enabled during deployment.

This prevents accidental deletion of the EC2 instance.

Even if a user attempts to terminate the instance:

```text id="jlwm2r"
Terminate Instance
```

AWS blocks the action until protection is disabled.

---

## ⛔ Stop Protection

The lab later enabled Stop Protection:

```text id="jlwm5q"
Stop Protection
```

![](assets/Enabling%20Stop%20protection.png)

This prevents accidental shutdown of production systems.

If a user attempts to stop the instance while protection is enabled, AWS rejects the request.

![](assets/Failed%20Stop%20due%20to%20Stop%20Protection.png)

Once Stop Protection is disabled, the instance can be stopped successfully.

![](assets/Instance%20stopped%20after%20closing%20stop%20protection.png)

---

# 💾 Amazon EBS Storage

The EC2 instance uses:

Amazon Elastic Block Store

as persistent storage.

---

## 📦 Initial Storage Configuration

| Setting      | Value |
| :----------- | :---- |
| Volume Type  | EBS   |
| Initial Size | 8 GiB |

---

## 📈 Volume Resizing

The lab increased storage capacity from:

```text id="jlwm7a"
8 GiB → 10 GiB
```

![](assets/Modifing%20Volume.png)

This demonstrates how AWS allows dynamic scaling of storage resources without recreating servers.

---

# ⚡ EC2 Scaling

To simulate infrastructure scaling, the EC2 instance type was upgraded from:

```text id="jlwm4p"
t2.micro
```

to:

```text id="jlwm9v"
t2.small
```

![](assets/Channging%20the%20type%20to%20t2.small.png)

This increased the available memory and compute capacity.

---

# 📚 Understanding Stop vs Terminate

| Action        | Result                                                             |
| :------------ | :----------------------------------------------------------------- |
| **Stop**      | Shuts down the instance while preserving storage and configuration |
| **Start**     | Powers the instance back on                                        |
| **Terminate** | Permanently deletes the instance and associated resources          |

---

# 📈 EC2 Service Quotas

The lab explored Service Quotas to understand AWS resource limits.

![](assets/Service%20quotes.png)

AWS applies limits per region for resources such as:

* Running EC2 instances
* vCPUs
* EBS volumes
* Elastic IP addresses

These limits help:

* Prevent accidental overconsumption
* Improve platform stability
* Protect accounts from abuse

---

# 🧪 Lab Validation

The lab environment was successfully validated through:

* Launching the EC2 instance
* Passing EC2 status checks
* Installing and running Apache Web Server
* Accessing the hosted web page
* Modifying Security Groups
* Resizing EC2 resources
* Testing Stop Protection
* Exploring AWS Service Quotas

![](assets/Grades%20After%20lab.png)

---

# 🏆 Key Learning Outcomes

This lab demonstrates several foundational AWS cloud infrastructure concepts:

* Launching and managing EC2 instances
* Configuring Linux-based web servers
* Understanding Security Groups and firewall rules
* Monitoring infrastructure using CloudWatch
* Protecting production resources from accidental actions
* Scaling compute and storage resources dynamically
* Understanding persistent storage with EBS
* Exploring AWS regional service limits

---

# 📚 Conclusion

This lab provides hands-on experience with core AWS infrastructure management using EC2, EBS, CloudWatch, and Security Groups.

The deployment workflow closely resembles real-world cloud operations where engineers must securely deploy, monitor, protect, and scale production infrastructure.

Understanding these concepts is essential for careers in:

* Cloud Engineering
* DevOps Engineering
* Solutions Architecture
* Site Reliability Engineering (SRE)
* Cloud Security

Mastering EC2 management forms the foundation for operating scalable and resilient cloud-native applications on AWS.

---

<div align="center">

### ☁️ Built with AWS Cloud Infrastructure

**Amazon EC2 • EBS • CloudWatch • Security Groups • Monitoring • Scaling**

</div>
