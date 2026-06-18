# ☁️ AWS Solutions Architect — Documentation & Labs Repository

<div align="center">

![AWS](https://img.shields.io/badge/AWS-Solutions%20Architect-orange?style=for-the-badge&logo=amazonaws)
![Cert](https://img.shields.io/badge/Target-SAA--C03-blue?style=for-the-badge)
![Docs](https://img.shields.io/badge/Docs-In%20Progress-yellow?style=for-the-badge)
![Labs](https://img.shields.io/badge/Labs-Console%20Based-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)

### ⚡ A structured, engineering-grade documentation and lab repository for AWS services — built while studying for the SAA-C03 certification.

*Every service documented here is backed by real console-based labs, ASCII architecture diagrams, exam trap analysis, and production best practices — not just copied notes.*

</div>

---

## 📖 What This Repository Is

This repository is **two things at once**:

**A personal study system** — structured documentation I write while working through Stephane Maarek's SAA-C03 Udemy course. Each file forces me to understand a service deeply enough to explain it clearly, draw its architecture, and identify where exams try to trick you.

**A professional portfolio** — a growing knowledge base that demonstrates real AWS engineering understanding. Every document is written to the same standard: foundations, architecture diagrams, configuration rules, security model, exam traps, and best practices.

The goal is not to collect notes. The goal is to build a reference I would actually use on the job.

---

## 🏗️ How Each Service Is Documented

Every service folder follows the same structure:

```
Service-Name/
├── README.md          ← Deep-dive reference (architecture, rules, exam traps, best practices)
└── labs/
    └── lab-01-*.md   ← Console-based lab walkthroughs with screenshots and observations
```

Each `README.md` contains:

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Foundations        — What the service is and why it exists  │
│  2. Architecture       — ASCII diagrams of how it works         │
│  3. Configuration      — Key settings, rules, and limits        │
│  4. Security Model     — IAM, encryption, network controls      │
│  5. Exam Traps         — Common wrong answers and why           │
│  6. Best Practices     — Production-grade engineering guidance  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📂 Repository Structure

```
aws-docs/
│
├── EC2/
├── IAM/
├── S3/
├── RDS/
├── Aurora/
├── ElastiCache/
├── VPC/
├── ELB/
├── Auto-Scaling/
├── Lambda/
├── DynamoDB/
├── CloudWatch/
└── (expanding as I progress through the course)
```

> Services are added in the order they appear in the SAA-C03 curriculum. Each folder is self-contained — you can jump to any service without reading the others first.

---

## 📊 Coverage Progress

| Category | Services |
|---|---|
| **Compute** | EC2, Lambda, Auto Scaling |
| **Storage** | S3 |
| **Database** | RDS, Aurora, ElastiCache, DynamoDB |
| **Networking** | VPC, ELB |
| **Security & Identity** | IAM |
| **Monitoring** | CloudWatch |

> This table expands as new services are documented. Each entry gets its own folder with a full README and at least one lab.

---

## 🧪 Labs Approach

Labs in this repository are **console-based** — meaning every step is performed through the AWS Management Console, not the CLI or IaC tools.

Each lab documents:

```
┌─────────────────────────────────────────────────────────────────┐
│  Objective     — What the lab proves or demonstrates            │
│  Pre-requisites — What needs to exist before starting           │
│  Steps         — Numbered, reproducible walkthrough             │
│  Observations  — What I noticed, what surprised me             │
│  Cleanup       — How to delete all resources to avoid charges   │
└─────────────────────────────────────────────────────────────────┘
```

> Every lab ends with a **Cleanup** section. AWS bills for running resources — documenting teardown is not optional.

---

## 🎯 Certification Target

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│   🎯  AWS Certified Solutions Architect — Associate (SAA-C03)        │
│                                                                      │
│   Primary Resource:  Stephane Maarek — Ultimate AWS SAA-C03 (Udemy) │
│   Lab Environment:   AWS Management Console                          │
│   Documentation:     This repository                                 │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

The SAA-C03 exam tests the ability to design scalable, highly available, cost-optimized, and secure AWS architectures. Every service documented here is studied with those four dimensions in mind — not just "what does it do" but "when do you use it, and why not something else."

---

## ✍️ Writing Standard

Documentation in this repository is written to be useful under exam pressure and useful on the job. That means:

* **No copy-pasted AWS docs** — everything is rewritten in my own words after understanding it.
* **Ambiguous concepts get explicit explanations** — if a term like "eventual consistency" or "SYNC replication" appears, it is defined in context, not assumed.
* **Exam traps are called out explicitly** — wrong answers are documented alongside correct ones, with reasoning.
* **ASCII diagrams over walls of text** — architecture is visual; prose alone is not enough.

---

## 👨‍💻 Author

**Mohamed Ibrahim**
Junior DevOps & Cloud Engineer · AI Student · SAA-C03 Candidate

Faculty of Artificial Intelligence — Menofia University (Graduating 2027)
DEPI DevOps Trainee · CCNA — NTI (2025)

---

## 📄 License

This repository is for educational and portfolio purposes.
Content is original documentation written during personal study — not redistributed course material.