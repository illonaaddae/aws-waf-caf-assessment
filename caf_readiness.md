# CAF Readiness Summary — Six-Perspective Analysis

## Task 3: AWS Cloud Adoption Framework Organisational Readiness

**Organisation:** [Organisation Name]
**Workload:** Two-Tier Web Application Migration to AWS
**Assessment Date:** March 1, 2026

## Framework Overview

The AWS Cloud Adoption Framework (CAF) structures cloud adoption readiness across six perspectives. Each perspective examines a distinct organisational dimension that must be addressed for a migration to succeed sustainably — not merely technically, but operationally, commercially, and culturally.

This assessment evaluates the organisation's current readiness against each perspective, assigns a readiness rating, and identifies the specific enabling actions required before or during migration.

## Perspective 1: Business

The Business perspective examines whether cloud migration objectives are aligned with measurable business outcomes and whether the financial model of cloud adoption is understood across the organisation. The decision to migrate the two-tier web application to AWS confirms executive sponsorship and an acknowledged need to modernise the infrastructure layer. However, sponsorship alone does not constitute readiness — a migration without documented success criteria cannot be evaluated for value delivered.

**Readiness Assessment:** Low to medium. Leadership has initiated the programme, which establishes organisational commitment. However, no formal business case has been documented. Expected total cost of ownership (TCO) reduction versus the on-premises baseline has not been quantified. Return on investment (ROI) timelines are undefined. Success metrics — such as target uptime SLA, deployment frequency improvement, or mean time to recovery (MTTR) reduction — have not been agreed. Finance and procurement teams are unlikely to have adjusted their planning models to account for the shift from capital expenditure (CapEx) hardware cycles to operational expenditure (OpEx) consumption-based billing.

**Key Actions and Enablers:**

- Produce a formal business case quantifying expected TCO savings over a three-year horizon, benchmarked against the current on-premises cost model.
- Define quantifiable cloud success KPIs prior to migration: target uptime percentage, deployment frequency, infrastructure cost per active user, and MTTR improvement.
- Assign a named business owner accountable for migration outcomes and value realisation — distinct from the technical delivery lead.
- Brief finance and procurement stakeholders on AWS billing mechanics, including the implications of Reserved Instance commitments and Savings Plans for budget forecasting.
- Establish a quarterly cloud value review cadence to track KPI performance against baseline and adjust investment priorities accordingly.

## Perspective 2: People

The People perspective addresses the workforce transformation, skills development, and change management required for sustainable cloud adoption. Cloud migration is not a purely technical transition — it redefines roles, workflows, and operational practices in ways that require deliberate organisational preparation. The existing operations team brings deep expertise in Linux administration, on-premises networking, and traditional deployment tooling; these foundation skills are genuinely valuable and transferable, but they require significant supplementation.

**Readiness Assessment:** Low. The team is competent in on-premises operations but lacks the AWS-specific knowledge needed to safely design, deploy, and operate the target architecture. The security and governance domains present particular risk — cloud-native IAM, VPC design, and policy-as-code concepts differ substantially from their on-premises equivalents. Attempting to operate cloud infrastructure with a traditional mindset produces systematic misconfiguration. No formal cloud training programme, certification pathway, or structured internal knowledge transfer process is currently in place. The organisation has not yet defined what cloud operations roles look like or who is accountable for them.

**Key Actions and Enablers:**

- Enrol key team members in AWS Cloud Practitioner (foundational awareness) and AWS Solutions Architect Associate (technical implementation) certification paths, completing both prior to migration execution on production workloads.
- Designate a Cloud Champion within the existing team — a technically strong individual empowered to lead internal knowledge transfer, establish new practices, and resolve cloud-native questions without external dependency.
- Define an updated RACI model that explicitly accounts for emerging cloud roles: cloud infrastructure engineer, cloud security engineer, and FinOps analyst. Assign ownership rather than assuming responsibilities will self-organise.
- Develop and communicate a change management plan that acknowledges team concerns about evolving job responsibilities, new tooling mandates, and shifting operational workflows.
- Leverage AWS Skill Builder and AWS Training and Certification catalogues for structured self-paced and instructor-led learning across all team members, not only the designated Cloud Champion.

## Perspective 3: Governance

The Governance perspective ensures the organisation has the policies, controls, risk management frameworks, and compliance mechanisms required to operate responsibly in the cloud. On-premises governance is typically enforced through physical access restrictions, manually configured firewall policies, and change advisory board processes. Cloud environments require a fundamentally different model: governance must be codified as machine-enforceable policy, deployed as code, and continuously audited through automated controls — because the speed and self-service nature of cloud provisioning renders manual governance unworkable at scale.

**Readiness Assessment:** Low. No cloud-specific governance framework exists. There is no resource tagging standard, no defined approved AWS region list, no account structure, and no process for reviewing or approving new cloud service adoption or incremental spend. Existing regulatory obligations — data residency requirements, audit logging mandates, or applicable compliance frameworks such as GDPR or PCI-DSS — have not been mapped to AWS services or assessed against the AWS Shared Responsibility Model. Without governance established before first provisioning, the environment will accumulate technical debt in the form of untagged resources, unaudited access patterns, and unreviewed cost escalations that become progressively harder to remediate.

**Key Actions and Enablers:**

- Define and publish a cloud governance framework before any AWS account is used in production. The framework must cover: mandatory tagging policy (environment, owner, cost centre, project), naming conventions, approved AWS regions, and the process for requesting new service access.
- Implement AWS Organizations with Service Control Policies (SCPs) applied at the organisational unit level to enforce guardrails programmatically — preventing non-compliant resource creation rather than detecting it after the fact.
- Enable AWS Config with managed and custom conformance packs, and AWS CloudTrail with organisation-wide logging from the first provisioned resource. These cannot be retrofitted effectively; they must be baseline from day one.
- Establish a Cloud Centre of Excellence (CCoE) or cloud governance committee with representation from IT operations, security, legal, and finance — ensuring governance decisions reflect commercial and regulatory obligations, not only technical preferences.
- Map all applicable regulatory requirements to AWS compliance programmes and document the organisation's specific obligations under the Shared Responsibility Model for each applicable standard.

## Perspective 4: Platform

The Platform perspective covers the technical foundation of the cloud environment: landing zone design, account structure, VPC architecture, infrastructure provisioning methodology, and the framework for selecting managed versus self-managed services. The two-tier application is a well-understood, architecturally simple pattern that maps cleanly to AWS managed service equivalents without requiring fundamental application re-architecture — this is a significant advantage. The migration complexity is primarily in establishing the platform foundation correctly, not in the application itself.

**Readiness Assessment:** Medium. The team has sufficient understanding of the application's components, dependencies, and traffic patterns to reduce migration risk at the application layer. However, no cloud-native provisioning methodology exists — all infrastructure changes are currently manual. No IaC practice is in place. No AWS account structure has been defined, no landing zone designed, and no VPC architecture planned. The absence of clear criteria for choosing managed services over self-managed equivalents creates risk of inadvertently self-managing services that AWS manages better natively.

**Key Actions and Enablers:**

- Establish a Well-Architected landing zone using AWS Control Tower to provision a secure, governed multi-account environment with pre-configured account baselines before any production workloads are deployed.
- Adopt AWS CloudFormation or AWS CDK as the sole infrastructure provisioning mechanism. All resources must be defined in version-controlled stacks from day one — no manual console provisioning permitted in non-sandbox environments.
- Define a multi-account strategy with separate AWS accounts for development, staging, and production. Account-level isolation limits the blast radius of misconfiguration and enforces environment parity.
- Apply a managed-services-first decision rule: adopt Amazon RDS over self-hosted MySQL, ALB over self-managed Nginx, and Amazon S3 over local disk storage for all applicable use cases. Self-managing services the cloud provider manages better is an operational liability.
- Design the VPC architecture with a three-tier subnet model — public (ALB), private (application tier), and isolated (database tier) — deployed across a minimum of two Availability Zones before any workload migration begins.

## Perspective 5: Security

The Security perspective ensures the organisation protects its data, workloads, and cloud environment by applying the AWS security best practices, enforcing least-privilege access, encrypting sensitive data, and meeting all applicable compliance obligations. The current on-premises environment presents a critical security posture across multiple dimensions: access controls are insufficiently granular, data is unprotected at rest and in transit, and no security monitoring capability exists. These conditions represent a high-risk baseline that must not be replicated in AWS — the internet-accessible nature of cloud infrastructure significantly amplifies the impact of each of these gaps.

**Readiness Assessment:** Low. The three most critical security control categories — identity and access management, encryption, and monitoring — are all absent or inadequate in the current state. Migrating without a structured security remediation plan would produce a cloud environment that is substantially more exposed than the on-premises environment it replaces. A compromised cloud account has a materially larger blast radius than a compromised on-premises server. Security controls must be designed into the architecture from the first deployment, not applied as a retrofit.

**Key Actions and Enablers:**

- Mandate IAM least-privilege as a non-negotiable baseline before any workload reaches AWS: root account usage must be disabled for all operational activities, MFA enforced on every human identity, and IAM roles (not static access keys) assigned to all compute resources.
- Enable Amazon GuardDuty across all accounts from day one. GuardDuty operates passively on existing data sources (VPC Flow Logs, CloudTrail, DNS logs) and imposes no performance overhead — there is no operational cost to enabling it immediately.
- Implement AWS Secrets Manager for all application credentials — database passwords, API keys, third-party tokens — with automatic rotation policies enabled and rotation tested prior to production deployment.
- Define a data classification policy and enforce encryption as the default: AWS KMS customer-managed keys for RDS storage, EBS volumes, S3 buckets, and CloudWatch Logs; TLS via AWS Certificate Manager for all data in transit.
- Onboard AWS Security Hub and activate the AWS Foundational Security Best Practices standard to provide a continuously updated, aggregated security posture score with prioritised remediation guidance.

## Perspective 6: Operations

The Operations perspective addresses how the organisation will manage, monitor, respond to incidents within, and continuously improve the cloud environment in production. It encompasses observability, incident response, patch management, change management, and backup and recovery procedures. The current operational posture is reactive and unstructured: failures surface through end-user complaints, deployments have no rollback path, patching is performed ad-hoc, and there are no documented procedures for any operational scenario. This posture is operationally sustainable only because the current environment is small and the failure impact is contained. In AWS, with a more complex service graph, internet-accessible endpoints, and a broader attack surface, an equivalent operational posture would produce unacceptable mean times to detection and resolution.

**Readiness Assessment:** Low to medium. The team possesses the foundational operational awareness that enables effective cloud operations — they understand the application's behaviour, its failure modes, and its dependencies. However, all operational knowledge is individualised and undocumented. There is no on-call process, no incident severity classification, no runbook library, and no post-incident review practice. These deficiencies must be addressed in parallel with the technical migration — operationalising the cloud environment is not a post-go-live activity.

**Key Actions and Enablers:**

- Deploy Amazon CloudWatch dashboards, metrics, and alarms for all tiers prior to go-live: ALB request rate and 5xx error percentage, EC2 CPU and memory (via CloudWatch Agent), RDS connections, read/write IOPS, and free storage space. Define alarm thresholds and SNS notification targets before the first production deployment.
- Define an incident response runbook library covering the most likely operational events: deployment failure, database failover, SSL certificate expiry, unexpected traffic spike, and security alert from GuardDuty. Integrate with AWS Incident Manager for automated escalation and response coordination.
- Implement AWS Systems Manager as the operational platform for patch management, software inventory, and secure server access. SSM Session Manager replaces direct SSH, providing audited, credential-free access with full session logging.
- Enforce a change management discipline: all infrastructure changes must be executed through version-controlled IaC change sets with mandatory peer review. Direct console changes to production accounts must be prohibited via SCP.
- Configure AWS Backup with documented policies specifying RTO and RPO targets for the RDS database and critical S3 data. Schedule and document quarterly restoration tests — untested backups cannot be relied upon for recovery.

## Readiness Summary

| Perspective | Readiness Rating | Primary Gap                                                      |
| ----------- | ---------------- | ---------------------------------------------------------------- |
| Business    | Low–Medium       | No documented business case or success KPIs                      |
| People      | Low              | No cloud training programme or defined cloud roles               |
| Governance  | Low              | No governance framework, tagging standard, or compliance mapping |
| Platform    | Medium           | No IaC practice, no account structure, no landing zone           |
| Security    | Low              | No encryption, no least-privilege IAM, no security monitoring    |
| Operations  | Low–Medium       | No observability, no runbooks, no incident response process      |
