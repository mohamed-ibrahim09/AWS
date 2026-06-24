# 🔐 Amazon S3 Security — Complete Reference Guide

<div align="center">

![AWS](https://img.shields.io/badge/AWS-S3%20Security-orange?style=for-the-badge&logo=amazonaws)
![Encryption](https://img.shields.io/badge/Encryption-SSE--S3%20%7C%20SSE--KMS%20%7C%20SSE--C-blue?style=for-the-badge)
![CORS](https://img.shields.io/badge/CORS-Cross--Origin-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-SAA--C03%20Ready-success?style=for-the-badge)

### ⚡ S3 Encryption, Access Control & Data Protection Blueprint

*A deep-dive technical reference covering all four S3 encryption methods, in-transit enforcement, CORS, MFA Delete, Access Logs, Pre-Signed URLs, Object Lock, Glacier Vault Lock, Access Points, and Object Lambda.*

</div>

---

## 📖 Overview

Amazon S3 security is built on two pillars: **protecting data at rest** (encryption) and **protecting data in transit** (TLS/HTTPS). On top of those, S3 provides fine-grained access control mechanisms — bucket policies, MFA Delete, pre-signed URLs, Access Points, and Object Lambda — to enforce who can read, write, or transform your objects.

---

## 🔑 1. S3 Object Encryption — The Four Methods

S3 supports four mutually exclusive encryption methods. You cannot apply more than one to the same object simultaneously.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    S3 ENCRYPTION METHODS AT A GLANCE                        │
├─────────────────┬──────────────────┬──────────────────┬─────────────────────┤
│  SSE-S3         │  SSE-KMS         │  SSE-C           │  Client-Side        │
│  (Default)      │  (KMS-managed)   │  (You provide    │  Encryption         │
│                 │                  │   the key)       │  (You do it all)    │
├─────────────────┼──────────────────┼──────────────────┼─────────────────────┤
│ AWS owns and    │ You control keys │ AWS encrypts,    │ You encrypt before  │
│ manages keys.   │ via KMS.         │ you own keys,    │ upload; AWS stores  │
│ AES-256.        │ AES-256 + audit. │ HTTPS mandatory. │ ciphertext only.    │
└─────────────────┴──────────────────┴──────────────────┴─────────────────────┘
```



---

## 🟠 2. SSE-S3 — Server-Side Encryption with S3-Managed Keys

**SSE-S3** is the default encryption for every S3 bucket and every new object upload since January 5, 2023. You do not need to do anything to enable it.

![SSE-S3 Encryption](assets/Amazon%20S3%20Encryption%20%E2%80%93%20SSE-S3.png)

```
[ You ]
   │
   │  PUT request + Header: "x-amz-server-side-encryption: AES256"
   ▼  (or HTTP/HTTPS — header optional for new buckets, it's automatic)
[ Amazon S3 Service ]
   │
   │  Encrypts object with a unique per-object key (AES-256)
   │  Then encrypts that key with a regularly-rotated S3 root key
   ▼
[ S3 Bucket — Stored Encrypted ]
   (Both the object and its key are stored; the root key is managed by S3)
```

### Key Facts

* **Algorithm:** AES-256 (strongest block cipher available).
* **Key ownership:** AWS owns and rotates the root key — you have zero visibility or control over the actual key material.
* **Required header for explicit use:** `"x-amz-server-side-encryption": "AES256"`
* **Default behavior:** Automatically applied to all new object uploads with no configuration required.
* **Cost:** No additional charge on top of S3 storage costs.

### When to choose SSE-S3

Choose SSE-S3 when you need encryption at rest with zero operational overhead, no compliance requirement to audit key usage, and no need to control who has access to the key itself.

---

## 🟡 3. SSE-KMS — Server-Side Encryption with AWS KMS Keys

**SSE-KMS** replaces S3's automatic key with an **AWS KMS key** (Customer Managed Key or AWS Managed Key), giving you full audit trails via CloudTrail and fine-grained key access control via IAM.

![SSE-KMS Encryption](assets/Amazon%20S3%20Encryption%20%E2%80%93%20SSE-KMS.png)

```
[ You ]
   │
   │  PUT request + Header: "x-amz-server-side-encryption: aws:kms"
   ▼
[ Amazon S3 Service ]
   │
   │  Calls KMS: GenerateDataKey API ──► [ AWS KMS ]
   │                                        Returns:
   │                                        - Plaintext data key (used to encrypt)
   │                                        - Encrypted data key (stored with object)
   ▼
[ S3 Bucket — Object stored encrypted + encrypted data key stored in metadata ]

─── On DOWNLOAD ────────────────────────────────────────────────────────────────
[ S3 ] calls KMS: Decrypt API to get the plaintext data key ──► decrypts object
────────────────────────────────────────────────────────────────────────────────
```

### Key Facts

* **Required header:** `"x-amz-server-side-encryption": "aws:kms"`
* **CloudTrail audit:** Every upload and download logs which KMS key was used, by which IAM principal, and at what time.
* **Key types available:**
  * **AWS Managed Key** (`aws/s3`) — Free, auto-rotated, no policy control.
  * **Customer Managed Key (CMK)** — You define the key policy, rotation schedule, and cross-account sharing. Costs $1/month per key.
* **Cross-account access:** Uses CMKs — the KMS key policy must explicitly grant access to the external account.

### ⚠️ SSE-KMS Throttling Limitation

SSE-KMS makes API calls to KMS on every upload and download. These calls count against your **KMS API quota**:

```
┌──────────────────────────────────────────────────────────────────┐
│  KMS API Quotas (per second, vary by region):                    │
│                                                                  │
│  Upload  → GenerateDataKey API call  ─► counts toward quota     │
│  Download → Decrypt API call         ─► counts toward quota     │
│                                                                  │
│  Default limits: 5,500 / 10,000 / 30,000 req/s (by region)      │
│                                                                  │
│  If you exceed this limit → requests are throttled (HTTP 429)    │
│  Fix: Request a quota increase via the Service Quotas Console    │
└──────────────────────────────────────────────────────────────────┘
```

**High-throughput S3 workloads** (e.g., data pipelines uploading thousands of files per second) are most likely to hit this limit. Use **S3 Bucket Keys** to reduce KMS API calls by up to 99%.

### S3 Bucket Keys

**S3 Bucket Keys** reduce the number of AWS KMS API calls required for SSE-KMS encrypted objects.

```
Without Bucket Keys:

Object Upload
     │
     ▼
GenerateDataKey (KMS)
     │
     ▼
Encrypt Object

Every object may require additional KMS interactions.

With Bucket Keys:

KMS
 │
 ▼
Bucket Key
 │
 ├── Object 1
 ├── Object 2
 ├── Object 3
 └── Object N
```

Amazon S3 obtains a bucket-level key from AWS KMS and uses it to derive object-level encryption keys internally, dramatically reducing KMS traffic.

**Benefits:**

* **Reduces AWS KMS request volume** — a single KMS call generates a bucket-level key that is reused for many objects.
* **Lowers SSE-KMS costs** — fewer KMS API calls means lower charges.
* **Improves scalability for high-throughput workloads** — avoids KMS throttling.
* **Maintains SSE-KMS security and auditability** — CloudTrail still logs when the bucket key is created.

> **Exam Tip:** If a scenario requires SSE-KMS but also mentions reducing KMS costs or KMS API throttling, the correct answer is often to enable S3 Bucket Keys.

### When to choose SSE-KMS

Choose SSE-KMS when you have a compliance requirement to audit who accessed what key, when. Also choose it when you need to revoke access to encrypted data by disabling or deleting the CMK.

---

## 🔴 4. SSE-C — Server-Side Encryption with Customer-Provided Keys

**SSE-C** means you supply the encryption key with every single request. AWS never stores the key — it uses it to encrypt/decrypt and then immediately discards it.

![SSE-C Encryption](assets/Amazon%20S3%20Encryption%20%E2%80%93%20SSE-C.png)

```
[ You ] ──── HTTPS ONLY ──────────────────────────────────────────────────────►
  │  PUT request + 3 mandatory headers:
  │    x-amz-server-side-encryption-customer-algorithm: AES256
  │    x-amz-server-side-encryption-customer-key: <base64-encoded 256-bit key>
  │    x-amz-server-side-encryption-customer-key-MD5: <base64-encoded MD5 digest>
  ▼
[ Amazon S3 ]
  │  Uses your key to encrypt → stores ciphertext
  │  IMMEDIATELY DISCARDS your key (never stored)
  ▼
[ S3 Bucket — encrypted object, NO copy of your key anywhere in AWS ]

─── On DOWNLOAD ────────────────────────────────────────────────────────────────
[ You ] must send the exact same key in the HTTPS request headers
[ S3 ] uses it to decrypt → returns plaintext → discards your key again
────────────────────────────────────────────────────────────────────────────────
```

### Critical Rules

* **HTTPS is mandatory** — HTTP requests with SSE-C are rejected with an error. The key would be transmitted in plaintext over HTTP, which is a critical security risk.
* **AWS never stores your key** — if you lose the key, you permanently lose access to that object. There is no recovery path.
* **You are responsible for key rotation** — AWS cannot rotate a key it does not store.
* **SSE-C does not require bucket-level configuration** — the client must provide the required SSE-C encryption headers with every upload and download request. Amazon S3 never stores the customer-provided key and cannot recover encrypted objects if the key is lost.
* **Cannot be used via the S3 Console** — SSE-C is only available via CLI, SDK, or REST API.

### When to choose SSE-C

Choose SSE-C when your security policy requires that no key material ever leaves your environment — not even into a managed service like KMS. This is a niche requirement (e.g., certain financial or defense workloads). For most use cases, SSE-KMS with a CMK is a better alternative.

---

## 🟢 5. Client-Side Encryption

**Client-Side Encryption** means your application encrypts the data **before** it is sent to S3. S3 stores only ciphertext and has no knowledge of the encryption key or the plaintext.

![Client-Side Encryption](assets/Amazon%20S3%20Encryption%20%E2%80%93%20Client-Side%20Encryption.png)

```
[ Your Application ]
   │
   │  1. Retrieve encryption key (from KMS or your own key store)
   │  2. Encrypt the file locally using the Amazon S3 Client-Side Encryption Library
   ▼
[ Encrypted File (ciphertext) ]
   │
   │  Upload via HTTP or HTTPS (data is already encrypted before leaving your machine)
   ▼
[ Amazon S3 Bucket — stores only the encrypted blob ]
   (AWS sees only random bytes — no plaintext ever sent to AWS)

─── On DOWNLOAD ────────────────────────────────────────────────────────────────
[ Download encrypted blob from S3 ]
[ Your Application decrypts it locally using the same key ]
────────────────────────────────────────────────────────────────────────────────
```

### Key Facts

* Uses the **Amazon S3 Client-Side Encryption Library** (available in AWS SDKs for Java, Python, etc.).
* Key management is entirely your responsibility — AWS never sees the key.
* Encryption happens in your application's memory before the `PutObject` API call.
* The only way to guarantee no plaintext ever leaves your environment — even SSE-C sends your key (but not the plaintext) to AWS over HTTPS.

### When to choose Client-Side Encryption

Choose Client-Side Encryption when your compliance requirement states that unencrypted data must **never** leave your infrastructure, even over an encrypted channel. This is the strictest option and imposes the most operational complexity.

---

## 📊 6. Encryption Method Comparison Matrix

| Attribute | SSE-S3 | SSE-KMS | SSE-C | Client-Side |
|---|---|---|---|---|
| **Key owner** | AWS | AWS / You (CMK) | You | You |
| **Key visibility** | None | CloudTrail audit | None (discarded) | Full |
| **AWS stores key?** | Yes (AWS-managed) | Yes (KMS-managed) | ❌ Never | ❌ Never |
| **HTTPS required?** | No | No | ✅ Yes (mandatory) | No |
| **Audit trail** | None | ✅ CloudTrail | None | None |
| **KMS API quota risk** | None | ✅ Yes | None | None |
| **Console support** | ✅ | ✅ | ❌ | ❌ |
| **Cost overhead** | None | KMS key + API calls | None | None |
| **Plaintext sent to AWS?** | Yes (over HTTPS) | Yes (over HTTPS) | ✅ Yes (key only) | ❌ Never |
| **Required header** | `AES256` | `aws:kms` | 3 custom headers | None (app-level) |

---

## 🔒 7. Encryption in Transit — SSL/TLS

All data moving between your client and S3 should be protected using **TLS (Transport Layer Security)**, also referred to as SSL/TLS or "encryption in flight."

```
┌────────────────────────────────────────────────────────────────────┐
│  S3 ENDPOINT OPTIONS:                                              │
│                                                                    │
│  HTTP  → http://bucket.s3.amazonaws.com/key                        │
│           Data travels as PLAINTEXT — anyone on the network can    │
│           intercept and read it. Never use for sensitive data.     │
│                                                                    │
│  HTTPS → https://bucket.s3.amazonaws.com/key                       │
│           Data is encrypted in transit via TLS 1.2+.               │
│           Recommended for all workloads.                           │
│           MANDATORY when using SSE-C.                              │
└────────────────────────────────────────────────────────────────────┘
```

### Forcing HTTPS via Bucket Policy

You can use a bucket policy with the `aws:SecureTransport` condition key to **deny all HTTP requests** at the bucket level:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureConnections",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

**How it works:** `aws:SecureTransport` evaluates to `true` for HTTPS requests and `false` for HTTP requests. The policy denies any request where it is `false` — meaning only HTTPS requests are allowed through.

---

## ⚙️ 8. Default Encryption vs. Bucket Policy Enforcement

Understanding the evaluation order is critical for the exam:

![Default Encryption vs Bucket Policies](assets/Amazon%20S3%20%E2%80%93%20Default%20Encryption%20vs.%20Bucket%20Policies.png)

```
[ PUT Object Request Arrives ]
             │
             ▼
[ Step 1: Bucket Policy Evaluated FIRST ]
   Does the policy Deny this request?
   ├── YES → Request REJECTED (403 Forbidden)
   └── NO  → Continue
             │
             ▼
[ Step 2: Default Bucket Encryption Applied ]
   If no encryption header was set in the request,
   apply the bucket's default encryption (SSE-S3 by default)
```

> ⚠️ **Exam trap:** Bucket policies are evaluated **before** default encryption. This means a bucket policy can block an upload that doesn't include the required encryption header, even if the bucket has a default encryption setting.

### Force SSE-KMS — Bucket Policy

This policy **denies** any `PutObject` request that does not include the `aws:kms` encryption header:

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Sid": "RequireKMSEncryption",
    "Effect": "Deny",
    "Action": "s3:PutObject",
    "Principal": "*",
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
      "StringNotEquals": {
        "s3:x-amz-server-side-encryption": "aws:kms"
      }
    }
  }
}
```

### Force SSE-C — Bucket Policy

This policy **denies** any `PutObject` request that does not include the SSE-C customer algorithm header:

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Sid": "RequireSSE-C",
    "Effect": "Deny",
    "Action": "s3:PutObject",
    "Principal": "*",
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
      "Null": {
        "s3:x-amz-server-side-encryption-customer-algorithm": "true"
      }
    }
  }
}
```

**Condition logic:** `Null: true` means "the header is absent." So the policy denies uploads where the SSE-C header is absent — forcing all uploads to include an SSE-C key.

---

## 🌐 9. CORS — Cross-Origin Resource Sharing

CORS is a browser security mechanism that controls which origins can make HTTP requests to a server hosted at a **different origin**.

![CORS Overview](assets/What%20is%20CORS.png)

### What is an Origin?

An origin is defined by three components: **scheme + host + port**

```
Origin = scheme + host + port

Same origin  : http://example.com/app1  &  http://example.com/app2
Different origin: http://www.example.com  &  http://other.example.com
                  (different hosts → CORS required)
```

### The CORS Preflight Flow

Before making the actual cross-origin request, browsers send a **preflight** request using the `OPTIONS` HTTP method to check if the server permits the cross-origin access:

```
[ Web Browser ]
      │
      │  1. OPTIONS request (Preflight)
      │     Host: www.other.com
      │     Origin: https://www.example.com
      │     Access-Control-Request-Method: GET
      ▼
[ www.other.com (Cross-Origin Server / S3 Bucket) ]
      │
      │  2. Preflight Response (if CORS is configured)
      │     Access-Control-Allow-Origin: https://www.example.com
      │     Access-Control-Allow-Methods: GET, PUT, DELETE
      ▼
[ Web Browser ] — CORS check PASSED
      │
      │  3. Actual GET request
      ▼
[ www.other.com ] — Returns the resource
```

If the S3 bucket does not have a CORS configuration matching the requesting origin, the browser blocks the response and the JavaScript code receives a CORS error.

### S3 CORS Configuration Example

**Scenario:** A static website is hosted in `my-bucket-html`. The `index.html` file loads an image (`coffee.jpg`) from a separate S3 bucket `my-bucket-assets`. The assets bucket must have a CORS configuration that allows requests from the website bucket's origin.

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT"],
    "AllowedOrigins": [
      "http://my-bucket-html.s3-website.us-west-2.amazonaws.com"
    ],
    "ExposeHeaders": [],
    "MaxAgeSeconds": 3000
  }
]
```

> You can also use `"AllowedOrigins": ["*"]` to allow **all** origins — but this should only be used for fully public content where there is no risk of sensitive data exposure.

---

## 🔐 10. MFA Delete

**MFA Delete** adds an extra layer of protection for the two most destructive S3 operations by requiring a **Multi-Factor Authentication token** from the bucket owner before they can be performed.

```
┌─────────────────────────────────────────────────────────────────────┐
│  MFA REQUIRED FOR:                  │  MFA NOT REQUIRED FOR:        │
│  ─────────────────────────────────  │  ──────────────────────────── │
│  • Permanently delete object version│  • Enable Versioning          │
│  • Suspend Versioning on the bucket │  • List deleted versions      │
└─────────────────────────────────────┴───────────────────────────────┘
```

### Prerequisites and Constraints

* **Versioning must be enabled** before MFA Delete can be activated.
* **Only the bucket owner (root account)** can enable or disable MFA Delete — IAM users, even admins, cannot.
* **Supported MFA devices:** Virtual MFA (Google Authenticator, Authy), Hardware MFA tokens.
* Once enabled, any request to permanently delete a versioned object or suspend versioning must include the `x-amz-mfa` header containing: `<device-serial-number> <MFA-code>`.

### CLI Command to Delete with MFA

```bash
aws s3api delete-object \
  --bucket my-bucket \
  --key my-object.txt \
  --version-id <VERSION_ID> \
  --mfa "arn:aws:iam::123456789012:mfa/root-account-mfa-device 123456"
```

---

## 📋 11. S3 Access Logs

S3 Access Logs record **every request** made to a bucket — whether authorized or denied — into a designated **target logging bucket**.

```
[ Source Bucket: my-production-bucket ]
              │
              │  Any request (GET, PUT, DELETE, HEAD)
              │  from any account, authorized or denied
              ▼
[ Amazon S3 Logging Service ]
              │
              ▼
[ Target Logging Bucket: my-logs-bucket ]
   (must be in the SAME AWS region)
   Log entries contain:
   - Requester identity
   - Operation type
   - HTTP status code
   - Error code (if any)
   - Bytes transferred
   - Timestamp
              │
              ▼
[ Analysis: Amazon Athena / S3 Select / third-party tools ]
```

### ⚠️ Critical Warning — Logging Loop

> **Never set the logging bucket and the source bucket to be the same bucket.**

![S3 Access Logs Loop](assets/S3%20Access%20Logs%20loop.png)

If the logging bucket equals the monitored bucket, every log write generates a new log entry, which triggers another log write, and so on — creating an **infinite logging loop** that causes your bucket storage to grow exponentially and your costs to spiral out of control.

```
WRONG SETUP (DO NOT DO THIS):
[ my-bucket ] ─► logs to ─► [ my-bucket ]
                                   │
                                   │ Log write triggers another log entry
                                   └──────────────────────────────────►  ∞ loop!

CORRECT SETUP:
[ my-production-bucket ] ─► logs to ─► [ my-logs-bucket ]
```

---

## 🔗 12. Pre-Signed URLs

A **pre-signed URL** is a time-limited URL that embeds AWS credentials and a cryptographic signature, granting temporary access to a specific S3 object for anyone who has the URL — without requiring them to have AWS credentials.

![Pre-Signed URLs](assets/Amazon%20S3%20%E2%80%93%20Pre-Signed%20URLs.png)

```
[ URL Generator (IAM user, role, or root) ]
              │
              │  Generates a signed URL using their own credentials
              │  URL contains: bucket, object key, expiry time, signature
              ▼
[ Pre-Signed URL ]
  Example: https://my-bucket.s3.amazonaws.com/video.mp4
           ?X-Amz-Algorithm=AWS4-HMAC-SHA256
           &X-Amz-Credential=AKIAIOSFODNN7EXAMPLE
           &X-Amz-Expires=3600
           &X-Amz-Signature=<calculated-signature>
              │
              │  Share with anyone (email, SMS, API response, embed in app)
              ▼
[ End User ]
   Accesses the URL in a browser or app — no AWS credentials needed
   They inherit EXACTLY the permissions of the generator — no more, no less
```

### Expiration Limits

| Generation Method | Minimum | Maximum |
|---|---|---|
| **S3 Console** | 1 minute | 720 minutes (12 hours) |
| **AWS CLI** | 1 second | 604,800 seconds (7 days) |
| **AWS SDK** | 1 second | 604,800 seconds (7 days) |
| **Temporary credentials (STS AssumeRole)** | — | Expires with the session (max 1 hour by default) |

### Common Use Cases

* **Premium content access:** Generate a URL for a logged-in subscriber to download a video — the URL expires after 1 hour.
* **Secure file upload:** Give an external contractor a URL to `PUT` a file directly to S3 without exposing your bucket credentials.
* **Dynamic file sharing:** Generate a fresh URL per request so access cannot be bookmarked or shared permanently.

### CLI Generation

```bash
# Generate a pre-signed GET URL expiring in 3600 seconds (1 hour)
aws s3 presign s3://my-bucket/my-file.pdf --expires-in 3600

# Generate a pre-signed PUT URL for upload
aws s3 presign s3://my-bucket/upload-slot.csv --expires-in 7200
```

---

## 🧊 13. S3 Glacier Vault Lock

**Glacier Vault Lock** is a compliance mechanism that enforces a **Write Once, Read Many (WORM)** policy on a Glacier vault. Once locked, the policy **cannot be changed or deleted** — even by AWS.

```
[ S3 Glacier Vault ]
        │
        │  Step 1: Create a Vault Lock Policy
        │    (define retention rules, e.g., "no delete for 7 years")
        ▼
[ Policy in "InProgress" state — you can test and validate ]
        │
        │  Step 2: Lock the Policy (within 24 hours)
        ▼
[ Policy LOCKED — immutable, cannot be changed or deleted by anyone ]
        │
        │  Any attempt to delete the vault or its archives before
        │  the policy period expires → BLOCKED
        ▼
[ Compliance guaranteed for the retention period ]
```

### Key Points

* The vault lock policy is **irrevocable** once locked. Even AWS support cannot override it.
* Useful for compliance frameworks requiring tamper-proof archival: **SEC Rule 17a-4, HIPAA, CJIS**.
* You have a **24-hour window** to validate the policy before permanently locking it.

---

## 🔒 14. S3 Object Lock (Versioning Required)

**S3 Object Lock** extends WORM protection to individual S3 objects (not just Glacier). Versioning must be enabled on the bucket before Object Lock can be used.

### Two Retention Modes

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  COMPLIANCE MODE                    │  GOVERNANCE MODE                       │
│  ─────────────────────────────────  │  ──────────────────────────────────    │
│  • No one can delete/overwrite the  │  • Most users cannot delete/overwrite  │
│    object version during retention, │    during retention period.            │
│    including the ROOT user.         │  • Users with s3:BypassGovernanceRe-   │
│  • Retention mode itself cannot     │    tention IAM permission CAN override │
│    be changed to a less strict one. │    the lock and delete the object.     │
│  • Retention period cannot be       │  • Retention period CAN be shortened   │
│    shortened, only extended.        │    by authorized users.                │
│                                     │                                        │
│  → Use for: strict regulatory       │  → Use for: protection against         │
│    compliance (SEC, HIPAA, etc.)    │    accidental deletion + some flex     │
└─────────────────────────────────────┴────────────────────────────────────────┘
```

### Legal Hold

A **Legal Hold** is separate from the retention period — it protects an object **indefinitely**, regardless of when the retention period expires.

```
Object has:
  Retention period (expires: 2027-01-01) AND Legal Hold: ON

2027-01-01 arrives → retention period expires, but:
  Legal Hold is STILL active → object CANNOT be deleted

Legal Hold must be explicitly removed using the s3:PutObjectLegalHold IAM permission.
Once Legal Hold is removed → object can be deleted (if retention period is also expired).
```

### Object Lock Summary Table

| Feature | Compliance | Governance | Legal Hold |
|---|---|---|---|
| **Root user can delete?** | ❌ No | ✅ With bypass permission | ❌ No |
| **Retention changeable?** | Extend only | ✅ Yes (with permission) | N/A |
| **Duration** | Fixed period | Fixed period | Indefinite |
| **Use case** | Strict compliance | Flexible protection | Legal proceedings |

---

## 📍 15. S3 Access Points

**S3 Access Points** simplify security management for large shared S3 buckets by creating dedicated named access endpoints, each with its own policy, for different user groups or applications.

![S3 Access Points](assets/S3%20%E2%80%93%20Access%20Points.png)

```
[ S3 Bucket: company-data ]
    /finance/   /sales/   /analytics/   /raw/
         │
         ▼
┌────────────────────────────────────────────────────────────┐
│                   S3 ACCESS POINTS                         │
├──────────────────┬──────────────────┬──────────────────────┤
│  finance-ap      │  sales-ap        │  analytics-ap        │
│  Allows access   │  Allows access   │  Allows access to    │
│  to /finance/*   │  to /sales/*     │  /analytics/* only   │
│  only            │  only            │                      │
│  Policy: Finance │  Policy: Sales   │  Policy: Analytics   │
│  IAM roles only  │  IAM roles only  │  IAM roles only      │
└──────────────────┴──────────────────┴──────────────────────┘
```

### Each Access Point Has

* Its own **DNS name** (internet-accessible or VPC-only).
* Its own **Access Point Policy** (similar in structure to a bucket policy).

### VPC-Only Access Points

You can restrict an Access Point so it is only accessible from inside a specific VPC:

```
[ EC2 Instance in VPC ]
         │
         │  Request to Access Point
         ▼
[ VPC Endpoint (Gateway or Interface) ]
         │
         │  The VPC Endpoint Policy must ALSO allow access
         │  to both the target bucket AND the access point
         ▼
[ S3 Access Point (VPC Origin) ]
         │
         ▼
[ S3 Bucket ]
```

**VPC Endpoint Policy Example:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::company-data/*",
        "arn:aws:s3:us-east-1:123456789012:accesspoint/finance-vpc-ap/object/*"
      ]
    }
  ]
}
```

> Both the **VPC Endpoint Policy** AND the **Access Point Policy** must grant access for the request to succeed. They are evaluated together.

---

## ⚡ 16. S3 Object Lambda

**S3 Object Lambda** lets you attach a Lambda function to an S3 Access Point so that it runs **on every GET request** and transforms the object before it is returned to the caller — without creating a second copy of the data.

![S3 Object Lambda](assets/S3%20Object%20Lambda.png)

```
[ Single S3 Bucket with Original Data ]
              │
              │  Only ONE copy of the data ever exists
              ▼
[ Supporting S3 Access Point ]
   (Standard access point required as the backing source)
              │
    ┌─────────┴─────────────────────────────────┐
    ▼                                           ▼
[ Object Lambda Access Point 1 ]    [ Object Lambda Access Point 2 ]
  Lambda: Redacting Function          Lambda: Enriching Function
  (strips PII before returning)       (adds computed fields)
    │                                           │
    ▼                                           ▼
[ Analytics App ]                   [ Marketing App ]
  Gets: redacted data                Gets: enriched data
        (no SSNs, no emails)               (with geo data appended)
```

### Supported Use Cases

* **PII Redaction:** Strip sensitive fields (SSN, email, phone) for analytics environments without duplicating the dataset.
* **Format Conversion:** Convert XML to JSON, CSV to Parquet, or any format transformation on the fly.
* **Image Resizing:** Resize or watermark images dynamically based on the requesting user's plan or screen resolution.
* **Data Enrichment:** Add computed or lookup fields to an object before it reaches the consumer.

> ⚠️ **Note:** As of November 7, 2025, S3 Object Lambda is available only to existing customers currently using the service and select AWS Partner Network partners. New customers should evaluate alternative transformation architectures.

---

## 📊 17. Complete Security Feature Comparison

| Feature | Scope | Who Controls | Reversible? | Primary Use |
|---|---|---|---|---|
| **SSE-S3** | Object at rest | AWS | ✅ Yes | Default encryption, zero overhead |
| **SSE-KMS** | Object at rest | AWS + You (CMK) | ✅ Yes | Compliance audit, key revocation |
| **SSE-C** | Object at rest | You | ✅ Yes | External key management requirement |
| **Client-Side** | Before upload | You | ✅ Yes | No plaintext ever leaves your environment |
| **TLS / HTTPS** | Data in transit | AWS + You | ✅ Yes | Protect data from network interception |
| **Bucket Policy (HTTPS enforce)** | Bucket-wide | You | ✅ Yes | Block HTTP requests at API level |
| **CORS** | Browser requests | You | ✅ Yes | Allow cross-origin JS requests |
| **MFA Delete** | Bucket versioning | Root only | ✅ (by root) | Prevent accidental/malicious deletes |
| **Access Logs** | All requests | You | ✅ Yes | Audit trail for access patterns |
| **Pre-Signed URLs** | Object-level, temporary | Generating IAM principal | ✅ (via expiry) | Time-limited external access |
| **Glacier Vault Lock** | Vault | You (then immutable) | ❌ No | Long-term compliance archival (WORM) |
| **S3 Object Lock (Compliance)** | Object version | No one | ❌ (until period ends) | Regulatory WORM compliance |
| **S3 Object Lock (Governance)** | Object version | Privileged users | ✅ With permission | Flexible WORM protection |
| **Legal Hold** | Object version | IAM-authorized users | ✅ Yes | Legal proceedings, indefinite hold |
| **Access Points** | Bucket path prefix | You | ✅ Yes | Simplified multi-team access control |
| **Object Lambda** | GET object response | You | ✅ Yes | On-the-fly data transformation |

---

## 🕳️ 18. Exam Traps & Scenario Analysis

### 🚨 Trap 1: SSE-C vs. Client-Side Encryption — "No plaintext to AWS"

* **Question pattern:** "A company requires that plaintext data must never be transmitted to AWS under any circumstances. Which encryption method satisfies this?"
* **Wrong answer:** SSE-C — because you transmit the encryption key over HTTPS to AWS (the key, not the plaintext, but still something sensitive leaves your environment).
* **Correct answer:** **Client-Side Encryption** — the data is encrypted on your machine before the `PutObject` call; only ciphertext is ever transmitted to AWS.
* **Reasoning:** SSE-C sends the encryption key to S3 in the request headers (over HTTPS). Client-side encryption encrypts before the request is made, so nothing sensitive reaches AWS's network.

### 🚨 Trap 2: Bucket Policy vs. Default Encryption Evaluation Order

* **Question pattern:** "A bucket has SSE-S3 set as default encryption. A developer uploads an object without any encryption header. A bucket policy requires SSE-KMS. What happens?"
* **Wrong answer:** The object is uploaded and encrypted with SSE-S3 (default).
* **Correct answer:** The upload is **denied** with a 403 error.
* **Reasoning:** Bucket policies are evaluated **before** default encryption is applied. The policy Deny fires first — the object never reaches the default encryption step.

### 🚨 Trap 3: MFA Delete — Who Can Enable It

* **Question pattern:** "An S3 administrator (IAM admin user) wants to enable MFA Delete on a bucket. Will this work?"
* **Wrong answer:** Yes, an IAM admin has full S3 permissions.
* **Correct answer:** **No.** Only the bucket owner via the **root account** can enable or disable MFA Delete.
* **Reasoning:** MFA Delete is intentionally restricted to the root account to prevent an attacker who compromises an IAM admin from disabling the MFA Delete protection.

### 🚨 Trap 4: Glacier Vault Lock vs. S3 Object Lock

* **Question pattern:** "A company stores compliance records in S3 Glacier and needs to guarantee records cannot be deleted for 7 years, even by an administrator."
* **Wrong answer:** Use S3 Object Lock in Governance mode.
* **Correct answer:** Use **Glacier Vault Lock** OR **S3 Object Lock in Compliance mode**.
* **Reasoning:** Governance mode allows privileged users with `s3:BypassGovernanceRetention` to delete objects. Only Compliance mode (and Glacier Vault Lock) is truly irrevocable.

### 🚨 Trap 5: Access Logs — Logging Loop

* **Question pattern:** "A developer sets the S3 access log destination to be the same bucket that is being monitored. What is the result?"
* **Wrong answer:** Logs are stored correctly, just in a subfolder.
* **Correct answer:** An **infinite logging loop** is created. Every log write generates a new log entry, causing exponential storage growth and cost explosion.
* **Reasoning:** S3 treats the log-write itself as an access event, which is logged, which triggers another log write, indefinitely.

### 🚨 Trap 6: Pre-Signed URL Permissions

* **Question pattern:** "A pre-signed URL is generated by an IAM user who has `s3:GetObject` permission. The URL is shared with an external partner. The IAM user's permission is later revoked. Can the partner still use the URL?"
* **Wrong answer:** Yes, the URL has its own credentials baked in.
* **Correct answer:** **No.** The URL's validity is tied to the generating IAM principal's active credentials. When permissions are revoked, the pre-signed URL is immediately invalidated.
* **Reasoning:** Pre-signed URLs are bearer tokens of the generating principal's permissions at the time of use, not at the time of generation. Revoking permissions or deactivating the key invalidates all outstanding URLs.

---

## ⚡ 19. High-Frequency Exam Facts

| Fact | Answer |
|---|---|
| Default S3 encryption (since Jan 2023) | **SSE-S3 (AES-256), automatic** |
| SSE-C requires this protocol | **HTTPS (mandatory)** |
| SSE-C stores your key in AWS? | **No — discarded immediately** |
| SSE-KMS API call on upload | **GenerateDataKey** |
| SSE-KMS API call on download | **Decrypt** |
| Bucket policy vs. default encryption: evaluated first? | **Bucket policy** |
| Who can enable MFA Delete? | **Root account only** |
| MFA Delete requires Versioning? | **Yes** |
| Access Logs: target bucket must be in same...? | **AWS Region** |
| Pre-signed URL max expiry (console) | **12 hours (720 minutes)** |
| Pre-signed URL max expiry (CLI/SDK) | **7 days (604,800 seconds)** |
| Glacier Vault Lock: reversible after locking? | **No — permanently immutable** |
| Object Lock Compliance mode: root can bypass? | **No** |
| Object Lock Governance mode: root can bypass? | **Yes (with `s3:BypassGovernanceRetention`)** |
| Legal Hold duration | **Indefinite — until explicitly removed** |
| Object Lambda: creates a copy of the data? | **No — transforms on the fly from one source bucket** |
| CORS preflight HTTP method | **OPTIONS** |

---

## ✅ 20. Production Best Practices

* **Enable SSE-KMS with a CMK for sensitive data** — SSE-S3 is secure but invisible. SSE-KMS with a CMK gives you CloudTrail audit trails, key rotation control, and the ability to instantly revoke access to all encrypted data by disabling the key.
* **Use S3 Bucket Keys to reduce KMS costs** — Bucket Keys generate a short-lived, bucket-level data key from KMS, dramatically reducing the number of KMS API calls (and costs) for high-throughput SSE-KMS workloads.
* **Always enforce HTTPS via bucket policy** — Set the `aws:SecureTransport: false → Deny` policy on every bucket. This is a one-time configuration that protects against accidental HTTP requests from SDKs or misconfigured clients.
* **Keep access log buckets completely separate** — Never use the monitored bucket as its own log destination. Create a dedicated, restricted logging bucket with no versioning and a lifecycle rule to expire logs after your retention period.
* **Set short expiry times on pre-signed URLs** — Use the minimum expiry time that satisfies your use case. Long-lived pre-signed URLs are equivalent to public access — they can be forwarded, cached, and accessed by unintended parties.
* **Enable MFA Delete on buckets containing critical data** — Enables a second human verification step for permanent version deletions, protecting against both accidental and malicious data destruction.
* **Use Access Points instead of complex bucket policies** — As a bucket grows to serve multiple teams, Access Points provide a clean, maintainable alternative to a single bucket policy with dozens of conditions.
* **Use Compliance mode for regulatory WORM requirements** — Governance mode provides flexibility but can be bypassed. If a regulation requires truly irrevocable immutability (SEC 17a-4, HIPAA), use Compliance mode or Glacier Vault Lock.