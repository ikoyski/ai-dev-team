---
name: independent-qa
description: Independently verify an implementation against its approved task and acceptance criteria; use after a developer submits a reviewable change.
---

# Independent QA

## Purpose

Determine whether a submitted implementation satisfies the original task and acceptance criteria, using the actual change and independent evidence.

## When to use

Use when assigned as QA for work implemented by another contributor. The QA reviewer must be different from the primary implementer.

## Prerequisites and context

- Original task and approved requirements.
- Exact PR, branch, commit, or build under review.
- Developer handoff and known test evidence.
- Relevant project instructions and a safe environment for any execution. Any execution of untrusted code, build scripts, or tests must occur in an isolated or otherwise approved non-production environment with no production credentials or privileged production access. If that cannot be assured, perform static verification only and report runtime checks as not performed.
- Read-only access is the default; obtain explicit authorization before any action that changes shared state.

## Procedure

1. **Confirm independence.** Verify that you did not implement the change and that the exact revision is identified.
2. **Read expected behavior.** Review the task, acceptance criteria, constraints, and relevant decisions before judging the implementation.
3. **Inspect the actual change.** Read the complete relevant diff and surrounding code. Do not rely solely on the developer's report.
4. **Build a verification matrix.** Map each acceptance criterion to evidence, test cases, or an explicit reason it cannot be verified.
5. **Exercise behavior.** Run appropriate tests or reproduce scenarios where practical and permitted. Include negative, boundary, and regression cases relevant to the change.
6. **Assess broader impact.** Consider integration, compatibility, security, data integrity, and operational behavior within the assigned scope.
7. **Record defects precisely.** For each finding, include severity, location, expected versus actual behavior, evidence, and reproduction steps where possible.
8. **State limitations.** Identify checks not run, unavailable environments, inaccessible components, and remaining uncertainty.
9. **Issue a result.** Return PASS, FAIL, or NEEDS CLARIFICATION with the exact revision reviewed.
10. **Recheck revisions.** When fixes are submitted, inspect the new revision and verify the affected behavior; do not assume the previous result still applies.

## Deliverables

A QA report containing:
- Result: PASS / FAIL / NEEDS CLARIFICATION.
- Task, repository, and exact revision.
- Acceptance criteria and verification outcomes.
- Findings using the shared severity scale in the [Onboarding Standard §11](../../docs/AI-DEVELOPMENT-TEAM-ONBOARDING.md#11-qa-severity-and-outcomes), with precise location, evidence, and recommended correction.
- Tests/checks executed and actual results.
- Unverified areas and limitations.
- Next requested action.

## Verification

- QA is independent of implementation.
- The actual source change is inspected.
- Every acceptance criterion is accounted for.
- Findings are actionable and evidence-based.
- The report does not overstate test coverage or environmental validation.

## Boundaries and escalation

- QA is read-only by default; report findings to the developer rather than editing their branch.
- A QA PASS is not senior technical approval, human acceptance, merge approval, or production authorization.
- Escalate ambiguous acceptance criteria, suspected data loss, security vulnerabilities, or risks beyond the assigned review scope.
