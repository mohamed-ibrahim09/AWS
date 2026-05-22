# ☁️ AWS IAM: Users, Groups, and Policies

<div align="center">

![AWS](https://img.shields.io/badge/AWS-IAM-orange?style=for-the-badge&logo=amazonaws)
![Security](https://img.shields.io/badge/Security-Access%20Control-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

### 🔐 Identity & Access Management (IAM) Hands-On Lab

*A comprehensive guide to managing authentication, authorization, users, groups, and precise permissions in Amazon Web Services using real-world scenarios.*

</div>

---

## 📖 Overview

This documentation provides a detailed walkthrough of an AWS Identity and Access Management (IAM) lab. It demonstrates how organizations can enforce security best practices—specifically **Role-Based Access Control (RBAC)** and the **Principle of Least Privilege**—to securely manage user access to cloud infrastructure.

![Lab Overview](assets/Lab%20Overview.png)

---

## 🎯 Lab Objectives

The primary goal of this lab is to provide hands-on experience with AWS IAM. By the end of this exercise, you will have successfully:

- **Explored IAM Users and Groups:** Understanding the core entities of AWS identity management.
- **Analyzed AWS Managed Policies:** Reviewing pre-defined policies provided by AWS for common use cases.
- **Created and Attached Inline Policies:** Crafting custom permission sets mapped to specific business needs.
- **Implemented Role-Based Access:** Assigning IAM users to corresponding groups to inherit permissions dynamically.
- **Validated Permissions:** Testing access boundaries by switching between different IAM user contexts.
- **Enforced Security Controls:** Ensuring robust access control to vital AWS resources.

---

## 🏢 Business Scenario

To simulate a real-world enterprise environment, this lab revolves around a scenario where an organization requires distinct access levels for its support and administrative teams. By logically grouping users and defining stringent policies, we prevent unauthorized access to sensitive services like S3 and EC2.

![Business Scenario](assets/Business%20Scenario.png)

---

## 🛠️ AWS Services Utilized

This lab leverages the following core AWS services:

| Service | Purpose |
| :--- | :--- |
| **🔐 IAM (Identity and Access Management)** | The centralized service for managing access to AWS resources securely. |
| **💻 Amazon EC2 (Elastic Compute Cloud)** | Scalable virtual servers where access must be strictly regulated. |
| **🪣 Amazon S3 (Simple Storage Service)** | Highly durable object storage service, requiring distinct read-only access in our scenario. |

---

## 👥 Preconfigured IAM Entities

The lab environment comes pre-provisioned with the following IAM users to simulate a multi-tiered operations team.

| IAM User | Assigned Role |
| :--- | :--- |
| 👤 `user-1` | **Amazon S3 Support** - Requires read-only access to storage buckets. |
| 👤 `user-2` | **Amazon EC2 Support** - Requires read-only access to view compute instances. |
| 👤 `user-3` | **Amazon EC2 Administrator** - Requires elevated privileges to manage compute instances. |

---

## 👨‍👩‍👧 IAM Groups & Access Mapping

To adhere to best practices, permissions are assigned to **Groups** rather than individual users. Users inherit permissions based on their group membership.

| Group Name | Access Level / Permissions |
| :--- | :--- |
| 🪣 **S3-Support** | **Read-only** access to Amazon S3. |
| 💻 **EC2-Support** | **Read-only** access to Amazon EC2. |
| ⚙️ **EC2-Admin** | **Operational** access to Start, Stop, and View Amazon EC2 instances. |

---

## 📜 IAM Policies Examined

Policies define the exact permissions granted to a group or user. We utilize a combination of AWS Managed Policies and custom Inline Policies.

### 🪣 AmazonS3ReadOnlyAccess (AWS Managed Policy)
Grants read-only permissions for Amazon S3 resources.
- `s3:Get*`
- `s3:List*`

### 💻 AmazonEC2ReadOnlyAccess (AWS Managed Policy)
Grants read-only monitoring and viewing access for Amazon EC2.
- View EC2 resources (`ec2:Describe*`)
- Access to basic CloudWatch metrics.

### ⚙️ EC2-Admin (Custom Inline Policy)
A tightly scoped custom policy that allows specific operational actions on EC2 instances.
- Start EC2 instances (`ec2:StartInstances`)
- Stop EC2 instances (`ec2:StopInstances`)
- Describe EC2 instances (`ec2:DescribeInstances`)

---

## 🧠 IAM Policy Anatomy

AWS IAM policies are JSON documents that define the permissions applied to an identity or resource. Every statement within a policy contains the following core elements:

| Element | Description |
| :--- | :--- |
| ✅ **Effect** | Declares whether the policy allows or denies access (`Allow` or `Deny`). |
| ⚡ **Action** | Specifies the exact AWS API operations the policy applies to (e.g., `s3:ListBucket`). |
| 🎯 **Resource** | Defines the specific AWS resources the actions can be performed on (e.g., a specific bucket ARN, or `*` for all resources). |

### Example Policy Structure

Below is a foundational example of a policy granting read-only access to all EC2 instances:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*"
    }
  ]
}
```

---

## 🧪 Lab Execution & Access Validation

The theoretical configurations were put to the test to ensure that the Principle of Least Privilege is actively functioning.

### 1️⃣ Scenario 1: Unassigned User Access
Initially, if `user-1` attempts to access AWS services without being assigned to the appropriate group, they encounter an **API Authorization Error**, verifying that default access is strictly denied.
![User1 Authorization Problem](assets/User1%20authorization%20Problem.png)

To remediate this, `user-1` is successfully added to the **S3-Support** group, instantly inheriting the required read-only policies.
![Adding User1 to S3-Support Group](assets/Adding%20User1%20to%20S3-Support%20group.png)

### 2️⃣ Scenario 2: Enforcing Boundaries for Support
`user-2` is assigned to **EC2-Support**, a group explicitly restricted to read-only visibility. If this user attempts a write or operational action—such as modifying or stopping an instance—the action fails immediately, proving that the permission boundaries are airtight.
![User2 Authorization Problem](assets/User2%20authorizatioin%20problem.png)

### 3️⃣ Scenario 3: Validating Administrative Actions
Conversely, `user-3` belongs to the **EC2-Admin** group and possesses the elevated custom inline policy. When this user executes the command to stop the EC2 instances, the API call succeeds without issue.
![User3 Stopping EC2 Instances](assets/User3%20Stopping%20the%20instances.png)

### 🏆 Final Lab Validation
Upon successful assignment of all users and verification of correct access scopes, the lab requirements are fully satisfied.
![Lab Completion Grade](assets/Grade%20after%20solution.png)

---

*This lab provides the fundamental building blocks necessary for architecting secure, compliant, and scalable access management within the AWS Cloud.*