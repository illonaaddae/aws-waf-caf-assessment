# Reflection

## Brief Reflection on Learning — AWS WAF & CAF Lab

**Date:** March 1, 2026

This lab demonstrated that effective cloud architecture is an exercise in structured reasoning, not service selection. The Well-Architected Framework provided a disciplined lens: evaluating the same two-tier application through five distinct pillars surfaced risks that a single-axis review would miss — the same Multi-AZ design decision that resolves a Reliability gap simultaneously introduces a Cost Optimization consideration that must be justified against the workload's availability requirements.

The Cloud Adoption Framework reframed the migration as an organisational programme rather than a technical project. The People and Governance perspectives presented the most acute readiness gaps — an architecturally correct deployment operated by an undertrained team without enforceable policy guardrails will degrade toward the same insecure, unobservable state it replaced.

The most durable learning is the traceability discipline: every architectural decision in Task 4 can be traced backward to a specific risk in Task 1. That closed loop — from identified weakness to resolved design element — is the core reasoning pattern of cloud architecture work.
