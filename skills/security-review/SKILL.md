---
name: security-review
description: Assess a software change or design for security risks and control gaps; use for authentication, authorization, sensitive data, external exposure, dependencies, and other security-relevant work.
---

# Security Review

## Purpose

Identify security-relevant risks in a proposed design or implementation and recommend proportionate, actionable mitigations.

## When to use

Use when work affects identity, authentication, authorization, sessions, tokens, secrets, user input, sensitive data, network exposure, dependencies, file handling, administrative operations, or security controls. Also use when a task's impact warrants a focused threat assessment.

## Prerequisites and context

- Exact design, requirements, or code revision being assessed.
- Relevant project architecture and trust boundaries.
- Data classification and deployment context where known.
- Apply the shared risk tiers and minimum review controls in the [Onboarding Standard §14](../../docs/AI-DEVELOPMENT-TEAM-ONBOARDING.md#14-risk-based-review-requirements) when determining review depth and escalation.
- Read-only access to relevant configuration, code, and evidence.

Do not request or reproduce live secrets, private keys, passwords, or personal data in reports.

## Procedure

1. **Define scope.** Identify assets, actors, entry points, affected components, environment, and exact revision.
2. **Map trust boundaries.** Trace identities, privileges, tokens, network boundaries, service-to-service calls, and data flows.
3. **Identify threats.** Consider unauthorized access, privilege escalation, spoofing, tampering, information disclosure, denial of service, and relevant abuse cases.
4. **Classify review risk.** Use the Routine/low-risk, Moderate-risk, and High-risk tiers and required controls in the [Onboarding Standard §14](../../docs/AI-DEVELOPMENT-TEAM-ONBOARDING.md#14-risk-based-review-requirements). Escalate high-risk work for the required architecture review, independent evidence-based QA, human review/approval, staging verification, and recovery planning.
5. **Inspect controls.** Review authentication, authorization, input validation, output encoding, session/token handling, secret management, transport security, logging, rate limiting, and dependency/configuration risks as applicable.
6. **Check failure paths.** Examine defaults, missing configuration, expired or invalid credentials, partial failures, error messages, and fail-open/fail-closed behavior.
7. **Assess impact and likelihood.** Explain realistic attack preconditions and consequences. Avoid unsupported claims that a system is secure or exploitable.
8. **Recommend mitigations.** Prioritize concrete changes and identify defense-in-depth measures. Separate required fixes from optional hardening.
9. **Verify where safe.** Use code inspection, tests, configuration review, or authorized non-destructive checks. Do not perform intrusive testing without explicit authorization and a defined scope.
10. **Report and escalate.** Communicate urgent findings through the project's approved security process and avoid exposing exploit details or sensitive data beyond authorized recipients.

## Deliverables

Report:
- Scope, system/revision, and trust boundaries examined.
- Assets and threat scenarios considered.
- Findings classified using the shared severity scale in the [Onboarding Standard §11](../../docs/AI-DEVELOPMENT-TEAM-ONBOARDING.md#11-qa-severity-and-outcomes), with affected component, preconditions, impact, evidence, and mitigation.
- Controls that were verified and checks performed.
- Areas not assessed and assumptions.
- Recommended remediation priority and escalation needs.

## Verification

- Findings are tied to the actual design, code, or configuration.
- Severity reflects impact and realistic exploitability.
- Recommendations address root causes and do not rely solely on obscurity.
- Sensitive values are not copied into the report.
- Unverified controls are clearly marked as unverified.

## Boundaries and escalation

- Perform only authorized, non-destructive assessment unless explicit scope permits more.
- Never expose, retrieve, or test against third-party secrets or systems without authorization.
- Do not claim certification, compliance, or comprehensive security from a focused review.
- Escalate suspected active compromise, exposed credentials, critical vulnerabilities, or privacy-impacting data exposure promptly to the responsible human.
