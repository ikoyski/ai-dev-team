---
name: project-coordination
description: Turn approved plans into actionable tracked work, maintain task ownership and workflow state, and coordinate evidence-based handoffs; use when preparing or managing project work items.
---

# Project Coordination

## Purpose

Coordinate approved project work from task preparation through implementation, review, acceptance, and completion. Keep the work tracker accurate and make handoffs clear without changing approved requirements or exercising authority reserved for the Team Lead.

This skill describes a reusable coordination procedure. Follow the project's own tracker conventions and the shared [AI Development Team Onboarding Standard](../../docs/AI-DEVELOPMENT-TEAM-ONBOARDING.md) and [Workflow and Branching Guide](../../docs/AI-DEVELOPMENT-WORKFLOW-AND-BRANCHING.md) when the project has adopted them.

## When to use

Use when:
- Converting an approved requirements document or technical plan into work items.
- Decomposing approved scope into independently deliverable tasks.
- Assigning or coordinating developers and independent QA reviewers.
- Updating task status, recording handoffs, tracking blockers, or coordinating review and deployment stages.
- Auditing whether tracker records accurately reflect confirmed work.

Do not use to define or approve product requirements, make unapproved architecture decisions, implement tasks, independently QA an implementation, or authorize merges/releases.

## Prerequisites and context

- Identify the project, its authoritative repository, and the approved requirements/plan revision.
- Read the project's instructions and only the technical context needed to understand task boundaries.
- Consult the shared onboarding/workflow standards only when the project has adopted them or their shared rules are needed.
- Identify the project's work tracker, workflow states, existing labels, fields, templates, and conventions.
- Verify actual tracker access and the available operations before promising or attempting changes.
- Confirm the Team Lead's priorities, approval status, and any constraints on assignment or scheduling.
- Identify participants' current capabilities and permissions; do not infer access from their product name or access in another environment.
- For AI developers, consult the access-controlled developer roster and identify each contributor by its stable DevN ID (Dev1, Dev2, … DevN), not only by tool or model.
- Verify that each DevN has a distinct Kanban user identity and its own OAuth token/session context. The MCP OAuth client ID may be shared by instances of the same application type; user accounts and tokens must not be shared.

If the plan is not approved, materially ambiguous, or missing acceptance criteria, keep the work in the appropriate proposed/blocked state and request clarification or approval. Do not create implementation-ready tasks from an unapproved proposal.

## Procedure

1. **Establish the source of truth.**
   - Record the canonical requirements/plan link and exact revision (commit, version, or other stable identifier).
   - Confirm that the plan is approved by the authorized human.
   - Treat the approved plan as authoritative; tracker cards summarize and link to it rather than silently replacing it.
   - Flag conflicts between the plan, project policy, and existing tracker records for resolution.

2. **Review and decompose the work.**
   - Identify deliverable outcomes, dependencies, sequencing, and integration points.
   - Create tasks that are independently actionable and reviewable, with one primary outcome each.
   - Keep tightly coupled work together when splitting it would create unclear ownership or unusable intermediate changes.
   - Preserve traceability to relevant requirements and acceptance criteria.
   - Identify cross-cutting, high-risk, or externally dependent work for explicit coordination.
   - Do not invent scope, technical decisions, estimates, deadlines, or priorities. Mark unknowns and seek a decision.

3. **Prepare each work item.**
   Include, as applicable:
   - Specific, action-oriented title.
   - Objective and relevant background.
   - Link to the approved plan and relevant requirement/acceptance-criterion references.
   - Scope, explicit non-goals, and constraints.
   - Observable acceptance criteria.
   - Testing requirements and expected evidence.
   - Dependencies and sequencing.
   - Relevant project repository, expected branch/PR conventions, and deliverables.
   - One primary developer identified by DevN and a different independent QA reviewer identified by another DevN.
   - Required technical authority and human approver, where applicable.
   - Priority and target date only when confirmed by the Team Lead or project policy.

   Use the project's existing card template and required fields. Keep card content concise; do not paste entire design documents or duplicate the approved plan.

4. **Apply labels and metadata consistently.**
   - Inspect the project's existing label taxonomy and reuse applicable labels.
   - Apply only labels whose meaning is clear and supported by the task (for example, established type, priority, component, or risk labels).
   - Do not create new labels, invent a taxonomy, or change shared project conventions without authorization.
   - Preserve existing labels when the tracker operation replaces rather than adds the label set.
   - If a required label or field is missing, report the gap and ask the authorized project owner how to handle it.

5. **Assign ownership and verify readiness.**
   - Name one primary DevN developer and a different DevN as independent QA reviewer; record each instance's tool/harness as supplemental information.
   - Verify that the developer and reviewer are different instances with separate Kanban user identities, workspaces, and credential stores. Different tool names alone do not prove identity separation.
   - Check that each participant has the repository, tracker, environment, and tool access required for the assigned responsibility.
   - Use `dev:devN` and `qa:devN` labels or native owner/reviewer fields; tool-family labels are supplemental only.
   - Avoid assigning a participant to a role that conflicts with their work on the same change.
   - Confirm dependencies are resolved or explicitly represented.
   - Move work to Ready only when it is approved, sufficiently specified, and has clear ownership and acceptance criteria under the project's workflow.
   - If capacity or access is unknown, do not imply that the task has been accepted.

6. **Create or update tracker records.**
   - Check for an existing card before creating a new one to avoid duplicates.
   - Use the supported tracker operations and only fields the tool actually accepts.
   - Record the canonical plan link, relevant repository references, and task dependencies.
   - After each write, inspect the returned result or read the record back to confirm the intended title, description, labels, assignments, and state.
   - Capture the resulting card identifier and URL for subsequent handoffs.
   - If an operation fails or its result is ambiguous, stop and report the uncertainty; do not retry blindly if doing so could create duplicates.

7. **Coordinate work and workflow transitions.**
   - Follow the project's established workflow states and entry conditions; do not add columns or bypass gates for convenience.
   - Move a card only when the transition's required evidence and approvals are present.
   - Keep the card in its current state while recording substages in attributed comments when the workflow calls for it.
   - Record blockers with their impact, responsible party, and specific action needed. Use Blocked when progress depends on missing access, an external decision, an unavailable environment, or unresolved requirements.
   - Route minor clarification to the appropriate decision-maker; use Revision when the developer must change or clarify the implementation.
   - Never interpret QA PASS as human acceptance, merge approval, or production authorization.
   - For rejected test acceptance, coordinate the documented fix-forward or revert decision; do not mark the work eligible for production.

8. **Record handoffs using consistent comments.**
   Use the project's templates where available. Otherwise include clear role attribution and the relevant fields below.

   **Task assignment / Ready**
   ```text
   [Coordinator]
   Status: Ready for implementation
   Approved plan: <link and revision>
   Scope / acceptance criteria: <references>
   Developer: <DevN> (<tool/harness>)
   Independent QA: <different DevN> (<tool/harness>)
   Dependencies: <items or none>
   Next: Developer to review the task and confirm acceptance.
   ```

   **Developer handoff**
   ```text
   [Developer | DevN | tool/harness]
   Status: Implementation complete
   Branch: <branch>
   PR/change: <link>
   Commit: <sha>
   GitHub actor: <GitHub user or App identity>
   Summary: <brief summary>
   Tests run and results: <actual results>
   Known limitations: <items or none>
   Next: Independent QA by <different DevN>.
   ```

   **QA handoff**
   ```text
   [QA | DevN | tool/harness]
   Result: PASS / FAIL / NEEDS CLARIFICATION
   Reviewed PR/commit: <exact revision>
   Developer reviewed: <DevN>
   Checks and results: <actual checks>
   Findings: <severity, location, evidence, or none>
   Limitations: <unverified areas>
   Next: <specific action and recipient>.
   ```

   **Blocker**
   ```text
   [Coordinator]
   Status: Blocked
   Blocker: <specific issue>
   Impact: <what cannot proceed>
   Decision/access/action needed from: <responsible party>
   Next review: <condition or agreed checkpoint>
   ```

   **Test deployment and human acceptance**
   Record the environment, URL where appropriate, exact deployed commit, deployment evidence, health check, human test scenarios, and the Team Lead's explicit ACCEPTED or REJECTED decision. Record production authorization separately and only when explicitly granted.

9. **Maintain traceability and monitor progress.**
   - Keep links among the approved plan, cards, branches, PRs, commits, review reports, deployments, and acceptance records.
   - Follow up on stale assignments, unresolved findings, and dependencies without changing scope or priority unilaterally.
   - Summarize status using confirmed tracker data and distinguish facts, estimates, and unknowns.
   - When a card changes materially, ensure the reason and decision are recorded.
   - Close or mark Done only after all project-required gates and verification are complete.

10. **Verify every operation and report accurately.**
    - For each create, edit, assignment, comment, label, or state change, rely on a successful tool response and, where practical, read-after-write confirmation.
    - Distinguish prepared content from content actually submitted to the tracker.
    - If access is unavailable, provide ready-to-copy card/comment content and identify the authorized person who must enter it.
    - Report identifiers, URLs, and resulting states only when returned or independently confirmed by the tracker.

## Deliverables

Depending on the task, provide:
- A task breakdown linked to the approved plan.
- Created or updated tracker records with confirmed identifiers and URLs.
- Ownership, reviewer, dependency, and label mapping.
- Attributed implementation, QA, review, deployment, and acceptance handoffs.
- A concise coordination summary listing current states, blockers, pending decisions, and next actions.
- Ready-to-copy tracker content when direct tracker access is unavailable.

## Verification

Before reporting coordination complete, confirm:
- The source plan is approved and its exact revision is recorded.
- Each task maps to approved scope and acceptance criteria.
- Tasks are neither duplicated nor missing required dependencies.
- Each Ready task has one identified DevN developer and a different DevN QA reviewer with separate identity/session contexts.
- Labels and metadata follow existing project conventions.
- Tracker writes were confirmed and the resulting records are accessible.
- State changes satisfy the project's transition rules and approval gates.
- Handoffs identify the exact relevant artifact/revision and actual evidence.
- Blockers and unresolved decisions have named owners or responsible parties.
- No action is described as completed solely because it was attempted.

## Boundaries and escalation

- Never approve requirements, change approved scope, or make architecture decisions on behalf of the Team Lead or architect.
- Never fabricate card creation, comments, assignments, labels, state changes, test results, deployments, approvals, or completion.
- Never assign a DevN as the independent QA reviewer of its own change; verify instance IDs, not merely tool names.
- Never move work past a required QA, senior review, human acceptance, merge, or release gate.
- Never treat a reviewer's PASS, a card state, or a general instruction as authorization for a specific merge or production release.
- Do not put secrets, credentials, private keys, or sensitive personal data in tracker records.
- Escalate missing approval, conflicting requirements, unavailable capabilities, unclear ownership, serious risk, or disputed workflow decisions to the appropriate human authority.
