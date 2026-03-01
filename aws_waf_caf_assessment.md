# AWS Well-Architected Framework & Cloud Adoption Framework Assessment

## Two-Tier Web Application Migration to AWS

**Prepared by:** [Student Name]
**Date:** March 1, 2026
**Course:** Cloud Engineering

---

> **Consolidated Report** (`aws_waf_caf_assessment.md`) — required per submission criteria.
> Each deliverable is also available as a standalone file:
>
> | Deliverable                    | File                                             |
> | ------------------------------ | ------------------------------------------------ |
> | Approach & repo overview       | [README.md](README.md)                           |
> | Task 2 — WAF Assessment Table  | [waf_assessment.md](waf_assessment.md)           |
> | Task 3 — CAF Readiness Summary | [caf_readiness.md](caf_readiness.md)             |
> | Task 4 — Architecture Design   | [architecture_design.md](architecture_design.md) |
> | Reflection                     | [reflection.md](reflection.md)                   |

---

## Approach

This assessment evaluates the migration of a two-tier web application from on-premises infrastructure to AWS. The analysis is structured in four tasks:

1. Review of the existing on-premises architecture identifying components, risks, and weaknesses.
2. Evaluation of the workload against the five pillars of the AWS Well-Architected Framework (WAF), producing a structured assessment table.
3. Organisational readiness analysis using the six perspectives of the AWS Cloud Adoption Framework (CAF), with key actions identified per perspective.
4. Design of an improved target architecture that addresses all five WAF pillars and incorporates CAF recommendations.

The methodology follows a risk-first approach: weaknesses in the current state are catalogued, mapped to the appropriate framework dimension, and resolved through specific AWS services and architectural patterns. All recommendations are grounded in AWS best practices and the AWS Well-Architected Framework documentation.

The architecture described in Task 4 is achievable using AWS-native managed services and requires no third-party tooling. Infrastructure-as-code (IaC) is the assumed provisioning mechanism throughout.

---

## Task 1 — Existing Architecture Review

### 1.1 Workload Components

The existing on-premises environment consists of the following components:

## Frontend / Application Tier

- Single Linux server running Apache HTTP Server with a Node.js application runtime.
- Serves both static content and dynamic application logic from a single host.
- No load balancing or horizontal redundancy exists.
- Deployed in a single physical location; no geographic or availability zone-level redundancy.

## Backend / Database Tier

- Single Linux server running MySQL (or PostgreSQL).
- Database and application tiers share a flat network segment with no enforced isolation.
- No managed backup service; snapshots are absent or performed manually and inconsistently.

## Networking

- Single flat network with no subnet segregation between application and database tiers.
- Firewall rules are statically defined by IP address.
- No reverse proxy, CDN, or DDoS mitigation layer at the network edge.

## Access & Identity

- SSH access to both servers is direct, with no bastion host or jump server.
- Root or shared OS credentials are in use; no IAM-equivalent identity framework exists.
- No secrets management solution; database credentials stored in application configuration files.

## Storage

- All data resides on locally attached server disks.
- No external object storage, no offsite backups.

## Observability

- Monitoring is limited to local OS-level log files.
- No centralised log aggregation, no alerting, no dashboards.

## Deployment

- All deployments are performed manually via SSH.
- No CI/CD pipeline, no version-controlled infrastructure configuration, no rollback mechanism.

### 1.2 Risks and Weaknesses

The following risks are present in the current architecture:

## Reliability

- **R1.** Single point of failure — no redundancy at any tier. Any hardware failure causes a complete application outage.
- **R2.** Single availability zone equivalent — the application has no geographic or fault-domain distribution.
- **R3.** No backup or disaster recovery strategy — data loss is guaranteed in the event of disk failure. RTO and RPO are undefined.

## Security

- **S1.** Overly permissive firewall rules — port 22 (SSH) and potentially other ports are open to broad or unrestricted source ranges.
- **S2.** No encryption at rest or in transit — database data and application traffic are unprotected.
- **S3.** Shared/root credentials — no principle of least privilege is enforced for human or application identities.

## Operational Excellence

- **O1.** No monitoring, logging, or alerting — failures are discovered reactively, typically by end users.
- **O2.** Manual deployment process — no audit trail, no rollback capability, high error risk on every change.

## Performance Efficiency

- **P1.** No autoscaling — the application cannot respond to traffic spikes; peak load is handled by over-provisioning static capacity.

## Cost Optimization

- **C1.** On-premises capacity is always-on regardless of actual utilisation, resulting in consistent idle spend and capital lock-in.

---

## Task 2 — AWS Well-Architected Framework Assessment

| Pillar                     | Observation                                                                                                                                                                                                                                                                                                                                                             | Improvement Recommendation                                                                                                                                                                                                                                                                                                                                           | Supporting AWS Service(s)                                                                                                                                                                      |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Operational Excellence** | **Strength:** The two-tier model is architecturally simple and well-understood, reducing cognitive overhead for the operations team during migration. **Weakness:** No operational procedures are codified. Deployments are manual, irreversible without manual rollback, and unlogged. There is no centralised visibility into application health.                     | Adopt infrastructure-as-code to make all environment changes versioned, repeatable, and auditable. Implement a CI/CD pipeline to replace manual SSH deployments. Centralise all application and infrastructure logs. Define runbooks for common failure scenarios to enable consistent, repeatable incident response.                                                | AWS CodePipeline + CodeDeploy (CI/CD); AWS CloudFormation / CDK (IaC); Amazon CloudWatch Logs (centralised logging); AWS Systems Manager OpsCenter (runbooks and operational automation)       |
| **Security**               | **Strength:** The two-tier architecture establishes a natural boundary between the application and database layers, providing a structural foundation for network segmentation. **Weakness:** Security groups are overly permissive. No encryption at rest or in transit. Root or shared credentials are in use with no access control framework in place.              | Enforce least-privilege security groups permitting only required traffic between defined tiers. Enable TLS for all traffic in transit. Replace static credentials with IAM roles assigned to compute resources. Encrypt all data at rest. Centralise secrets management with automated rotation. Implement a web application firewall at the network edge.           | AWS IAM (roles and policies); AWS Certificate Manager (TLS); AWS Secrets Manager (credential rotation); AWS KMS (encryption at rest); AWS WAF (OWASP Top 10 protection)                        |
| **Reliability**            | **Strength:** The frontend application tier is stateless, meaning horizontal scaling and multi-AZ distribution can be achieved with minimal application refactoring. **Weakness:** The architecture is a single point of failure at every layer. Any AZ- or hardware-level failure causes a complete outage. No RTO/RPO targets are defined.                            | Deploy the workload across a minimum of two AWS Availability Zones. Place the frontend tier behind an Application Load Balancer with an Auto Scaling Group. Migrate the database to Amazon RDS in Multi-AZ configuration for automated failover. Define RTO and RPO targets. Implement automated backups with tested restoration procedures.                         | Elastic Load Balancing / ALB (traffic distribution); Amazon RDS Multi-AZ (automated failover); AWS Backup (policy-based retention); Amazon Route 53 (health-check-based DNS failover)          |
| **Performance Efficiency** | **Strength:** The decoupled two-tier architecture allows the frontend and database tiers to be independently scaled on AWS without significant application restructuring. **Weakness:** No autoscaling exists. The database is vertically constrained with no read-scaling capability. Static assets are served by the application server, increasing response latency. | Implement an EC2 Auto Scaling Group for the web tier to match compute capacity to actual demand. Offload static asset delivery to a CDN, eliminating unnecessary load on the application server. Evaluate read-heavy query patterns for ElastiCache adoption. Use AWS Compute Optimizer post-migration to rightsize all compute instances based on observed metrics. | Amazon EC2 Auto Scaling (dynamic compute scaling); Amazon CloudFront (CDN for static assets); Amazon ElastiCache (database query caching); AWS Compute Optimizer (rightsizing recommendations) |
| **Cost Optimization**      | **Strength:** Migration to AWS eliminates capital expenditure cycles for hardware refresh. Compute and storage become variable operating expenses aligned to actual workload demand. **Weakness:** Without tagging, rightsizing, or budgeting discipline, the migration risks replicating on-premises over-provisioning in the cloud with no visibility into spend.     | Implement a mandatory resource tagging policy covering environment, owner, and cost centre from day one. Enable Cost Explorer and set budget alarms immediately. After 30–60 days of production usage, evaluate Reserved Instances or Savings Plans for baseline workloads. Adopt a FinOps review cadence to continuously optimise spend.                            | AWS Cost Explorer (spend analysis); AWS Budgets (threshold alerting); AWS Trusted Advisor (cost checks); EC2 Savings Plans / Reserved Instances; AWS Cost Anomaly Detection                    |

---

## Task 3 — AWS Cloud Adoption Framework: Organisational Readiness Summary

### Perspective 1: Business

The Business perspective examines whether the organisation has aligned cloud migration objectives with measurable business outcomes and financial governance. The decision to migrate the two-tier web application to AWS indicates executive-level sponsorship and a recognised need to modernise. However, the migration currently lacks a formalised business case that quantifies expected total cost of ownership (TCO) reduction, defines return on investment (ROI) timelines, or establishes KPIs against which success can be measured.

**Readiness Assessment:** Low to medium. Leadership has initiated the migration, which signals organisational commitment. However, without documented success metrics — such as target uptime SLA, cost-per-transaction reduction, or mean time to recovery (MTTR) improvement — the business value of the migration cannot be validated post-execution. Finance and procurement teams are likely unfamiliar with the shift from capital expenditure (CapEx) hardware refresh cycles to operational expenditure (OpEx) cloud billing.

**Key Actions and Enablers:**

- Produce a formal business case documenting expected TCO savings versus the on-premises baseline, covering a 3-year horizon.
- Define quantifiable cloud success KPIs: uptime percentage, deployment frequency, infrastructure cost per user, and MTTR targets.
- Assign a named business owner accountable for migration outcomes and value realisation.
- Brief finance and procurement stakeholders on AWS billing models, Reserved Instance commitments, and Savings Plans.
- Establish a quarterly cloud value review cadence to track KPIs against baseline and adjust strategy accordingly.

---

### Perspective 2: People

The People perspective addresses workforce transformation, skills readiness, and the change management required for successful cloud adoption. The organisation's operations team possesses deep expertise in Linux system administration, on-premises networking, and traditional deployment practices. These skills are transferable but require supplementation with cloud-native competencies — specifically AWS IAM, managed services, cloud-native observability, and infrastructure-as-code.

**Readiness Assessment:** Low. The team is technically competent in on-premises operations but lacks the AWS-specific knowledge required to design, deploy, and operate the target architecture safely. Without structured training, the migration carries a high risk of misconfiguration — particularly in the security and governance domains, where cloud-native concepts differ significantly from on-premises equivalents. No formal cloud training programme, certification pathway, or internal knowledge transfer process is currently in place.

**Key Actions and Enablers:**

- Enrol key team members in AWS Cloud Practitioner (foundational) and AWS Solutions Architect Associate (technical) certification paths prior to migration execution.
- Designate a Cloud Champion within the existing team to lead internal knowledge transfer and serve as the first point of contact for cloud-native operational questions.
- Define an updated RACI model that incorporates new cloud roles: cloud infrastructure engineer, security engineer, and FinOps analyst.
- Develop a change management plan that addresses team concerns about new tooling, altered workflows, and evolving job responsibilities.
- Leverage AWS Skill Builder and AWS Training and Certification for self-paced and instructor-led learning programmes.

---

### Perspective 3: Governance

The Governance perspective ensures the organisation has the policies, automated guardrails, risk management frameworks, and compliance controls required to operate responsibly in the cloud. On-premises governance is typically enforced through physical access controls and manually configured firewall policies. Cloud environments require governance to be codified as policy, automated through enforcement mechanisms, and continuously audited.

**Readiness Assessment:** Low. No cloud-specific governance policies are in place. There is no resource tagging standard, no approved AWS region list, no account structure, and no process for reviewing or approving new cloud service adoption or spend. Existing regulatory obligations — such as data residency requirements, audit logging mandates, or industry compliance frameworks (GDPR, PCI-DSS) — have not been assessed against AWS compliance programmes or mapped to specific AWS controls.

**Key Actions and Enablers:**

- Define and document a cloud governance framework covering: mandatory tagging policy (environment, owner, cost centre, project), naming conventions, and approved AWS regions.
- Implement AWS Organizations with Service Control Policies (SCPs) enforced at the organisational unit level to prevent non-compliant resource creation.
- Enable AWS Config and AWS CloudTrail organisation-wide from day one to establish a continuous compliance and audit trail baseline.
- Establish a Cloud Centre of Excellence (CCoE) or governance committee with representatives from IT, security, finance, and legal.
- Map all applicable regulatory requirements to AWS compliance programmes and document the shared responsibility model for each.

---

### Perspective 4: Platform

The Platform perspective covers the technical foundation of the cloud environment: landing zone architecture, infrastructure provisioning approach, account strategy, network design, and the selection of managed versus self-managed services. The organisation's existing two-tier application is an architecturally simple, well-understood pattern that translates directly to AWS managed service equivalents without requiring fundamental re-architecture.

**Readiness Assessment:** Medium. The team understands the application's components and dependencies, which reduces migration risk at the application layer. However, no cloud-native deployment methodology exists. Infrastructure provisioning is entirely manual, there is no IaC practice, no AWS account structure is defined, and no landing zone design has been produced. The decision-making framework for selecting managed AWS services over self-managed equivalents is absent.

**Key Actions and Enablers:**

- Establish a Well-Architected landing zone using AWS Control Tower to create a standardised, secure multi-account environment.
- Adopt AWS CloudFormation or AWS CDK as the mandatory IaC framework; all infrastructure must be provisioned through version-controlled templates from the outset.
- Define a multi-account strategy separating development, staging, and production environments into distinct AWS accounts.
- Adopt managed services wherever equivalent self-managed alternatives exist: Amazon RDS in place of self-hosted MySQL; ALB in place of self-managed Nginx; Amazon S3 in place of local disk for static assets.
- Design the VPC with a three-tier subnet model (public, private, isolated) across a minimum of two Availability Zones.

---

### Perspective 5: Security

The Security perspective ensures the organisation protects its data, workloads, and cloud environment by aligning to AWS security best practices, applying the principle of least privilege, and meeting all applicable compliance obligations. The current on-premises environment presents a critical security posture: permissive firewall rules, unencrypted data, shared credentials, and no security monitoring. These gaps must be remediated before or during migration — they must not be replicated in AWS.

**Readiness Assessment:** Low. The security posture of the existing environment is insufficient for a cloud deployment. Without a structured remediation plan, migrating the current state to AWS would expose the organisation to a materially higher threat surface due to the internet-accessible nature of cloud infrastructure. The absent encryption strategy, identity framework, and monitoring capability represent three distinct high-severity risk areas requiring immediate action.

**Key Actions and Enablers:**

- Enforce IAM least-privilege as a non-negotiable baseline: disable root account for day-to-day operations, enforce MFA on all human identities, and assign IAM roles (not static keys) to all compute resources.
- Enable Amazon GuardDuty across all accounts from day one for continuous threat detection and anomaly alerting.
- Implement AWS Secrets Manager for all application credentials with automated rotation policies enabled.
- Define a data classification policy and enforce encryption at rest (AWS KMS) for all RDS, EBS, and S3 resources, and TLS in transit via AWS Certificate Manager.
- Onboard AWS Security Hub and enable the AWS Foundational Security Best Practices standard to provide a continuous, aggregated security posture score.

---

### Perspective 6: Operations

The Operations perspective addresses how the organisation will manage, monitor, and continuously improve the cloud environment in production. This encompasses monitoring and alerting, incident response, change management, patch management, and backup procedures. The current state is entirely reactive: there is no centralised monitoring, no defined incident response process, and no operational runbooks beyond individual team member knowledge.

**Readiness Assessment:** Low to medium. The team has the foundational operational discipline required (awareness of the application's behaviour, familiarity with the technology stack), but all operational processes are manual and undocumented. The absence of a defined on-call process, incident severity classification, and runbook library represents significant operational risk for a production workload in AWS.

**Key Actions and Enablers:**

- Deploy Amazon CloudWatch dashboards, metrics, and alarms covering key signals for all tiers: EC2 CPU/memory/disk, RDS connections and latency, ALB request count and 5xx error rate, and application-level custom metrics.
- Define an incident response runbook library and integrate with AWS Incident Manager for automated escalation and response coordination.
- Implement AWS Systems Manager for all patch management, software inventory, and server access (Session Manager replaces direct SSH).
- Mandate that all infrastructure changes are executed through version-controlled IaC pull requests with peer review; no manual console changes in production.
- Configure AWS Backup with defined policies covering RTO and RPO targets for both the RDS database and any critical S3 data, with quarterly restoration tests.

---

## Task 4 — Improved AWS Architecture Design

### 4.1 Architecture Overview

The improved architecture transforms the single-server, single-location deployment into a resilient, secure, scalable, and observable AWS-native environment. The design addresses all five WAF pillars and incorporates the platform and security recommendations from the CAF assessment.

All infrastructure is provisioned via AWS CloudFormation or AWS CDK. No manual console changes are permitted in production environments.

### 4.2 Network Layer

A dedicated Virtual Private Cloud (VPC) is deployed across two AWS Availability Zones (AZ-1 and AZ-2) with a three-tier subnet architecture:

## Public Subnets (AZ-1 and AZ-2)

- Houses the Application Load Balancer (ALB) only.
- Internet Gateway attached for inbound traffic routing.
- NAT Gateway deployed in each public subnet for outbound internet access from private tiers (software updates, AWS API calls).

## Private Subnets (AZ-1 and AZ-2)

- Houses the EC2 web/application tier within an Auto Scaling Group.
- No direct inbound route from the internet; all traffic arrives via the ALB.
- Outbound internet access via NAT Gateway.

## Isolated / Database Subnets (AZ-1 and AZ-2)

- Houses the Amazon RDS instance; no internet route in or out.
- Security group permits inbound traffic only from the application tier security group on the database port (3306/5432).

VPC Flow Logs are enabled and shipped to Amazon CloudWatch Logs for network traffic analysis and security investigation.

Security groups are constructed on a least-privilege basis:

- **ALB SG:** inbound 443 from `0.0.0.0/0` only.
- **App tier SG:** inbound from ALB SG on application port only.
- **DB SG:** inbound from App tier SG on DB port only.
- No port 22 open to any CIDR; all server access via SSM Session Manager.

### 4.3 Edge / Ingress Layer

## Amazon CloudFront

- CDN distribution sits in front of the ALB.
- Caches and serves static assets (images, CSS, JS) from AWS edge locations, reducing origin load and improving global latency.
- Origin access restricted to CloudFront only via ALB security group rules and custom header validation.

## AWS WAF (Web Application Firewall)

- Attached to the CloudFront distribution.
- AWS Managed Rule Groups enabled covering OWASP Top 10, known bad inputs, and Amazon IP reputation lists.
- Rate-based rules configured to mitigate layer-7 DDoS attempts.

## AWS Certificate Manager (ACM)

- TLS certificates provisioned and managed by ACM at no additional cost.
- TLS termination occurs at CloudFront and the ALB.
- End-to-end HTTPS enforced; HTTP redirected to HTTPS at CloudFront.

### 4.4 Compute Tier

## Application Load Balancer (ALB)

- Deployed in public subnets across AZ-1 and AZ-2.
- Routes HTTP/HTTPS traffic to the Auto Scaling Group target group.
- Health checks configured with appropriate thresholds to remove unhealthy instances from rotation automatically.

## EC2 Auto Scaling Group

- Web/application instances deployed in private subnets across both AZs.
- Scaling policies defined: minimum 2 instances (one per AZ), desired 2, maximum defined by load testing benchmarks.
- Dynamic scaling policy tied to `ALB RequestCountPerTarget` metric to scale out under load and in during low-traffic periods.

## Amazon Machine Images (AMI)

- Immutable AMIs baked via AWS CodeBuild as part of the deployment pipeline; no in-place updates to running instances.
- Blue/green or rolling deployments managed by AWS CodeDeploy.

## AWS Systems Manager Session Manager

- All server access is performed via SSM Session Manager.
- Port 22 is not open on any security group.
- All session activity is logged to Amazon CloudWatch Logs and Amazon S3 for audit purposes.

### 4.5 Database Tier

## Amazon RDS (MySQL or PostgreSQL) — Multi-AZ

- Primary instance in AZ-1; standby replica in AZ-2.
- Automated failover: RDS promotes the standby synchronously without manual intervention (typical failover < 60 seconds).
- Automated backups enabled with 7-day retention minimum.
- Read Replica (optional): deployed for read-heavy query workloads to offload read traffic from the primary.

## AWS Backup

- Centralised backup policy applied to RDS and EBS volumes.
- Backup schedule: daily snapshots, weekly long-term retention.
- Restoration tests scheduled quarterly; results documented.

## Amazon ElastiCache (Optional / Phase 2)

- Redis or Memcached cluster deployable in private subnets to cache frequent database queries, reducing RDS load and improving application response time.

### 4.6 Storage

## Amazon S3

- Dedicated bucket for static web assets; served via CloudFront origin access control.
- Dedicated bucket for deployment artefacts (build outputs).
- Dedicated bucket for access logs (ALB, CloudFront, VPC Flow Logs, SSM session logs).
- All buckets: S3 Block Public Access enabled, default encryption with KMS, versioning enabled on artefact and log buckets, and lifecycle policies for cost-managed log retention.

### 4.7 Security Controls

## AWS IAM

- EC2 instances carry instance profiles (IAM roles); no static access keys on any server.
- Roles follow least-privilege: only the permissions required for the specific workload function are granted.
- Human identities use IAM Identity Center with MFA enforced.

## AWS Secrets Manager

- RDS master credentials stored in Secrets Manager.
- Automatic rotation configured (30-day rotation schedule).
- Application retrieves credentials at runtime via the Secrets Manager API; no credentials present in environment variables or configuration files.

## AWS KMS

- Customer-managed KMS keys used for encryption of: RDS storage, EBS volumes, S3 buckets, and CloudWatch Logs.
- Key rotation enabled on all CMKs.

## Amazon GuardDuty

- Enabled across all accounts from day one.
- Threat findings routed to AWS Security Hub and Amazon EventBridge for automated alerting and response triggering.

## AWS Security Hub

- AWS Foundational Security Best Practices standard enabled.
- Aggregates findings from GuardDuty, AWS Config, and Inspector.
- Security score tracked weekly; critical findings trigger immediate incident response.

## AWS Config

- Conformance packs deployed for security and compliance baseline.
- Config rules evaluate resources continuously against defined policies (e.g., S3 bucket public access, RDS encryption, MFA on IAM users).

### 4.8 Observability

## Amazon CloudWatch

- Dashboards: ALB (request count, latency, 5xx rate), EC2 (CPU, memory via CloudWatch Agent, disk), RDS (connections, read/write IOPS, free storage).
- Log Groups: application logs, system logs, SSM session logs, VPC Flow Logs, CloudTrail.
- Alarms: SNS notifications for critical thresholds (e.g., RDS storage < 20%, EC2 CPU > 80% sustained for 5 minutes, 5xx rate > 1%).

## AWS CloudTrail

- Organisation-wide trail enabled logging all management and data events to a dedicated, immutable S3 bucket.
- CloudTrail Insights enabled to detect unusual API activity.

## AWS X-Ray (Phase 2)

- Distributed tracing deployed to identify latency bottlenecks across the application and database tiers.

### 4.9 CI/CD and Operations

## AWS CodePipeline + CodeBuild + CodeDeploy

- Source: GitHub repository triggers pipeline on push to `main` branch.
- Build: CodeBuild compiles application, runs tests, builds AMI.
- Deploy: CodeDeploy performs blue/green deployment to the Auto Scaling Group with automatic rollback on health check failure.

## AWS CloudFormation / CDK

- All infrastructure defined as code in version-controlled stacks.
- Separate stacks for: networking, security, compute, database, observability, and CI/CD.
- Change sets reviewed and approved before execution in production.

## AWS Trusted Advisor

- Weekly review of Trusted Advisor recommendations across security, fault tolerance, performance, and cost optimisation categories.

### 4.10 WAF Pillar Coverage Summary

| WAF Pillar                 | Design Elements Addressing This Pillar                                                                                                  |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Operational Excellence** | CodePipeline CI/CD, CloudFormation IaC, CloudWatch Logs, Systems Manager runbooks                                                       |
| **Security**               | Private/isolated subnets, IAM roles, AWS WAF, KMS encryption, Secrets Manager, GuardDuty, Security Hub, Config                          |
| **Reliability**            | Multi-AZ ALB, EC2 Auto Scaling across 2 AZs, RDS Multi-AZ, AWS Backup, Route 53 failover                                                |
| **Performance Efficiency** | CloudFront CDN, EC2 Auto Scaling, ElastiCache (phase 2), Compute Optimizer rightsizing                                                  |
| **Cost Optimization**      | Mandatory tagging policy, AWS Budgets alarms, Cost Explorer, Savings Plans post-baseline, S3 lifecycle policies, Cost Anomaly Detection |

---

## Reflection

This assessment reinforced a principle that separates proficient cloud practitioners from genuinely capable cloud architects: the ability to reason about a workload holistically rather than by component. Evaluating the two-tier application against all five WAF pillars simultaneously exposed interdependencies that would be missed in siloed analysis. A Multi-AZ deployment resolves a Reliability risk but introduces Cost Optimization considerations; that tension must be resolved through deliberate design, not deferred.

Applying the Cloud Adoption Framework surfaced the non-technical dimension of migration risk. The organisation's most acute vulnerabilities are not architectural — they are in the People and Governance perspectives. A technically flawless architecture degrades without trained operators and enforceable policy guardrails.

The most instructive exercise was tracing a specific risk from identification through to resolution: overly permissive credentials → Secrets Manager + IAM roles → enforced by SCPs and Config rules. This end-to-end remediation pattern is the core reasoning skill of cloud architecture work, and this lab provides a structured method for developing it systematically.
