# ☁️ AWS Basics

Amazon Web Services (AWS) is the world's leading cloud platform. This section explores what AWS is, its advantages, infrastructure, pricing, security, and architectural frameworks.

---

## 📌 AWS Overview

Launched in **2006**, AWS began by offering IT infrastructure services as web services. The goal was to solve infrastructure-related challenges that traditional setups couldn’t scale with. AWS enables businesses to:
- Provision servers instantly
- Scale globally within minutes
- Lower infrastructure costs
- Focus on business outcomes instead of physical hardware

---

## ✅ Advantages of AWS Cloud Computing

- **💸 Trade capital expense for variable expense** – Pay only for what you use. No need to buy extra hardware "just in case".
- **📉 Benefit from economies of scale** – Lower prices as more customers use AWS.
- **📊 Stop guessing capacity** – Scale compute and storage based on actual need.
- **⚡ Increase speed and agility** – Launch resources with a few clicks, accelerating development.
- **💰 Save on maintenance costs** – Focus on your applications, not on server upkeep.
- **🌍 Go global in minutes** – Deploy in multiple regions to reduce latency and improve customer experience.

---

## 🌐 AWS Global Infrastructure

### 📍 Regions
- Multiple, physically isolated locations around the globe.
- Each region includes **Availability Zones**.

### 🧱 Availability Zones
- One or more data centers with redundant power and networking.
- Examples: `us-east-1a`, `eu-west-2b`.
- Widely used in AWS CLI and SDKs.

### 🌆 AWS Local Region & Local Zones
- **Local Region**: A smaller AWS region designed to complement an existing one.
- **Local Zones**: Brings services like compute/storage closer to users in specific cities or industries.

### 🚦 Points of Presence
- Includes **Edge Locations** and **Edge Caches** used by **CloudFront** and **Lambda@Edge** to serve content faster to users.

---

## 🔐 AWS Security and Compliance

Security is AWS’s top priority. They provide:
- Multiple layers of protection: hardware, software, network, and data
- Regular audits and compliance certifications
- Shared responsibility model:
    - **AWS secures the cloud infrastructure**
    - **You secure what you build in the cloud**

You also gain access to AWS's best practices, tools, and reports to support your security requirements.

---


## 💲 AWS Pricing

### Key Cost Drivers:
- **Compute**
- **Storage**
- **Data Transfer**

### Pricing Models:
- **Pay-as-you-go**: Flexible and usage-based
- **Reserved Instances**: Save up to 75% (Available for EC2, RDS, EMR)
    - **All Upfront**
    - **Partial Upfront**
    - **No Upfront**

### Additional Options:
- **Volume-based discounts** (e.g., for Amazon S3)
- **AWS Free Tier** for 12 months: [aws.amazon.com/free](https://aws.amazon.com/free/)
- **Pricing Calculator**: [AWS Pricing Calculator](https://calculator.aws.amazon.com/)

---


## 🏛️ AWS Well-Architected Framework

A set of guidelines to build reliable, secure, efficient, and cost-effective cloud systems.

### Core Pillars:
1. **Operational Excellence**
    - Improve processes and support development
    - 📌 *Principles*: Operations as code, small reversible changes, anticipate failure

2. **Security**
    - Protect data, systems, and assets
    - 📌 *Principles*: Identity foundation, traceability, layered security, encryption

3. **Reliability**
    - The ability of a workload to perform correctly and consistently throughout its lifecycle.
    - 📌 *Design Principles*:
        - Automatically recover from failure
        - Test recovery procedures
        - Scale horizontally to increase aggregate workload availability
        - Stop guessing capacity
        - Manage change in automation

4. **Performance Efficiency**
    - Use computing resources efficiently as demand changes and technology evolves.
    - 📌 *Design Principles*:
        - Democratize advanced technologies
        - Go global in minutes
        - Use serverless architectures
        - Experiment more often
        - Consider mechanical sympathy

5. **Cost Optimization**
    - Deliver business value at the lowest cost.
    - 📌 *Design Principles*:
        - Implement Cloud Financial Management
        - Adopt a consumption model
        - Measure overall efficiency
        - Stop spending money on undifferentiated heavy lifting
        - Analyze and attribute expenditure

6. **Sustainability**
    - Maximize resource efficiency and reduce environmental impact.
    - 📌 *Design Principles*:
        - Understand your impact
        - Establish sustainability goals
        - Maximize utilization
        - Adopt new efficient hardware/software
        - Use managed services
        - Reduce downstream impact of cloud workloads

---

## 🛠️ Best Practices When Architecting in the Cloud

### 🔄 Focus on Scalability

- **Horizontal Scaling**: Add more resources (e.g., EC2 instances, read replicas).
- **Vertical Scaling**: Upgrade an individual instance type (e.g., t3 → m5).

### 🧹 Disposable Resources Instead of Fixed Servers

- **Instantiating Compute Resources**: Automate with bootstrapping, Docker, AMIs.
- **Infrastructure as Code**: Use tools like Terraform, CloudFormation, and CDK.

### ⚙️ Use Automation

- **Serverless Deployment**: AWS manages provisioning, patching, and scaling.
- **Infrastructure Management & Monitoring**:
    - Load balancing, autoscaling, alarms/events

---

## 🔗 Implement Loose Coupling

- **Well-Defined Interfaces**: Use RESTful APIs or technology-agnostic endpoints.
- **Service Discovery**: Microservices should auto-detect and communicate dynamically.
- **Asynchronous Integration**: Use message queues like SQS/SNS for loose integration.
- **Distributed Systems Best Practices**: Build for graceful failure.

---

## 🧰 Services, Not Servers

- **Managed Services**: Leverage AWS services for databases, ML, search, etc.
- **Serverless Architectures**: Minimize server ops overhead with Lambda, API Gateway.

---

## 🧮 Appropriate Use of Databases

- Choose based on workload:
    - **Relational DBs**: SQL, joins, integrity (e.g., RDS, Aurora)
    - **NoSQL**: Schemaless, scalable (e.g., DynamoDB)
    - **Data Warehouses**: For analytics (e.g., Redshift)
    - **Graph DBs**: For relationships (e.g., Neptune)

---

## 📊 Managing Large Data Volumes

- **Data Lake**: Centralized repository for diverse data.
- **Search**: Search services for structured/unstructured data (OpenSearch, CloudSearch).

---

## 🧱 Removing Single Points of Failure

- **Redundancy**:
    - **Standby**: Failover-based recovery
    - **Active**: Load distributed across instances
    - **Quorum**: Combine sync and async replication

- **Durable Storage**:
    - **Synchronous replication**
    - **Asynchronous replication**
    - **Quorum-based replication**

- **Resilience**:
    - Use **multi-AZ** and **shuffle sharding**
    - Use **automated failover detection**

---

## 💸 Optimize for Cost

- **Right Sizing**: Match instance size to workload
- **Elasticity**: Scale up/down based on demand
- **Purchase Options**: Reserved Instances, Spot Instances, Savings Plans

---

## ⚡ Caching

- **App Data Caching**: Use fast in-memory caches (e.g., ElastiCache)
- **Edge Caching**: Use CDN (e.g., CloudFront) for global performance

---

## 🔐 Security

- **Defense in Depth**: Secure across multiple layers (app, data, network)
- **Shared Responsibility Model**:
    - AWS secures the infrastructure
    - You secure your applications
- **Least Privilege**: Limit access based on user role
- **Security as Code**: Use templates for firewall/NACL/hardening
- **Real-Time Auditing**: Monitor and log with CloudWatch/CloudTrail

---

---

## 🆘 Disaster Recovery in AWS

Disaster recovery ensures business continuity by restoring services after a failure or disruption. AWS offers flexible, cost-efficient strategies to meet different **RTO** and **RPO** goals:

### 🕒 Key Metrics

- **RTO (Recovery Time Objective)**: Time to restore business operations.
- **RPO (Recovery Point Objective)**: Acceptable data loss duration.

---

### 🔁 Disaster Recovery Methods

| Method           | Description                                                                                          | Cost      | RTO   | RPO   |
|------------------|------------------------------------------------------------------------------------------------------|-----------|-------|-------|
| **Backup and Restore** | Regularly back up data to secure, durable storage. Restore data when needed.                   | 💰 Low    | ⏱️ High | 🕓 Varies |
| **Pilot Light**        | Keep core systems (e.g., database mirrors) running and updated.                              | 💰 Medium | ⏱️ Moderate | 🕓 Low |
| **Warm Standby**       | Smaller, live environment that mirrors production, kept ready for quick failover.            | 💰 Higher | ⏱️ Low | 🕓 Low |
| **Multi-Site**         | Fully replicated infrastructure in active-active setup. Redirect traffic upon failure.        | 💰 Highest | ⏱️ Very Low | 🕓 Very Low |

---

### 🚀 Benefits of AWS for DR

- Flexible architecture for your desired RTO/RPO goals.
- Global infrastructure with multiple **regions** and **availability zones**.
- Eliminate hardware procurement worries.
- Pay only for what you use.

---

### 🔧 AWS Elastic Disaster Recovery

AWS also provides a managed tool: **[AWS Elastic Disaster Recovery](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)**

- Automates disaster recovery setup and failover.
- Ideal for minimizing downtime and data loss.
- Supports workloads across AWS and on-prem environments.

---
