# Skill Authoring Standard

## 1. Purpose

This standard defines how to create and maintain reusable task skills for the AI Development Team. A skill is a focused procedure that helps a capable contributor perform a recurring task consistently. It is not a role description, project manual, or replacement for approved requirements.

## 2. Ownership and placement

- **Shared skills** belong in this repository when the procedure is useful across unrelated projects, organizations, and technology stacks.
- **Project skills** belong in the relevant project repository when they depend on that project's domain, architecture, tools, environments, or operational conventions.
- **Technology skills** belong centrally only when they are genuinely reusable and maintained independently of a single project.
- Do not copy project-specific instructions into shared skills or make a project depend on retrieving every shared skill for routine work.

Suggested layout:

```text
skills/
  <skill-name>/
    SKILL.md
```

Use lowercase kebab-case directory names.

## 3. Required skill format

Every `SKILL.md` must begin with YAML frontmatter:

```yaml
---
name: skill-name
description: A concise statement of the task and when this skill should be used.
---
```

The body should use these sections where applicable:

1. **Purpose** — the task this skill supports and its intended outcome.
2. **When to use** — clear triggers; include when not to use it if ambiguity is likely.
3. **Prerequisites and context** — required access, project documents, approvals, tools, and inputs.
4. **Procedure** — ordered, actionable steps.
5. **Deliverables** — expected report, artifact, or handoff.
6. **Verification** — evidence needed to support completion.
7. **Boundaries and escalation** — authority limits, safety constraints, and when to stop or ask.

Keep the skill focused. Omit sections that add no value, but do not omit important prerequisites, boundaries, or verification.

## 4. Authoring principles

- **Task-oriented:** describe how to perform one coherent task, not everything a role might do.
- **Project-agnostic by default:** use generic terms and placeholders; do not assume a particular repository, organization, framework, model, or work tracker.
- **Evidence-based:** distinguish actions attempted from actions confirmed; never claim tests, reviews, deployments, or changes succeeded without evidence.
- **Least context:** identify the minimum project context needed. Load project instructions and approved requirements before implementation or review; retrieve shared standards only when relevant.
- **No duplicated sources of truth:** link to canonical standards and project documentation rather than copying their full contents.
- **Capability-aware:** verify available tools, permissions, and access before assigning or attempting actions.
- **Human authority:** preserve human approval for scope changes, consequential decisions, merges, releases, and production changes as defined by project policy.
- **Risk-proportionate:** scale review depth to the impact and sensitivity of the work.
- **Tool-independent:** explain the required outcome and evidence; provide tool-specific instructions only when the skill is explicitly tool-specific.

## 5. Procedure quality

A useful procedure should:
- State the sequence of work in a way another qualified agent can follow.
- Identify decisions and conditions that change the path.
- Say what to inspect and what evidence to retain.
- Explain what to do when information, access, or evidence is missing.
- Avoid vague directions such as "check everything" without defining the relevant scope.
- Avoid prescribing commands or frameworks unless they are necessary and broadly applicable.

## 6. Outputs and handoffs

Specify a concise output format when consistency matters. A handoff should normally identify:
- Task and scope.
- Inputs and exact revision or artifact examined.
- Actions performed.
- Findings, decisions, and evidence.
- Checks actually run and their results.
- Limitations, unresolved questions, and blockers.
- Recommended next action and intended recipient.

Do not require a lengthy report for a trivial task. Match the output to the task's risk and complexity.

## 7. Maintenance

Before adding or changing a skill:
1. Check whether an existing skill already covers the task.
2. Confirm the procedure is reusable at the proposed location.
3. Check links to canonical standards and ensure they remain valid.
4. Remove duplicated, stale, tool-specific, or project-specific instructions.
5. Update the skill when its procedure or required evidence changes.

A skill should remain small enough to load only when needed. Do not automatically load every skill at the start of a task.

## 8. Relationship to other documentation

| Artifact | Responsibility |
|---|---|
| README | Human-facing repository overview and navigation |
| Onboarding standard | Shared roles, governance, decision rights, lifecycle, and quality gates |
| Workflow and branching guide | Shared handoffs, workflow states, and branch strategy |
| Skill | Repeatable procedure for a specific task |
| Project README / CLAUDE.md / AGENTS.md | Project-specific instructions and technical context |
| Approved requirements | Expected behavior, scope, and acceptance criteria for a change |
| Changelog | Recorded project or repository changes |

Skills complement these sources; they do not supersede them.
