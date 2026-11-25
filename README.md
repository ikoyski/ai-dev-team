# AI Development Team

Shared standards and operating guides for human and AI-assisted software development across projects of any organization, domain, or scale.

## Start here

- [AI Development Team — Onboarding & Operating Standard](docs/AI-DEVELOPMENT-TEAM-ONBOARDING.md)

## Scope

These documents define reusable team roles, responsibilities, development lifecycle, task contracts, review and QA practices, collaboration protocols, and security expectations. They are not specific to any organization, repository, coding agent, model provider, or project-management tool. Webtools Kanban is one example integration, not the team's identity or scope.

Project-specific documentation is authoritative for that project's architecture, conventions, environments, and technical decisions. The central standards in this repository are an optional reference and should be consulted only when a task requires shared team roles, workflow, governance, or templates.

## Context loading policy

**Project context by default; team context only when needed.**

- Start with the individual project's own instructions and load only the project documentation relevant to the task.
- Do not fetch or load this repository by default for ordinary implementation, bug fixes, or project-specific code reviews.
- Consult the relevant central guide only when shared team standards are needed (for example, role assignment, cross-agent handoffs, workflow states, branching policy, or creating a requirements plan).
- Load templates and examples only when creating the corresponding artifact.
- Do not duplicate project architecture or technical instructions here, and do not copy the entire team standard into each project.
- If a project adopts these standards, its own instructions may link to this repository without requiring agents to read it on every task.

Each project repository remains the source of truth for its own technical context. This repository owns only shared, reusable team practices.

## Repository structure

    .
    ├── README.md
    ├── docs/
    │   ├── AI-DEVELOPMENT-TEAM-ONBOARDING.md
    │   ├── AI-DEVELOPMENT-WORKFLOW-AND-BRANCHING.md
    │   ├── SKILL-AUTHORING-STANDARD.md
    │   └── templates/
    ├── skills/
    │   ├── requirements-analysis/
    │   ├── technical-planning/
    │   ├── code-review/
    │   ├── independent-qa/
    │   ├── security-review/
    │   └── project-coordination/
    └── projects/

Shared skills provide focused, reusable procedures and should be loaded only when relevant. Project-specific skills remain in their project repositories.


## Operating documentation

- [AI Development Team Onboarding & Operating Standard](docs/AI-DEVELOPMENT-TEAM-ONBOARDING.md) — organization-wide roles, principles, workflow states, quality gates, and security rules.
- [AI Development Workflow and Branching](docs/AI-DEVELOPMENT-WORKFLOW-AND-BRANCHING.md) — current capability-based agent handoffs, requirement planning, QA, test deployment, human acceptance, and Git branch strategy.
- [Feature Requirements Template](docs/templates/FEATURE-REQUIREMENTS-TEMPLATE.md) — reusable starting point for approved feature plans.
- [Project Requirement Plans](projects/README.md) — directory and naming convention for project plans.
- [Skill Authoring Standard](docs/SKILL-AUTHORING-STANDARD.md) — format, placement, quality, and maintenance rules for reusable skills.

## Shared skills

- [Requirements Analysis](skills/requirements-analysis/SKILL.md) — clarify requests and produce testable requirements.
- [Technical Planning](skills/technical-planning/SKILL.md) — turn approved requirements into a technical approach and task breakdown.
- [Code Review](skills/code-review/SKILL.md) — assess an actual code change and report evidence-based findings.
- [Independent QA](skills/independent-qa/SKILL.md) — verify another contributor's implementation against acceptance criteria.
- [Security Review](skills/security-review/SKILL.md) — assess security risks and control gaps.
- [Project Coordination](skills/project-coordination/SKILL.md) — turn approved plans into tracked tasks and coordinate ownership, workflow, and handoffs.

Load a skill only when its task is relevant. Keep project-specific procedures in the project repository; shared skills must not become a dependency for routine project work.
