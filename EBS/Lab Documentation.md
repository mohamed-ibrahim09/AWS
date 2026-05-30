# ☁️ AWS EBS Storage & Snapshot Management Lab

<div align="center">

![AWS](https://img.shields.io/badge/AWS-EBS-orange?style=for-the-badge\&logo=amazonaws)
![Storage](https://img.shields.io/badge/Storage-Block%20Storage-blue?style=for-the-badge)
![Snapshots](https://img.shields.io/badge/Snapshots-Backup-success?style=for-the-badge)
![EC2](https://img.shields.io/badge/EC2-Linux%20Instance-yellow?style=for-the-badge)

### 💾 Amazon EBS Hands-On Storage & Snapshot Management Lab

*A comprehensive hands-on lab focused on Amazon Elastic Block Store (EBS), Linux file systems, persistent cloud storage, snapshot backups, and disaster recovery using AWS cloud infrastructure.*

</div>

---

# 📖 Overview

This lab focuses on storage management using Amazon Elastic Block Store and its integration with Amazon EC2.

![](assets/Lab%20Overview.png)

The lab demonstrates how cloud engineers provision, attach, configure, back up, and restore persistent block storage volumes for Linux-based EC2 instances.

The environment simulates a real-world production scenario where organizations require reliable cloud storage systems capable of:

* Persisting application data independently from EC2 lifecycle
* Expanding storage dynamically
* Creating point-in-time backups
* Restoring deleted or corrupted data
* Supporting Linux-based workloads

This lab introduces foundational storage concepts required for:

* Cloud Engineering
* DevOps Engineering
* Linux System Administration
* Disaster Recovery
* Solutions Architecture

---

# 🎯 Lab Objectives

By completing this lab, you will successfully:

* **Create an Amazon EBS Volume**

  * Provision persistent SSD-backed cloud storage.

* **Attach EBS Storage to EC2**

  * Connect additional storage devices to Linux instances.

* **Configure Linux File Systems**

  * Format and mount EBS volumes using ext3.

* **Persist Mount Configurations**

  * Automatically mount storage during system boot.

* **Create EBS Snapshots**

  * Generate point-in-time storage backups.

* **Restore Data from Snapshots**

  * Recover deleted files using restored EBS volumes.

* **Understand Persistent Cloud Storage**

  * Learn how EBS storage survives independently from EC2 lifecycle events.

---

# 🏢 Business Scenario

A company hosts business-critical Linux applications on AWS cloud infrastructure. The organization requires a reliable and scalable storage architecture capable of:

* Persisting data beyond EC2 restarts
* Protecting business data using backups
* Recovering deleted files quickly
* Expanding storage capacity dynamically
* Supporting disaster recovery operations

To achieve this, the organization uses:

* Amazon EC2
* Amazon EBS Volumes
* EBS Snapshots
* Linux File Systems
* Snapshot-Based Backup & Recovery

The infrastructure demonstrates enterprise-grade storage management concepts commonly used in real-world cloud environments.

---

# 🛠️ AWS Services Utilized

| Service                                     | Purpose                                                         |
| :------------------------------------------ | :-------------------------------------------------------------- |
| **💻 Amazon EC2**                           | Hosts Linux-based virtual machine instances.                    |
| **💾 Amazon EBS**                           | Provides persistent block storage volumes for EC2 instances.    |
| **📸 EBS Snapshots**                        | Creates point-in-time backups of EBS volumes.                   |
| **🗂️ Amazon S3**                           | Stores EBS snapshot data durably in AWS backend infrastructure. |
| **🖥️ AWS Systems Manager Session Manager** | Provides secure browser-based terminal access to EC2 instances. |

---

# 🧠 Core Amazon EBS Concepts

---

## 💾 Amazon Elastic Block Store (EBS)

Amazon Elastic Block Store is a persistent block-level storage service designed for EC2 instances.

EBS behaves similarly to a physical hard drive attached to a server.

Unlike temporary instance storage, EBS volumes:

* Persist independently from EC2 lifecycle
* Survive instance stops and reboots
* Support backup snapshots
* Can be detached and reattached
* Provide high durability and reliability

---

# ⚡ Amazon EBS Features

| Feature            | Description                              |
| :----------------- | :--------------------------------------- |
| Persistent Storage | Data survives EC2 shutdowns and restarts |
| High Availability  | Replicated within an Availability Zone   |
| Snapshot Support   | Point-in-time backup functionality       |
| Flexible Sizing    | Volumes from 1 GB to 16 TB               |
| High Performance   | SSD-backed storage options               |
| Easy Recovery      | Restore volumes directly from snapshots  |

---

# 💻 Existing EC2 Environment

An EC2 instance named:

```text id="h6q3tw"
Lab
```

was already provisioned for the exercise.

The instance contained:

| Configuration    | Value        |
| :--------------- | :----------- |
| Operating System | Amazon Linux |
| Root Volume      | 8 GiB        |
| Storage Type     | Amazon EBS   |

---

# 📦 Task 1 — Create a New EBS Volume

A new EBS volume was created using the following configuration:

![](assets/Creating%20Volume.png)

| Setting           | Value                       |
| :---------------- | :-------------------------- |
| Volume Name       | `My Volume`                 |
| Volume Type       | `General Purpose SSD (gp2)` |
| Size              | `1 GiB`                     |
| Availability Zone | Same AZ as EC2 instance     |

---

# 🌍 Availability Zone Requirement

EBS volumes can only attach to EC2 instances located within the same Availability Zone.

Example:

| Resource     | Availability Zone |
| :----------- | :---------------- |
| EC2 Instance | `us-east-1a`      |
| EBS Volume   | `us-east-1a`      |

If the Availability Zones differ, attachment will fail.

---

# 🔗 Task 2 — Attach the Volume to EC2

The EBS volume:

```text id="y9m4rs"
My Volume
```

was attached to the EC2 instance using the AWS Console:

![](assets/Attaching%20volume%20to%20instance.png)

using the Linux device identifier:

```text id="u3p8vn"
/dev/sdb
```

Once attached, the EBS volume state changed from:

```text id="x5n7lk"
Available → In-use
```

![](assets/Attaching%20Volume.png)

indicating successful attachment.

---

# 🖥️ Task 3 — Connect to EC2 Using Session Manager

The lab used:

AWS Systems Manager Session Manager

to establish secure browser-based terminal access to the Linux EC2 instance.

This approach eliminates the need for:

* SSH keys
* Public SSH ports
* Bastion hosts

---

# ⚙️ Task 4 — Configure Linux File System

After attaching the EBS volume, Linux still viewed the storage as a raw, unformatted disk.

![](assets/Creating%20and%20Configuring%20system.png)

The volume required:

* File system creation
* Mount configuration
* Persistent boot mounting

---

# 📊 View Existing Storage

The following command displayed current mounted storage:

```bash
df -h
```

Initially, only the original root disk appeared:

```text id="j2v8ke"
/dev/xvda1
```

representing the EC2 operating system disk.

---

# 🗂️ Create ext3 File System

The EBS volume was formatted using:

```bash
sudo mkfs -t ext3 /dev/sdb
```

This created an:

```text id="p4c7qs"
ext3
```

Linux file system on the attached storage volume.

---

# 📁 Create Mount Directory

A mount point directory was created:

```bash
sudo mkdir /mnt/data-store
```

---

# 🔗 Mount the EBS Volume

The storage volume was mounted using:

```bash
sudo mount /dev/sdb /mnt/data-store
```

The EBS volume became accessible through:

```text id="r7m2fd"
/mnt/data-store
```

---

# ♻️ Persistent Mount Configuration

To ensure automatic mounting after reboot, the following configuration was added to:

```text id="d8n5vx"
/etc/fstab
```

```bash
echo "/dev/sdb   /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab
```

This permanently registers the volume mount settings during Linux boot.

---

# 📈 Verify Mounted Storage

Running:

```bash
df -h
```

now displayed:

```text id="m1p6tz"
/dev/xvdb
```

mounted successfully under:

```text id="n4w8uy"
/mnt/data-store
```

---

# 📝 Create Test File

A test file was created on the mounted EBS volume:

```bash
sudo sh -c "echo some text has been written > /mnt/data-store/file.txt"
```

The file contents were verified using:

```bash
cat /mnt/data-store/file.txt
```

This confirmed successful write operations to the EBS storage device.

---

# 📸 Task 5 — Create Amazon EBS Snapshot

An EBS snapshot named:

```text id="t6k3zr"
My Snapshot
```

was created from the EBS volume.

![](assets/My%20Snapshot.png)

Snapshots provide:

* Point-in-time backups
* Disaster recovery capabilities
* Long-term storage durability
* Volume restoration functionality

---

# 🗂️ Snapshot Storage Architecture

EBS snapshots are stored internally by AWS using:

Amazon Simple Storage Service

which provides extremely durable object storage.

Snapshots automatically replicate across multiple Availability Zones for increased resilience.

---

# ⚡ Snapshot Optimization

Amazon EBS snapshots are incremental.

This means:

* Only modified storage blocks are copied
* Empty blocks are ignored
* Storage costs are optimized

This significantly reduces backup storage usage and improves efficiency.

---

# 🗑️ Simulating Data Loss

After creating the snapshot, the test file was deleted:

```bash
sudo rm /mnt/data-store/file.txt
```

Verification using:

```bash
ls /mnt/data-store/
```

confirmed that the file no longer existed.

![](assets/Deleting%20a%20file%20to%20restore%20from%20the%20snapshot.png)

This simulated accidental data loss.

---

# ♻️ Task 6 — Restore the EBS Snapshot

To recover the deleted data, a new EBS volume was restored directly from the snapshot.

The restored volume configuration:

| Setting           | Value             |
| :---------------- | :---------------- |
| Volume Name       | `Restored Volume` |
| Source            | `My Snapshot`     |
| Availability Zone | Same EC2 AZ       |

---

# 🔄 Snapshot Restoration Flexibility

When restoring snapshots, AWS allows administrators to modify:

* Volume size
* Volume type
* Performance tier
* Availability Zone

This enables flexible storage migrations and upgrades.

---

# 🔗 Attach Restored Volume

The restored volume was attached to the EC2 instance using:

```text id="q7h2ma"
/dev/sdc
```

as the Linux device identifier.

---

# 📁 Create New Mount Point

A second mount directory was created:

```bash
sudo mkdir /mnt/data-store2
```

---

# 🔗 Mount Restored Volume

The restored snapshot volume was mounted using:

```bash
sudo mount /dev/sdc /mnt/data-store2
```

---

# ✅ Verify Restored Data

Running:

```bash
ls /mnt/data-store2/
```

displayed:

```text id="e3v9yr"
file.txt
```

![](assets/Mounting%20the%20restored%20volume.png)

This confirmed successful recovery of the deleted file from the snapshot backup.

---

# 🧪 Lab Validation

The storage environment was successfully validated through:

* Creating EBS volumes
* Attaching storage to EC2
* Configuring Linux file systems
* Mounting persistent storage
* Writing data to EBS volumes
* Creating EBS snapshots
* Simulating file deletion
* Restoring volumes from snapshots
* Recovering deleted data successfully

![](assets/Grades%20after%20Lab.png)

---

# 🏆 Key Learning Outcomes

This lab demonstrates several foundational AWS storage and Linux administration concepts:

* Understanding persistent cloud storage
* Managing EBS volumes in AWS
* Configuring Linux block devices
* Creating and mounting file systems
* Automating persistent mounts using `/etc/fstab`
* Performing snapshot-based backups
* Recovering deleted data from snapshots
* Understanding AWS disaster recovery workflows

---

# 📚 Conclusion

This lab provides practical experience with AWS block storage management using Amazon EBS and Linux-based EC2 instances.

The workflow closely mirrors real-world enterprise cloud operations where engineers must:

* Provision reliable storage
* Protect business-critical data
* Implement backup strategies
* Recover from accidental data loss
* Maintain persistent cloud infrastructure

Mastering EBS concepts forms a critical foundation for careers in:

* Cloud Engineering
* DevOps Engineering
* Linux Administration
* Site Reliability Engineering (SRE)
* Solutions Architecture
* Cloud Security

---

<div align="center">

### ☁️ Built with AWS Cloud Infrastructure

**Amazon EBS • EC2 • Snapshots • Linux Storage • Disaster Recovery • Persistent Cloud Storage**

</div>
