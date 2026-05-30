# 🔐 IAM – Exam Notes

## 🌍 Global Service

IAM is a Global AWS Service.

Users, Groups, Roles, and Policies are managed globally and are not tied to a specific AWS Region.

---

## 👑 Root User

The Root User is created automatically when an AWS account is created.

### 📌 Characteristics

* ✅ Full administrative access.
* 🚫 Cannot be restricted by IAM policies.
* ⚠️ Should not be used for daily operations.
* 🔐 MFA should always be enabled.

💡 Use the Root User only for account-level operations.

---

## 👤 IAM Users

An IAM User represents a physical person or application that requires access to AWS.

### 📌 Characteristics

* 🔑 Has long-term credentials.
* 🔒 Can authenticate using a password.
* 🗝️ Can authenticate using access keys.
* 📜 Can be assigned permissions directly or through groups.

### 📝 Recommended Practice

👤 One physical user should correspond to one IAM User.

---

## 👥 IAM Groups

Groups simplify permission management.

### 📌 Characteristics

* 👤 Contain IAM Users only.
* 🚫 Cannot contain other Groups.
* 🔄 Users can belong to multiple Groups.
* 📜 Permissions assigned to Groups are inherited by Group members.

💡 Groups are used to manage permissions at scale.

---

## 📜 IAM Policies

Policies are JSON documents used to define permissions.

### ❓ Policies Answer

* ✅ What action is allowed or denied?
* 🎯 On which resource?
* ⚙️ Under which conditions?

### 🏗️ Core Components

* 📌 Version
* 📄 Statement
* ✅ Effect
* 🎬 Action
* 🎯 Resource
* ⚙️ Condition

### ➕ Optional Components

* 🆔 Id
* 🏷️ Sid
* 👤 Principal

---

## ⚖️ IAM Policy Evaluation

AWS evaluates all applicable policies before making an authorization decision.

### 🚨 Important Rule

❌ Explicit Deny always overrides Allow.

If no Allow exists, access is denied by default.

### 🔄 Evaluation Order

1. ❌ Explicit Deny
2. ✅ Explicit Allow
3. 🚫 Default Deny

---

## 🛡️ Least Privilege Principle

Permissions should be limited to only those required to perform a specific task.

### 🎯 Benefits

* 🔒 Reduced attack surface.
* ⚠️ Reduced risk of accidental modifications.
* 🛡️ Improved security posture.

---

## 🔐 Multi-Factor Authentication (MFA)

MFA combines:

* 🧠 Something you know.
* 📱 Something you have.

### 🎯 Benefits

* 🔒 Protects against credential theft.
* 🛡️ Protects against password compromise.
* ⭐ Strongly recommended for privileged users.

### 👑 Recommended Targets

* 👑 Root User
* 👨‍💼 Administrative IAM Users

---

## 🗝️ Access Keys

Access Keys provide programmatic access to AWS.

### 📌 Components

* 🆔 Access Key ID
* 🔑 Secret Access Key

### 💻 Used By

* 🖥️ AWS CLI
* 💻 AWS SDK
* 🌐 AWS APIs

### 🛡️ Security Recommendations

* 🚫 Never share access keys.
* 🔄 Rotate access keys regularly.
* 🗑️ Remove unused access keys.

---

## 🖥️ AWS CLI

The AWS Command Line Interface provides command-based interaction with AWS services.

### 📌 Characteristics

* 🗝️ Uses access keys.
* 🌐 Directly accesses AWS APIs.
* 🤖 Supports automation and scripting.
* 💻 Cross-platform.

---

## 💻 AWS SDK

AWS SDKs allow applications to interact with AWS services programmatically.

### 📌 Characteristics

* 🌍 Language-specific.
* 📦 Embedded into applications.
* 🔑 Uses access keys or temporary credentials.

### 🚀 Common SDKs

* 🐍 Python (Boto3)
* ☕ Java
* 📜 JavaScript
* 🐹 Go
* 🔷 .NET

---

## 🎭 IAM Roles

IAM Roles provide temporary credentials.

### 📌 Characteristics

* 🚫 No long-term credentials.
* 🔄 Automatically rotated credentials.
* 🔗 Assumable by AWS services, users, or applications.

### ☁️ Common Service Roles

* 🖥️ EC2 Roles
* ⚡ Lambda Roles
* 🏗️ CloudFormation Roles

✅ Preferred over storing Access Keys.

---

## 🔍 Credentials Report

IAM Credentials Report provides account-level visibility into credential status.

### 📊 Information Included

* 🔒 Password status
* 🔐 MFA status
* 🗝️ Access key status
* ⏳ Credential age

### 🎯 Primary Use

🛡️ Security auditing.

---

## 🔎 Access Advisor

IAM Access Advisor provides visibility into service usage.

### 📊 Information Included

* 📜 Granted permissions
* 🕒 Last accessed services

### 🎯 Primary Use

⚙️ Permission optimization and least privilege implementation.

---

## 🔑 Password Policy

Password Policies improve account security.

### ⚙️ Configurable Parameters

* 📏 Minimum password length
* 🔠 Uppercase characters
* 🔡 Lowercase characters
* 🔢 Numbers
* ✨ Special characters
* ⏳ Password expiration
* ♻️ Password reuse prevention

---

## 🔥 High-Frequency Exam Facts

* 🌍 IAM is Global.
* 👑 Root User has unrestricted access.
* 👥 Groups contain Users only.
* 🚫 Groups cannot contain Groups.
* 🔄 Users can belong to multiple Groups.
* 📜 Policies are written in JSON.
* ❌ Explicit Deny overrides Allow.
* 🗝️ Access Keys are used by CLI and SDK.
* 🔐 MFA protects Console access.
* 🎭 Roles provide temporary credentials.
* 🖥️ EC2 should use IAM Roles instead of Access Keys.
* 🔍 Credentials Report is account-level.
* 🔎 Access Advisor is identity-level.
* 🛡️ Least Privilege is a core AWS security principle.
