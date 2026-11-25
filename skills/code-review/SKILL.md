---
name: code-review
description: Review an actual code change for correctness, maintainability, compatibility, and risk; use when assessing a commit, pull request, or proposed patch.
---

# Code Review

## Purpose

Provide an evidence-based assessment of a specific code revision against its requirements and the target project's conventions.

## When to use

Use for reviewing pull requests, commits, patches, or other reviewable code changes. For a formal independent QA pass, also follow the [independent QA skill](../independent-qa/SKILL.md).

## Prerequisites and context

- Exact repository and revision (PR, branch, commit, or patch).
- Original task and approved requirements.
- Relevant project instructions and architecture documentation.
- Read access to the actual changed files and surrounding code.

If the diff or exact revision is unavailable, state that a full review cannot yet be performed.

## Procedure

1. **Establish scope.** Record the repository, PR/change reference, base and head revisions, and files reviewed.
2. **Read the contract.** Understand the task, acceptance criteria, constraints, and relevant design decisions.
3. **Inspect the full change.** Review the diff and enough surrounding code to understand call paths, data contracts, error handling, and side effects.
4. **Check correctness.** Look for logic errors, missing cases, invalid assumptions, race conditions, resource leaks, and incorrect error handling.
5. **Check integration.** Assess API and data compatibility, configuration, migrations, dependencies, and affected callers.
6. **Check maintainability.** Assess clarity, cohesion, unnecessary complexity, duplication, and consistency with project conventions.
7. **Check tests.** Evaluate whether tests cover the changed behavior, edge cases, and regressions. Run appropriate checks only when the environment permits.
8. **Check security and operations.** Consider authorization, validation, sensitive data, secrets, logging, resilience, and observability. Use the dedicated security review skill for a deeper security assessment.
9. **Validate findings.** Tie each actionable finding to a precise location and explain its impact. Avoid speculative findings and personal style preferences.
10. **Report the outcome.** Distinguish blocking defects, non-blocking improvements, questions, and areas not verified.

## Deliverables

Report:
- Result: PASS, FAIL, or NEEDS CLARIFICATION.
- Exact repository and revision reviewed.
- Scope/files reviewed.
- Findings ordered by severity, each with location, impact, evidence, and recommended correction.
- Checks actually run and results.
- Areas not reviewed or not verified.
- Recommended next action.

## Verification

- Review is based on the actual diff, not only a summary.
- Findings are reproducible or supported by clear code evidence.
- The reviewed revision is unambiguous.
- PASS does not imply tests or environments were verified when they were not.

## Boundaries and escalation

- Do not claim to have reviewed code you could not access.
- Do not modify the implementation during a read-only review unless explicitly assigned a separate implementation task.
- Do not approve your own implementation as an independent reviewer.
- Escalate unresolved requirements, high-impact security concerns, data-loss risks, and architectural conflicts.
