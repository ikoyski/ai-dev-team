# Feature Requirements — <Feature Name>

> Store approved plans at `projects/<project-name>/<feature-name>/requirements.md`. This document is the shared requirements baseline for related Kanban cards and pull requests.

## Metadata

- **Project:** <project name>
- **Repository:** <GitHub repository URL>
- **Feature:** <feature name>
- **Status:** Draft / In Review / Approved / Implementing / Accepted
- **Prepared by:** ChatGPT / <contributor>
- **Product owner:** Human Team Lead
- **Created:** YYYY-MM-DD
- **Last updated:** YYYY-MM-DD
- **Related Kanban cards:** <links>
- **Related PRs:** <links>
- **Related architecture decisions:** <links>

## 1. Problem statement

What problem, limitation, or opportunity are we addressing?

## 2. Objective and desired outcome

Describe the outcome and its value. Explain how success will be recognized.

## 3. Background and current behavior

Describe relevant existing behavior, architecture, constraints, and prior decisions. Link to project documentation, source files, issues, or examples where useful.

## 4. Proposed behavior

Describe the expected behavior from the user's and system's perspectives.

## 5. Functional requirements

List clear, numbered, testable requirements.

1. **FR-1:** ...
2. **FR-2:** ...

## 6. Non-functional requirements

Include relevant performance, reliability, observability, security, privacy, accessibility, compatibility, and maintainability requirements.

## 7. Scope

### In scope

- ...

### Out of scope

- ...

## 8. Technical design and constraints

Describe affected components, interfaces, data changes, integrations, and architectural constraints. Distinguish mandatory requirements from implementation suggestions. Identify alternatives or trade-offs when important.

## 9. Security and privacy considerations

Describe authentication, authorization, secrets, sensitive data, trust boundaries, and relevant abuse cases.

## 10. Dependencies and sequencing

List prerequisite tasks, external dependencies, and required implementation order. Identify anything that blocks a task from entering Ready.

## 11. Acceptance criteria

Use observable conditions. Given / When / Then is encouraged where helpful.

- [ ] **AC-1:** Given ... when ... then ...
- [ ] **AC-2:** ...

## 12. Testing requirements

List required unit, integration, end-to-end, regression, security, performance, and manual tests as applicable. Include exact commands when known. State what evidence should be attached to the PR or card.

## 13. Deployment, migration, and rollback

Describe environment impact, configuration changes, data migrations, deployment order, verification, and rollback/recovery requirements.

## 14. Suggested task breakdown

Claude.ai uses this section to review and create Kanban cards after the plan is approved. The final assignments must name one developer and a different QA reviewer per card.

| Task | Description / acceptance criteria | Developer | QA reviewer | Dependencies |
|---|---|---|---|---|
| 1 | ... | Antigravity / Claude Code | Opposite agent | ... |

Every card links back to this requirements document. Use the standard workflow states from the onboarding document; do not create extra columns just for individual handoff steps.

## 15. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| ... | ... | ... |

## 16. Open questions and decisions

| Question / decision | Owner | Status | Resolution |
|---|---|---|---|
| ... | ... | Open / Decided | ... |

## 17. Approval

- **Team Lead decision:** Approved / Changes requested / Rejected
- **Approved by:** <human approver>
- **Date:** YYYY-MM-DD
- **Approved revision/commit:** <commit SHA or link>
- **Notes:** ...

Only an approved revision is the implementation baseline. Material changes after approval must be reviewed and approved again.
