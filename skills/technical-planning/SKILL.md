---
name: technical-planning
description: Translate approved requirements into a feasible technical approach, task breakdown, risks, and verification plan; use before implementation of non-trivial changes.
---

# Technical Planning

## Purpose

Create a reviewable technical plan that explains how approved requirements can be delivered within the target project's architecture and constraints.

## When to use

Use for features, integrations, data changes, cross-component work, significant refactors, or changes with meaningful security or operational impact.

For a trivial, well-understood change, use proportionate judgment rather than producing unnecessary planning overhead.

## Prerequisites and context

- Approved requirements or a clearly authorized investigation scope.
- Project README and relevant project instructions (such as CLAUDE.md or AGENTS.md).
- Relevant architecture records, code, interfaces, tests, and deployment information.
- Verified access to the repository and any systems needed for investigation.

Do not load the entire shared team repository or unrelated project documentation by default.

## Procedure

1. **Confirm the baseline.** Identify the approved requirements revision, project, relevant branch/baseline, and known constraints.
2. **Inspect the existing system.** Trace affected components, data flows, APIs, dependencies, configuration, tests, and operational paths. Distinguish verified facts from assumptions.
3. **Identify design options.** Consider viable approaches and explain meaningful trade-offs. Avoid unnecessary alternatives when the design is straightforward.
4. **Recommend an approach.** Describe component responsibilities, interfaces, data changes, failure behavior, security boundaries, and compatibility considerations as applicable.
5. **Assess risk.** Consider migration and rollback, reliability, observability, performance, privacy, security, and deployment sequencing.
6. **Plan verification.** Map acceptance criteria to unit, integration, end-to-end, security, operational, and manual checks as appropriate.
7. **Break down delivery.** Define independently reviewable tasks, dependencies, ordering, and expected deliverables. Each task should have clear acceptance criteria.
8. **Identify decisions.** Separate settled design choices from assumptions and decisions requiring human approval.
9. **Review completeness.** Check that the proposed plan meets the approved requirements without silently changing their scope.
10. **Submit for approval.** Record the plan in the project's designated planning location or the agreed central project-plan location.

## Deliverables

A technical plan should include, as applicable:
- Requirements and baseline references.
- Current-state findings.
- Proposed design and component responsibilities.
- API/data/configuration changes.
- Security and operational considerations.
- Alternatives and trade-offs.
- Risks, dependencies, migration, and rollback.
- Task breakdown and acceptance criteria, using the [Feature Requirements Template](../../docs/templates/FEATURE-REQUIREMENTS-TEMPLATE.md) when preparing the formal requirements/plan artifact.
- Test and deployment strategy.
- Open decisions and approval status.

## Verification

- Every material requirement maps to a design element and verification method.
- The plan is grounded in inspected project context.
- Assumptions, unknowns, and unverified access are explicit.
- Tasks are independently actionable and reviewable.
- Required human approvals are identified.

## Boundaries and escalation

- Do not implement while presenting a plan as though it were already approved.
- Do not alter approved requirements without returning the change to the authorized approver.
- Do not invent repository behavior, APIs, deployment capabilities, or tool access.
- Escalate architecture conflicts, data-loss risk, security-sensitive design choices, and material cost or operational trade-offs.
