# AWS Well-Architected Framework & Cloud Adoption Framework Assessment

## Two-Tier Web Application Migration to AWS

**Author:** [Student Name]
**Date:** March 1, 2026
**Course:** Cloud Engineering

---

## Repository Structure

```md
.
├── README.md ← This file — approach and repo overview
├── waf_assessment.md ← Task 2: WAF five-pillar assessment table
├── caf_readiness.md ← Task 3: CAF six-perspective readiness summary
├── architecture_design.md ← Task 4: Improved architecture description
└── reflection.md ← Brief reflection on learnings
```

The main combined report (`aws_waf_caf_assessment.md`) is also included as a single-document reference for reviewers who prefer consolidated reading.

## Approach

This submission evaluates the migration of a two-tier on-premises web application to AWS, structured against two AWS frameworks:

- **AWS Well-Architected Framework (WAF)** — five pillars examined against the current state to surface specific architectural weaknesses, validate existing strengths, and prescribe service-level improvements.
- **AWS Cloud Adoption Framework (CAF)** — six perspectives applied to assess organisational readiness, identifying gaps and the enabling actions required before and during migration execution.

### Methodology

The analysis follows a **risk-first, resolution-mapped** approach:

1. The existing architecture is catalogued in full — components, access patterns, storage, observability, and deployment practices.
2. Each identified risk is tagged to the WAF pillar and CAF perspective it violates.
3. Every weakness is resolved with a specific AWS service or architectural pattern — not a category recommendation.
4. The improved architecture in Task 4 is designed so that every component can be traced back to a WAF pillar it satisfies.

This traceability — from risk through recommendation to architecture — is intentional. It reflects how cloud architects reason about workloads: not as a checklist exercise, but as a discipline of closing identified gaps with justifiable, measurable design decisions.

### Assumptions

The on-premises baseline assumed throughout is:

| Component      | Current State                            |
| -------------- | ---------------------------------------- |
| Web/App tier   | Single Linux server, Apache + Node.js    |
| Database tier  | Single Linux server, MySQL or PostgreSQL |
| Load balancing | None                                     |
| Redundancy     | Single physical location, no failover    |
| Backups        | None (or ad-hoc manual)                  |
| Firewall       | Static IP rules, some ports broadly open |
| Monitoring     | Local OS logs only                       |
| Deployments    | Manual via SSH, no version control       |

All AWS service recommendations are achievable without third-party tooling. Infrastructure-as-code is the assumed provisioning mechanism for the target architecture.
