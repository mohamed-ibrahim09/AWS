# ☁️ AWS VPC & EC2 Networking Lab

<div align="center">

![AWS](https://img.shields.io/badge/AWS-VPC-orange?style=for-the-badge\&logo=amazonaws)
![Networking](https://img.shields.io/badge/Networking-Cloud%20Infrastructure-blue?style=for-the-badge)
![EC2](https://img.shields.io/badge/EC2-Web%20Server-success?style=for-the-badge)

### 🌐 Amazon VPC & EC2 Hands-On Networking Lab

*A comprehensive hands-on lab focused on designing secure cloud networking infrastructure using Amazon VPC, subnets, route tables, NAT Gateways, security groups, and EC2 web servers in AWS.*

</div>

---

# 📖 Overview

This documentation provides a detailed walkthrough of building a custom virtual network environment using Amazon Web Services services. The lab demonstrates how organizations architect secure and scalable cloud infrastructure by combining networking, routing, internet connectivity, and compute services.

The environment simulates a production-style architecture where:

* Public-facing resources are isolated in public subnets
* Internal resources remain protected in private subnets
* Internet access is controlled through gateways and routing tables
* EC2 instances are secured using Security Groups

This lab introduces the foundational concepts required for:

* Cloud Engineering
* DevOps Engineering
* Solutions Architecture
* Cloud Security

---

# 🎯 Lab Objectives

By completing this lab, you will successfully:

* **Create a Custom VPC**

  * Build an isolated virtual network environment in AWS.

* **Configure Public and Private Subnets**

  * Separate internet-facing resources from internal resources.

* **Implement Internet Connectivity**

  * Configure Internet Gateway and NAT Gateway routing.

* **Configure Route Tables**

  * Control how network traffic flows inside the VPC.

* **Create Security Groups**

  * Apply firewall rules to EC2 instances.

* **Launch an EC2 Web Server**

  * Deploy and configure a Linux-based web server instance.

* **Understand High Availability Concepts**

  * Build subnets across multiple Availability Zones.

---

# 🏢 Business Scenario

A company is migrating its on-premises infrastructure to the cloud using AWS. The organization requires a secure network architecture where:

* Public web servers can be accessed by customers through the internet.
* Internal backend systems remain isolated from direct public access.
* Private resources still maintain outbound internet connectivity for updates and package downloads.
* The infrastructure supports scalability and high availability across multiple Availability Zones.

To achieve this, the company designs a VPC architecture using:

* Public Subnets
* Private Subnets
* Internet Gateway
* NAT Gateway
* EC2 Web Servers
* Security Groups

![Lab Scenario](assets/lab%20scenario.png)

---

# 🛠️ AWS Services Utilized

| Service                 | Purpose                                                                   |
| :---------------------- | :------------------------------------------------------------------------ |
| **🌐 Amazon VPC**       | Creates isolated virtual cloud networks within AWS.                       |
| **💻 Amazon EC2**       | Launches virtual machine instances for hosting applications and services. |
| **🛡️ Security Groups** | Acts as virtual firewalls controlling inbound and outbound traffic.       |
| **🚪 Internet Gateway** | Enables internet connectivity for public resources.                       |
| **🔄 NAT Gateway**      | Allows private subnet resources to access the internet securely.          |
| **🗺️ Route Tables**    | Controls network routing inside the VPC.                                  |

---

# 🧠 Core Networking Concepts

---

## 🌐 Amazon VPC (Virtual Private Cloud)

A VPC is a logically isolated network inside AWS where cloud resources can securely operate.

The lab VPC configuration:

| Setting             | Value         |
| :------------------ | :------------ |
| **VPC Name**        | `lab-vpc`     |
| **IPv4 CIDR Block** | `10.0.0.0/16` |

The `/16` CIDR block provides a large IP address range that can later be divided into multiple subnets.

---

## 🧩 Subnets

Subnets divide the VPC into smaller network segments.

### 🌍 Public Subnets

Public subnets are connected to the internet through an Internet Gateway.

| Subnet                          | CIDR          |
| :------------------------------ | :------------ |
| `lab-subnet-public1-us-east-1a` | `10.0.0.0/24` |
| `lab-subnet-public2`            | `10.0.2.0/24` |

Resources inside these subnets can receive internet traffic.

![Creating VPC in AZ_A](assets/Creating%20VPC%20in%20AZ_A.png)

---

### 🔒 Private Subnets

Private subnets do not allow direct inbound internet access.

| Subnet                           | CIDR          |
| :------------------------------- | :------------ |
| `lab-subnet-private1-us-east-1a` | `10.0.1.0/24` |
| `lab-subnet-private2`            | `10.0.3.0/24` |

Private resources remain protected while still being able to access the internet through a NAT Gateway.

![Editing Subnets](assets/Editing%20Subnets.png)

---

# 🌍 Availability Zones & High Availability

The lab architecture spans two Availability Zones:

* `us-east-1a`
* `us-east-1b`

Using multiple Availability Zones improves:

* Fault tolerance
* Redundancy

![Subnet associations](assets/Subnet%20associations.png)
* Service availability

This is a critical principle in production cloud architectures.

---

# 🚪 Internet Gateway (IGW)

An Internet Gateway enables communication between resources inside the VPC and the public internet.

The public route table contains:

```text id="o4qx84"
0.0.0.0/0 → Internet Gateway
```

This means all internet-bound traffic from public subnets is routed directly to the internet.

---

# 🔄 NAT Gateway

A NAT Gateway allows resources in private subnets to:

* Download updates
* Install packages
* Access external APIs

without exposing those resources directly to the internet.

The private route table contains:

```text id="9e2l7s"
0.0.0.0/0 → NAT Gateway
```

This ensures outbound internet access while maintaining inbound isolation.

---

# 🗺️ Route Tables

Route Tables determine how traffic flows within the VPC.

---

## 🌍 Public Route Table

| Destination | Target           |
| :---------- | :--------------- |
| `0.0.0.0/0` | Internet Gateway |

Associated with:

* Public Subnet 1
* Public Subnet 2

---

## 🔒 Private Route Table

| Destination | Target      |
| :---------- | :---------- |
| `0.0.0.0/0` | NAT Gateway |

Associated with:

* Private Subnet 1
* Private Subnet 2

---

# 🛡️ Security Group Configuration

A Security Group acts as a virtual firewall for EC2 instances.

The lab created the following security group:

| Setting                 | Value                |
| :---------------------- | :------------------- |
| **Security Group Name** | `Web Security Group` |
| **Description**         | Enable HTTP access   |

---

## 📥 Inbound Rule

| Type | Source        | Description         |
| :--- | :------------ | :------------------ |
| HTTP | Anywhere-IPv4 | Permit web requests |

This allows external users to access the hosted web application through HTTP (Port 80).

---

# 💻 EC2 Web Server Deployment

An EC2 instance was launched to host a web application.

| Configuration     | Value                |
| :---------------- | :------------------- |
| **Instance Name** | `Web Server 1`       |
| **AMI**           | Amazon Linux 2023    |
| **Instance Type** | `t2.micro`           |
| **Subnet**        | `lab-subnet-public2` |
| **Public IP**     | Enabled              |

---

# 🔑 Key Pair Configuration

The instance uses the following key pair:

```text id="mjlwmq"
vockey
```

This key pair enables secure SSH access to the EC2 instance.

---

# ⚙️ User Data Automation Script

During launch, the EC2 instance executes a bootstrap script automatically.

The script performs the following operations:

* Installs Apache Web Server
* Installs PHP
* Downloads the lab web application
* Extracts website files
* Starts the Apache service

---

## 🧾 User Data Script

```bash
#!/bin/bash

# Install Apache Web Server and PHP
dnf install -y httpd wget php mariadb105-server

# Download Lab Files
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-ACCLFO-2/2-lab2-vpc/s3/lab-app.zip

# Extract Website Files
unzip lab-app.zip -d /var/www/html/

# Enable Apache Service
chkconfig httpd on

# Start Apache
service httpd start
```

---

# 🌐 Final Architecture Overview

The final infrastructure consists of:

* One VPC
* Two Public Subnets
* Two Private Subnets
* One Internet Gateway
* One NAT Gateway
* Public and Private Route Tables
* One Security Group
* One EC2 Web Server

The deployed web server becomes publicly accessible through its Public IPv4 DNS address.

---

# 🧪 Lab Validation

The infrastructure was validated successfully by:

* Launching the EC2 instance
* Passing EC2 status checks
* Accessing the hosted web application through the browser
* Verifying internet routing functionality
* Confirming Security Group access permissions

---

# 🏆 Key Learning Outcomes

This lab demonstrates several foundational AWS networking and cloud architecture principles:

* Designing isolated cloud networks
* Separating public and private resources
* Implementing secure internet connectivity
* Understanding routing and subnet associations
* Configuring firewall rules using Security Groups
* Automating EC2 provisioning using User Data scripts
* Building highly available cloud infrastructure

---

# 📚 Conclusion

This lab provides practical experience with the networking foundations of AWS cloud infrastructure. Understanding VPCs, subnets, routing, gateways, and EC2 deployment is essential for modern cloud engineering roles.

The architecture implemented in this exercise reflects real-world enterprise cloud environments where security, scalability, and availability are critical requirements.

Mastering these concepts forms the backbone of:

* Cloud Engineering
* DevOps
* Solutions Architecture
* Cloud Security Engineering

---

<div align="center">

### ☁️ Built with AWS Cloud Infrastructure

**Amazon VPC • EC2 • Networking • Security • High Availability**

</div>
