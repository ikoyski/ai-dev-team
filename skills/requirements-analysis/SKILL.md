---
name: requirements-analysis
description: Clarify a software request and turn it into testable, approved requirements; use before technical planning when the desired outcome, scope, or acceptance criteria are unclear.
---

# Requirements Analysis

## Purpose

Turn an initial request, problem, or idea into a clear statement of expected outcomes and constraints. This skill defines requirements, not the technical solution.

## When to use

Use when a feature, bug, refactor, integration, or operational change needs clarification before design or implementation.

Do not use for a fully specified task that only needs implementation, or to silently revise already approved requirements.

## Prerequisites and context

- Identify the relevant project and its authoritative documentation.
- Read only project context relevant to the request.
- Identify the requester and who can approve scope and priorities.
- For an existing behavior change, inspect available evidence such as reproduction steps, current behavior, or related code when access permits.

## Procedure

1. **Restate the problem.** Describe the user or system need, current pain, and desired outcome without assuming a solution.
2. **Clarify ambiguity.** Ask focused questions about actors, triggers, expected behavior, edge cases, constraints, and success conditions. Do not ask questions already answered by authoritative project context.
3. **Define scope.** Separate included behavior from explicit non-goals. Identify affected users, systems, data, and integrations where known.
4. **Capture constraints.** Record compatibility, security, privacy, performance, accessibility, operational, and policy constraints that materially apply.
5. **Define observable criteria.** Write acceptance criteria that can be demonstrated or tested. Include failure paths and boundary conditions where relevant.
6. **Identify dependencies and risks.** Note prerequisites, external systems, migrations, unresolved decisions, and assumptions.
7. **Check completeness.** Ensure the requirements are understandable without relying on private chat history and are sufficiently precise for technical planning.
8. **Request approval.** Present assumptions and open questions explicitly. Do not treat a draft as approved until the authorized human confirms it.

## Deliverables

Produce a concise requirements brief or approved requirements document containing:
- Problem and desired outcome.
- Actors and relevant scenarios.
- Functional and applicable non-functional requirements.
- Scope and non-goals.
- Constraints, assumptions, dependencies, and risks.
- Testable acceptance criteria.
- Open questions and decisions requiring approval.

Use the central [Feature Requirements Template](../../docs/templates/FEATURE-REQUIREMENTS-TEMPLATE.md) when creating a formal plan.

## Verification

- Each acceptance criterion describes an observable result.
- Requirements distinguish expected behavior from implementation suggestions.
- Assumptions and unresolved questions are visible.
- The approved version and approver are traceable in the designated project record.

## Boundaries and escalation

- Do not invent requirements to fill gaps; ask or mark them unresolved.
- Do not make architecture or technology choices unless explicitly asked to propose options.
- Do not expand scope without approval.
- Escalate conflicting stakeholder needs, privacy/security concerns, or consequential data and compatibility decisions to the appropriate human authority.
