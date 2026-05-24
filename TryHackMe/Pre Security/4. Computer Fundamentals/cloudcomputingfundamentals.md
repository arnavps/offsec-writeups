# Cloud Computing Fundamentals

## Overview

Cloud computing is the delivery of computing resources — servers, storage, databases, networking, and software — over the internet on a pay-as-you-go basis. It has replaced traditional on-premises infrastructure for most modern applications. Understanding cloud fundamentals is increasingly essential in cybersecurity, as the majority of production environments now run in the cloud.

---

## Topics Covered

- Benefits and characteristics of cloud computing
- Cloud deployment models: Public, Private, Hybrid
- Cloud service models: IaaS, PaaS, SaaS
- Major cloud vendors
- Basic cloud terminology (EC2, instance types)

---

## Key Concepts

### Why Cloud Computing Exists

Traditional on-premises infrastructure required companies to buy, maintain, and manage physical servers. This meant high upfront costs, slow provisioning, and wasted capacity. Cloud computing addresses these problems by providing on-demand, scalable resources managed by a third-party provider.

---

### Cloud Benefits and Characteristics

| Benefit | Description |
|---|---|
| Scalability | Scale resources up or down based on demand |
| On-demand self-service | Provision or remove resources instantly without waiting for hardware |
| Pay-as-you-go | Charged based on actual usage, not upfront capital expenditure |
| High availability | Built-in redundancy keeps applications running even if part of the system fails |
| Global access | Deploy applications close to users anywhere in the world |
| Security | Cloud providers invest heavily in physical and infrastructure security |

---

### Cloud Deployment Models

| Model | Description | Typical Users |
|---|---|---|
| Public Cloud | Infrastructure owned and operated by a cloud provider, shared across customers | Startups, web apps, most general use cases |
| Private Cloud | Dedicated infrastructure for a single organisation, on-premises or hosted | Banks, healthcare, government — regulated industries with strict compliance requirements |
| Hybrid Cloud | Combination of public and private cloud, with data and workloads moving between them | E-commerce platforms that keep sensitive data private but scale publicly during peak demand |

---

### Cloud Service Models

The three models define how much of the stack the provider manages vs. how much the customer manages:

| Model | Provider Manages | Customer Manages | Examples |
|---|---|---|---|
| IaaS (Infrastructure as a Service) | Physical hardware, networking | OS, runtime, applications | AWS EC2, Azure VMs |
| PaaS (Platform as a Service) | Hardware + OS + runtime | Application code and data | AWS Elastic Beanstalk, Heroku |
| SaaS (Software as a Service) | Everything | Nothing (just uses the app) | Gmail, Zoom, Salesforce |

A useful way to think about it: IaaS gives you a virtual machine, PaaS gives you a platform to deploy code, SaaS gives you a finished product.

---

### Major Cloud Vendors

| Vendor | Notable Strengths |
|---|---|
| AWS (Amazon Web Services) | Industry leader — broadest service catalogue, largest global infrastructure |
| Microsoft Azure | Strong in enterprise and hybrid cloud environments |
| Google Cloud Platform (GCP) | Known for data analytics, AI, and machine learning |
| Alibaba Cloud | Major player in Asia-Pacific |
| IBM Cloud | Focused on hybrid cloud and AI for enterprise |
| Oracle Cloud | Focused on enterprise applications and databases |

AWS is the most widely used and has the largest market share. Most cloud security certifications and tools are AWS-focused.

---

### Basic Cloud Terminology

**EC2 (Elastic Compute Cloud)**
A virtual server in AWS. An EC2 instance is a virtual machine running in the cloud with its own CPU, RAM, and storage. You choose the instance type based on your workload requirements.

**Instance Types**
Define the compute power of an EC2 instance. Different families are optimised for different workloads (general purpose, compute-optimised, memory-optimised, etc.).

- Larger instance = more CPU and RAM = higher cost
- Smaller instance = less power = lower cost

Example instance type families: `t3` (general purpose, burstable), `m5` (general purpose, balanced), `c5` (compute-optimised).

---

## Important Terminology

| Term | Definition |
|---|---|
| Cloud Computing | On-demand delivery of computing resources over the internet |
| Public Cloud | Shared cloud infrastructure managed by a provider |
| Private Cloud | Dedicated cloud infrastructure for a single organisation |
| Hybrid Cloud | Mix of public and private cloud environments |
| IaaS | Infrastructure as a Service — rent virtual hardware |
| PaaS | Platform as a Service — deploy code without managing infrastructure |
| SaaS | Software as a Service — use a complete application over the internet |
| EC2 | AWS virtual server (Elastic Compute Cloud) |
| Instance Type | Defines the CPU/RAM configuration of a cloud VM |
| Scalability | Ability to increase or decrease resources based on demand |
| High Availability | System design that minimises downtime through redundancy |

---

## Real-World Relevance

- Cloud misconfigurations are one of the most common causes of data breaches — exposed S3 buckets, overly permissive IAM roles, and publicly accessible databases are frequent findings
- Cloud security is a dedicated discipline — certifications like AWS Security Specialty and tools like Prowler, ScoutSuite, and CloudMapper are used for cloud security assessments
- Understanding IaaS vs PaaS vs SaaS defines the shared responsibility model — knowing what the customer is responsible for securing vs. the provider is critical
- Attackers target cloud environments for credential theft (IAM keys), data exfiltration (storage buckets), and cryptomining (spinning up compute resources)
- Cloud metadata services (e.g., `http://169.254.169.254` on AWS) are a common SSRF target — they can expose IAM credentials to an attacker

---

## Key Learnings

- Cloud computing provides on-demand, scalable infrastructure without managing physical hardware
- Three deployment models: Public (shared), Private (dedicated), Hybrid (both)
- Three service models: IaaS (infrastructure), PaaS (platform), SaaS (software)
- AWS is the dominant cloud provider; Azure and GCP are the main competitors
- EC2 is the fundamental compute unit in AWS — a virtual server with configurable CPU and RAM

---

## Additional Notes

- The AWS free tier provides limited access to many services for 12 months — useful for hands-on learning
- Cloud security follows a shared responsibility model: the provider secures the infrastructure; the customer secures what they deploy on it
- SSRF (Server-Side Request Forgery) attacks against cloud metadata endpoints are a well-known attack vector — always restrict access to `169.254.169.254` from application code

---

## Conclusion

Cloud computing is now the default infrastructure model for most organisations. Understanding the deployment and service models, the major providers, and the basic building blocks like EC2 is essential groundwork for cloud security. The attack surface in the cloud is different from on-premises — misconfigurations and identity/access management issues dominate, rather than traditional network perimeter attacks.
