# ☁️ AWS RDS MySQL & Multi-AZ Database Management Lab

<div align="center">

![AWS](https://img.shields.io/badge/AWS-RDS-orange?style=for-the-badge&logo=amazonaws)
![Database](https://img.shields.io/badge/Database-MySQL-blue?style=for-the-badge&logo=mysql)
![MultiAZ](https://img.shields.io/badge/Multi--AZ-High%20Availability-success?style=for-the-badge)
![EC2](https://img.shields.io/badge/EC2-Web%20Server-yellow?style=for-the-badge)

### 🗄️ Amazon RDS Hands-On Managed Database & High Availability Lab

*A comprehensive hands-on lab focused on Amazon Relational Database Service (RDS), MySQL database provisioning, VPC networking, Multi-AZ deployments, and web application database integration using AWS cloud infrastructure.*

</div>

---

# 📖 Overview

This lab focuses on managed relational database deployment using Amazon RDS and its integration with Amazon EC2 web servers inside a Virtual Private Cloud.

The lab demonstrates how cloud engineers provision, secure, configure, and connect a production-grade managed MySQL database to a Linux-based web application running on EC2.

The environment simulates a real-world production scenario where organizations require reliable cloud database systems capable of:

* Hosting relational data independently from application servers
* Protecting database access using security groups and VPC isolation
* Providing high availability through Multi-AZ automatic failover
* Connecting web applications to managed databases securely
* Supporting MySQL-based workloads without manual database administration

This lab introduces foundational database concepts required for:

* Cloud Engineering
* DevOps Engineering
* Backend Development
* Database Administration
* Solutions Architecture
* Disaster Recovery

---

# 🎯 Lab Objectives

By completing this lab, you will successfully:

* **Create a Security Group for RDS**

  * Restrict database access to authorized web server instances only.

* **Create a DB Subnet Group**

  * Define which subnets across multiple Availability Zones RDS can use.

* **Provision an Amazon RDS MySQL Instance**

  * Launch a fully managed Multi-AZ MySQL database with high availability.

* **Connect a Web Application to RDS**

  * Configure a running EC2 web application to use the RDS endpoint.

* **Interact with a Live Database**

  * Perform create, read, update, and delete operations through a web interface.

* **Understand Multi-AZ Replication**

  * Learn how AWS automatically replicates data to a standby instance in a second Availability Zone.

---

# 🏢 Business Scenario

**TechNova Solutions** is a mid-sized SaaS company that provides a cloud-based Customer Relationship Management (CRM) platform to enterprise clients across the Middle East and North Africa.

Their current infrastructure runs on a single EC2 instance with a self-managed MySQL database installed directly on the server. This architecture has caused several critical problems:

* A single database server failure took the entire platform offline for 6 hours
* Manual database patching caused unplanned downtime every quarter
* The DBA team spends 30% of their time on routine database maintenance tasks
* There is no automatic failover or disaster recovery mechanism in place
* Compliance audits flagged insufficient network isolation between application and database layers

The CTO has approved a migration to Amazon RDS to resolve these issues. The infrastructure team has been tasked with:

* Migrating the CRM database to Amazon RDS MySQL with Multi-AZ enabled
* Isolating the database layer inside a dedicated VPC with strict security group rules
* Ensuring the web application can connect to the new managed database seamlessly
* Eliminating manual database administration overhead
* Achieving an RTO (Recovery Time Objective) of under 2 minutes using automatic Multi-AZ failover

This lab simulates that real-world migration scenario. You will provision the database infrastructure, configure network security, and connect the application — exactly as a cloud engineer would in a production environment.

---

# 🛠️ AWS Services Utilized

| Service | Purpose |
| :--- | :--- |
| **🗄️ Amazon RDS** | Provides a fully managed MySQL relational database instance. |
| **💻 Amazon EC2** | Hosts the Linux-based web server running the CRM application. |
| **🔒 VPC Security Groups** | Controls inbound and outbound traffic to the RDS instance. |
| **🌐 Amazon VPC** | Isolates database and application layers in a private network. |
| **🗂️ DB Subnet Group** | Defines the subnets across AZs available to the RDS instance. |

---

# 🧠 Core Amazon RDS Concepts

---

## 🗄️ Amazon Relational Database Service (RDS)

Amazon Relational Database Service is a fully managed database service that makes it easy to set up, operate, and scale relational databases in the cloud.

RDS handles routine database administration tasks automatically:

* Hardware provisioning
* Database setup and patching
* Automated backups
* Software updates
* Monitoring and alerting

Unlike a self-managed database on EC2, RDS:

* Eliminates undifferentiated database management work
* Provides built-in high availability through Multi-AZ
* Supports automatic failover with minimal downtime
* Scales compute and storage independently
* Integrates natively with VPC for network isolation

---

# ⚡ Amazon RDS Features

| Feature | Description |
| :--- | :--- |
| Managed Service | AWS handles patching, backups, and maintenance automatically |
| Multi-AZ Deployment | Synchronous replication to a standby in a second Availability Zone |
| Automatic Failover | Promotes standby to primary within ~1–2 minutes during failure |
| VPC Integration | Database runs inside a private subnet with full network isolation |
| Security Groups | Controls exactly which resources can connect on which ports |
| Supported Engines | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Amazon Aurora |

---

# 🌐 Lab VPC Architecture

The lab environment is pre-configured with a VPC containing:

| Resource | Value |
| :--- | :--- |
| VPC | `Lab VPC` |
| Web Server Subnet | Public subnet — hosts the EC2 web server |
| DB Subnet (AZ-1) | Private subnet `10.0.1.0/24` in `us-east-1a` |
| DB Subnet (AZ-2) | Private subnet `10.0.3.0/24` in `us-east-1b` |
| Web Security Group | Already assigned to the EC2 web server instance |

The database layer is completely isolated in private subnets. Only the web server can reach the database — no public internet access is allowed to RDS.

![Infrastructure Before Edit](assets/infrastructure%20before%20edit.png)

![Infrastructure After Edit](assets/infrastructure%20after%20edit.png)

---

# 🔐 Task 1 — Create a Security Group for RDS

Before launching the database, a dedicated security group was created to enforce strict access control.

## Why a Separate Security Group?

Security groups act as virtual firewalls. By creating a dedicated `DB Security Group`, only traffic from the `Web Security Group` (the EC2 web server) is permitted to reach the database on MySQL port `3306`. All other traffic is denied by default.

This enforces the principle of least privilege at the network layer.

## Security Group Configuration

Navigate to:

```text
AWS Console → VPC → Security Groups → Create Security Group
```

The following settings were configured:

| Setting | Value |
| :--- | :--- |
| Security Group Name | `DB Security Group` |
| Description | `Permit access from Web Security Group` |
| VPC | `Lab VPC` |

## Inbound Rule Configuration

An inbound rule was added with the following settings:

| Field | Value |
| :--- | :--- |
| Type | `MySQL/Aurora (3306)` |
| Protocol | `TCP` |
| Port | `3306` |
| Source | `Web Security Group` |

This rule allows inbound MySQL traffic on port `3306` exclusively from any EC2 instance associated with the `Web Security Group`.

> **Security Note:** Setting the source to a Security Group ID rather than an IP address range means any new web server automatically inherits database access — a scalable and maintainable approach for production environments.

The security group was saved and will be attached to the RDS instance during launch.

![Creating Security Group](assets/creating%20SG.png)

---

# 🗂️ Task 2 — Create a DB Subnet Group

Amazon RDS requires a DB Subnet Group to know which subnets it can use when deploying database instances across Availability Zones.

## Why a DB Subnet Group?

A DB Subnet Group is a collection of VPC subnets spanning at least two Availability Zones. RDS uses this group to:

* Place the primary database instance in one subnet
* Place the Multi-AZ standby instance in a different AZ subnet
* Enable automatic failover between Availability Zones

## DB Subnet Group Configuration

Navigate to:

```text
AWS Console → RDS → Subnet Groups → Create DB Subnet Group
```

The following settings were configured:

| Setting | Value |
| :--- | :--- |
| Name | `DB-Subnet-Group` |
| Description | `DB Subnet Group` |
| VPC | `Lab VPC` |

## Subnet Selection

The following Availability Zones and subnets were selected:

| Availability Zone | Subnet CIDR |
| :--- | :--- |
| `us-east-1a` | `10.0.1.0/24` |
| `us-east-1b` | `10.0.3.0/24` |

Both subnets appeared in the **Subnets selected** table, confirming successful configuration.

![DB Subnet Group](assets/db%20subnet%20group.png)

> **Architecture Note:** Placing subnets in two separate AZs is a requirement for Multi-AZ RDS deployments. AWS will automatically select one AZ for the primary instance and the other for the standby replica.

---

# 🚀 Task 3 — Create an Amazon RDS DB Instance

With the security group and subnet group in place, the RDS MySQL database instance was provisioned.

## Multi-AZ Deployment Architecture

Multi-AZ RDS automatically creates:

* A **primary DB instance** that serves all read/write traffic
* A **standby DB instance** in a different AZ that receives synchronous data replication
* **Automatic failover** — if the primary fails, RDS promotes the standby within ~1–2 minutes with no manual intervention

This makes Multi-AZ the standard deployment model for production workloads.

## Database Instance Configuration

Navigate to:

```text
AWS Console → RDS → Databases → Create Database
```

The following configuration was applied:

### Engine & Template

| Setting | Value |
| :--- | :--- |
| Engine | `MySQL` |
| Template | `Dev/Test` |
| Availability | `Multi-AZ DB Instance` |

### Database Credentials

| Setting | Value |
| :--- | :--- |
| DB Instance Identifier | `lab-db` |
| Master Username | `main` |
| Master Password | `lab-password` |

### Instance Class & Storage

| Setting | Value |
| :--- | :--- |
| Instance Class | `db.t3.micro` (Burstable) |
| Storage Type | `General Purpose SSD (gp2)` |
| Allocated Storage | `20 GiB` |

### Connectivity

| Setting | Value |
| :--- | :--- |
| VPC | `Lab VPC` |
| Security Group | `DB Security Group` |
| Enhanced Monitoring | Disabled |

### Additional Configuration

| Setting | Value |
| :--- | :--- |
| Initial Database Name | `lab` |
| Automatic Backups | Disabled (for lab speed) |
| Encryption | Disabled (for lab speed) |

> **Production Note:** In a real environment, automatic backups and encryption at rest should always be enabled. They are disabled here only to reduce lab provisioning time.

## Database Endpoint

After approximately 4 minutes, the database status changed to:

```text
Available
```

The **Endpoint** value was copied from the **Connectivity & security** section:

```text
lab-db.xxxx.us-east-1.rds.amazonaws.com
```

![Creating Database](assets/creating%20DB.png)

This endpoint is the DNS address used by the web application to connect to the database. It automatically points to the current primary instance and will update to the standby during a failover event.

---

# 🌐 Task 4 — Connect the Web Application to RDS

With the database running, the pre-deployed EC2 web application was configured to use the RDS instance.

## Accessing the Web Server

The WebServer IP address was retrieved from the lab environment details. Opening the IP address in a browser displayed the running web application showing EC2 instance information.

## Configuring the Database Connection

The **RDS** link at the top of the web application page was selected, which presented a database configuration form.

The following connection settings were entered:

| Field | Value |
| :--- | :--- |
| Endpoint | `lab-db.xxxx.us-east-1.rds.amazonaws.com` |
| Database | `lab` |
| Username | `main` |
| Password | `lab-password` |

After clicking **Submit**, the application executed its database initialization routine. Within a few seconds, the **Address Book** application interface appeared.

![Connecting to Database](assets/connecting%20to%20db.png)

## Application Behavior

The Address Book application:

* Reads and writes contact data directly to the `lab` MySQL database on RDS
* All contact records are stored in the managed RDS instance
* Data is simultaneously replicated to the Multi-AZ standby instance in `us-east-1b`
* The application remains fully functional even if the primary database instance experiences a failure

## Testing the Application

CRUD (Create, Read, Update, Delete) operations were tested by:

* **Adding** new contact records
* **Editing** existing contact information
* **Removing** contact entries

All operations successfully persisted to the RDS database and replicated to the standby instance, confirming end-to-end functionality.

---

# ♻️ Multi-AZ Replication — How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                        Lab VPC                              │
│                                                             │
│   ┌──────────────┐         ┌──────────────────────────┐    │
│   │  EC2 Web     │  :3306  │  RDS PRIMARY (us-east-1a)│    │
│   │  Server      │────────▶│  lab-db (MySQL)          │    │
│   │              │         └────────────┬─────────────┘    │
│   └──────────────┘                      │ Synchronous       │
│                                         │ Replication       │
│                                         ▼                   │
│                          ┌──────────────────────────┐       │
│                          │  RDS STANDBY (us-east-1b)│       │
│                          │  Auto-Failover Ready     │       │
│                          └──────────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

| Replication Property | Detail |
| :--- | :--- |
| Type | Synchronous — data written to standby before primary confirms write |
| Failover Trigger | Automatic — detects primary failure within seconds |
| Failover Time | ~1–2 minutes — DNS endpoint updated automatically |
| Data Loss | Zero — synchronous replication guarantees no data loss |
| Manual Action | None required — fully automated by AWS |

---

# 🧪 Lab Validation

The database environment was successfully validated through:

* Creating a dedicated RDS security group scoped to the Web Security Group
* Creating a DB Subnet Group spanning two Availability Zones
* Provisioning a Multi-AZ MySQL RDS instance inside the Lab VPC
* Retrieving and recording the RDS endpoint
* Connecting the web application to the RDS database
* Performing live CRUD operations against the database
* Confirming data persistence and Multi-AZ replication

---

# 🏆 Key Learning Outcomes

This lab demonstrates several foundational AWS database and networking concepts:

* Understanding managed database services vs. self-managed databases on EC2
* Designing VPC security group rules for database layer isolation
* Creating DB Subnet Groups for Multi-AZ deployments
* Provisioning Amazon RDS MySQL with high availability
* Connecting web applications to RDS using DNS endpoints
* Understanding synchronous Multi-AZ replication and automatic failover
* Applying the principle of least privilege to database network access

---

# 📊 RDS vs. Self-Managed DB on EC2

| Capability | Self-Managed on EC2 | Amazon RDS |
| :--- | :--- | :--- |
| OS Patching | Manual | AWS managed |
| DB Patching | Manual | AWS managed |
| Backups | Manual setup | Automated |
| High Availability | Manual setup | Built-in Multi-AZ |
| Failover | Manual | Automatic (~1–2 min) |
| Monitoring | Manual | CloudWatch integrated |
| Scaling | Manual | Console/API driven |
| Operational Overhead | High | Low |

---

# 📚 Conclusion

This lab provides practical experience with AWS managed relational database infrastructure using Amazon RDS and MySQL in a Multi-AZ configuration.

The workflow closely mirrors real-world enterprise cloud operations where engineers must:

* Isolate database layers using VPC security groups
* Provision managed databases with built-in high availability
* Connect web applications to cloud-managed database endpoints
* Reduce operational overhead through managed services
* Implement automatic disaster recovery at the database layer

Mastering RDS concepts forms a critical foundation for careers in:

* Cloud Engineering
* DevOps Engineering
* Database Administration
* Site Reliability Engineering (SRE)
* Solutions Architecture
* Backend Development

---

<div align="center">

### ☁️ Built with AWS Cloud Infrastructure

**Amazon RDS • MySQL • Multi-AZ • VPC • Security Groups • Managed Databases • High Availability**

</div>