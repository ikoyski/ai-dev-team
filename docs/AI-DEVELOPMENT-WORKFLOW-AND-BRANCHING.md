# AI Development Workflow and Branching

**Status:** Working procedure  
**Scope:** Projects of any organization, domain, or scale  
**Parent standard:** [AI Development Team Onboarding](AI-DEVELOPMENT-TEAM-ONBOARDING.md)

This guide translates the organization-wide standard into a practical workflow for the current team. It describes the intended responsibilities while allowing each handoff to adapt to the actual repository, Kanban MCP, and deployment access available to the participating agent.

## 1. Operating principles

1. The Human Team Lead owns priorities, approves requirements and consequential decisions, performs final test acceptance, and authorizes production promotion.
2. ChatGPT is the primary requirements and technical-plan author, and can later perform senior code review and approved GitHub operations.
3. Claude.ai is the primary work coordinator and Kanban operator when it has Kanban MCP access.
4. Registered DevN instances are the implementation and independent QA pool; assign by capability, capacity, access, and independence rather than by tool name alone.
5. The same agent must not implement and independently QA the same change.
6. A participant performs only actions supported by its actual access. If it cannot access a system, it prepares a clear handoff to someone who can.
7. The standard workflow states remain those in the onboarding document. This guide does not require extra columns for each kind of review or deployment.
8. GitHub PRs and commits are the evidence of code changes. Kanban cards coordinate work. Approved requirement documents define intended behavior.
9. Every independently operating developer runtime has a stable DevN identity (Dev1, Dev2, … DevN), regardless of tool, model, or provider.

OpenRouter is the model/provider routing layer; Claude Code is the coding harness. Identify the coding environment as Claude Code in task reports and optionally record the selected OpenRouter model.

## 2. Responsibility map

| Participant | Main responsibility | Typical actions when access is available |
|---|---|---|
| Human Team Lead | Product Owner and final authority | Discusses needs, approves plans, prioritizes work, reviews Kanban status, tests the test environment, authorizes production promotion |
| ChatGPT.com | Chief Architect / Requirements Author / Senior Code Reviewer | Creates and updates requirement plans in this repository, inspects project code, prepares technical plans, reviews PRs, performs explicitly authorized GitHub merge operations |
| Claude.ai | Project Coordinator (may support architecture discussions when assigned) | Reads approved plans, creates/updates Kanban cards, records handoffs, moves cards, coordinates work, initiates or coordinates test deployment if authorized access exists |
| Dev1 — Claude Code | Developer / independent QA reviewer | Implements assigned tasks or independently reviews another DevN's changes; uses the `kanban-claude` MCP OAuth client with its own SSO/Kanban user |
| Dev2 — Antigravity | Developer / independent QA reviewer | Implements assigned tasks or independently reviews another DevN's changes; uses the `kanban-antigravity` MCP OAuth client with its own SSO/Kanban user |
| Dev3 … DevN — additional instances | Developer / independent QA reviewer | Onboarded individually with stable IDs, separate identities/credentials, and the existing MCP client registration for the application type |

### Developer identity and MCP access model

Developer IDs identify independently operating instances, not tool families. Dev1 remains Dev1 if its model/provider route changes; a second independently provisioned Claude Code or Antigravity runtime receives a new DevN ID.

| Instance | Runtime/tool | Kanban MCP OAuth client | Kanban login |
|---|---|---|---|
| Dev1 | Claude Code | `kanban-claude` | Dev1's own Google/SSO account and distinct Kanban user |
| Dev2 | Antigravity | `kanban-antigravity` | Dev2's own Google/SSO account and distinct Kanban user |
| Dev3 | Another Claude Code instance | `kanban-claude` (reuse) | Dev3's own Google/SSO account and distinct Kanban user |
| Dev4 | Another Antigravity instance | `kanban-antigravity` (reuse) | Dev4's own Google/SSO account and distinct Kanban user |
| DevN | Any already-supported client type | Reuse the matching application-type client ID | DevN's own Google/SSO account and distinct Kanban user |

Rules:
- One OAuth client registration per MCP client application/integration type, not one per DevN. The client ID identifies the connecting application; the token subject identifies the user.
- Every DevN signs in independently and has its own OAuth token store, browser profile, workspace, and GitHub credential context. Never copy tokens or sessions between instances.
- A new DevN using an existing supported client type should not require a new OAuth registration or Auth Server code change, provided the existing registration supports the redirect URI, resource, and scopes.
- Each Kanban user receives only the board membership and permissions required for that developer's tasks. Developer accounts never receive human Team Lead approval or production authority.
- Map each DevN to an approved, separately attributable GitHub identity/automation identity. If a shared GitHub App performs operations for multiple DevN instances, record both the actual GitHub App actor and the initiating DevN in PR/commit handoffs; do not describe the shared App as a unique GitHub actor for each instance.
- Use `dev:devN` and `qa:devN` labels or native owner/reviewer fields so Kanban assignments identify the instance, not just its tool family.

Maintain actual account mappings in the access-controlled roster using [Developer Instance Registration Template](templates/DEVELOPER-INSTANCE-REGISTRATION-TEMPLATE.md). Never store credentials or private account details in this public standards repository. The canonical requirements are in [Developer Instance Identity and Access](DEVELOPER-INSTANCE-IDENTITY-AND-ACCESS.md).

### Capability-based handoff

Before assigning an action, confirm that the participant has the required access:

| Required capability | If available | If unavailable |
|---|---|---|
| Read the AI Development Team repository | Read the approved requirement plan directly | Coordinator or Team Lead supplies the canonical link/content; do not rely on an unapproved paraphrase |
| Write the AI Development Team repository | Author/update approved planning artifacts | ChatGPT or another authorized repository contributor performs the update |
| Kanban MCP | Read cards, comment, and move cards using supported operations | Coordinator or Team Lead with access performs the tracker update |
| Project repository | Inspect code, create branches, commit, push, open PRs, review diffs | Request repository access or a handoff containing sufficient review evidence; do not claim code inspection |
| Deployment platform / controls (e.g., Render) | Deploy or inspect the test environment within granted permissions | Prepare deployment instructions and hand off to the authorized operator |
| Test site | Perform relevant manual validation | Ask the Team Lead or authorized tester to perform acceptance testing |

Access is environment- and account-specific and can change. Do not treat this table as a promise that a product always has a capability. Check the actual connection and permissions at the time of work.

## 3. Requirement planning and source of truth

### 3.1 Discuss and define

The Team Lead discusses a feature, bug, refactor, or other change with ChatGPT. ChatGPT clarifies the objective, examines relevant project context and code where accessible, and prepares a plan containing:

- Problem statement and desired outcome.
- Current behavior and relevant background.
- Functional and non-functional requirements.
- Scope and explicit non-goals.
- Technical design, constraints, and affected components.
- Security, compatibility, data, and operational considerations.
- Dependencies and suggested task breakdown.
- Observable acceptance criteria.
- Test strategy and required evidence.
- Deployment, migration, and rollback notes where relevant.
- Open questions and decisions requiring approval.

The Team Lead reviews and approves the plan before it becomes the implementation baseline.

### 3.2 Store approved plans in this repository

Use this structure:

```text
projects/
  <project-name>/
    <feature-name>/
      requirements.md
      (optional supporting design notes, diagrams, or decision records)
```

Example:

```text
projects/
  homelab-management-mcp/
    docker-container-log-inspection/
      requirements.md
```

Use lowercase kebab-case for directory names. The approved Markdown document is the durable requirements source. Conversations and card comments provide context but do not silently supersede it.

ChatGPT writes the approved plan to this repository when repository access is available. If it is unavailable, provide the complete plan to an authorized contributor for committing. Record the GitHub link and revision in related cards.

### 3.3 Coordinator reviews and decomposes

Claude.ai reads the approved plan when repository access permits. It checks clarity, acceptance criteria, dependencies, and task boundaries, then creates appropriately sized Kanban cards.

Each card should link to the requirement document and include:
- Objective and relevant requirement/acceptance-criteria references.
- Scope and out-of-scope notes.
- Developer and independent QA reviewer.
- Project repository and branch/PR expectations.
- Dependencies.
- Test requirements and expected evidence.

If Claude.ai cannot read the repository, the Team Lead or ChatGPT supplies the canonical document or an authorized handoff. If Claude.ai cannot access Kanban MCP, it prepares the card content for an authorized user to enter. It must not report cards created or moved unless confirmed.

Material changes to approved requirements return to the Team Lead and are reflected in the requirement document before implementation proceeds.

## 4. Standard workflow states

Use the existing states from the onboarding standard:

| State | Meaning in this workflow |
|---|---|
| **Backlog** | Work idea or request not yet approved or sufficiently specified |
| **Ready** | Approved task with clear scope, acceptance criteria, dependencies, developer, and QA reviewer |
| **In Progress** | Assigned developer is implementing the task |
| **In Review** | Implementation is submitted for independent QA and senior technical review, then remains in review through authorized test integration, deployment, and Team Lead acceptance |
| **Revision** | The original developer must correct findings or clarify the implementation |
| **Ready to Merge** | Required reviews and human acceptance are complete; the change is eligible for production promotion, subject to separate explicit authorization |
| **Done** | Changes are merged and verified according to project policy |
| **Blocked** | A dependency, decision, permission, environment, or external issue prevents progress |

Do not add mandatory columns such as "Ready for Testing" or "Human Testing" solely to represent individual handoff steps. Use the card's current state plus a clearly attributed comment to indicate which review or deployment action is pending.

### 4.1 State transition interpretation

```text
Backlog
  -> Ready
  -> In Progress
  -> In Review
       -> Revision -> In Progress -> In Review
       -> Ready to Merge
            -> Done

Any active state -> Blocked -> appropriate prior state
```

The transitions have these meanings:

- **Backlog → Ready:** The plan/task is approved, sufficiently specified, dependencies are resolved, and developer/QA ownership is known.
- **Ready → In Progress:** The assigned developer accepts the card and begins work.
- **In Progress → In Review:** Implementation is committed and pushed, a reviewable PR targets `develop`, and the developer handoff is recorded.
- **In Review → Revision:** QA or senior review finds actionable implementation issues, or the original developer can resolve a specific missing detail within the approved requirements.
- **Revision → In Progress:** The original developer accepts the findings and resumes implementation.
- **In Progress → In Review:** The developer submits the revised change and updated evidence for another independent QA pass.
- **In Review → Ready to Merge:** Independent QA and required senior review pass; the Team Lead explicitly authorizes the feature PR merge into `develop`; the test deployment is verified; and the Team Lead accepts the tested result.
- **Ready to Merge → Done:** The Team Lead separately authorizes the specific production promotion to `master`; the authorized release process completes; and required production verification is recorded.
- **NEEDS CLARIFICATION:** Keep the card in In Review while the coordinator seeks a decision if the question is minor and can be resolved without changing approved scope. Move it to Revision when the developer must clarify or correct the implementation. Move it to Blocked when progress depends on an external decision, missing access, unavailable environment, or unresolved requirements/scope approval; record who must decide and what is needed.
- **Any active state → Blocked:** Record the blocker, impact, owner, and action needed. Return to the appropriate previous state when resolved.

Use comments to make the current substage explicit while the card remains in In Review. Examples: "QA 1 in progress", "QA passed; awaiting ChatGPT review", "ChatGPT review passed; awaiting Team Lead authorization to merge into develop", "Merged into develop; awaiting test deployment", and "Test deployment ready; awaiting Team Lead acceptance". Record production authorization separately after the card reaches Ready to Merge.

## 5. Developer workflow

Initial developer/QA pairing:

| Developer | Independent QA |
|---|---|
| Dev1 — Claude Code | Dev2 — Antigravity |
| Dev2 — Antigravity | Dev1 — Claude Code |

For Dev3 through DevN, the coordinator assigns a different active developer instance based on capability, capacity, project access, and independence. A developer instance must never QA its own implementation. A reviewer may use the same tool family as the implementer only when it is a separate DevN instance with its own workspace and credentials; prefer a different tool/provider for higher-risk work where practical.

The coordinator may change pairings when a different independent reviewer is named before QA. Record the exact DevN IDs on the card and in each handoff.

When a developer takes a task from Ready:

1. Confirm the card is assigned to that developer.
2. Read the full card and linked approved requirements.
3. Read project instructions and relevant architecture documentation.
4. Check dependencies, repository access, and current `develop` baseline.
5. Move the card to In Progress if Kanban access is available; otherwise ask the coordinator to do so.
6. Create a dedicated feature branch from the latest `develop`.
7. Implement only the approved scope. Ask for clarification before material scope or architecture changes.
8. Add/update tests and documentation.
9. Run appropriate checks and inspect the final diff for unintended changes and secrets.
10. Commit and push the feature branch.
11. Open a PR targeting `develop`; do not merge it.
12. Add a card comment with branch, PR link, commit SHA, summary, tests run/results, known limitations, and the next requested action.
13. Ask the coordinator to move the card to In Review.

If the developer cannot update Kanban, the implementation is not blocked solely by that limitation; the developer supplies the handoff and asks the coordinator to update the card. If the developer cannot push/open a PR, record the exact blocker and request an authorized handoff rather than claiming the change is submitted.

## 6. Independent QA workflow

The assigned QA reviewer takes the card from In Review after the implementation handoff is available.

1. Confirm the developer and QA identities are different.
2. Read the approved requirement plan, card, acceptance criteria, and developer report.
3. Inspect the actual PR, branch, and commit diff. Do not rely solely on the developer's summary.
4. Run tests or reproduce behavior where practical and permitted.
5. Assess functional behavior, edge cases, regressions, security, compatibility, and test adequacy.
6. Record the exact revision reviewed, checks performed, results, findings, and limitations.

If QA finds actionable defects:
- Report **FAIL** with severity, precise file/line or location, evidence/reproduction, expected behavior, and recommended correction.
- Ask the coordinator to move the card to Revision.
- The original developer fixes the change, updates the PR, and returns it to In Review with a new handoff.
- The same independent QA reviewer rechecks the revised change.

If QA passes:
- Report **PASS**, exact PR/commit reviewed, checks performed, and any unverified areas.
- Leave the card in In Review.
- Notify the coordinator that independent QA is complete and request ChatGPT senior review.

QA is read-only by default. The reviewer reports findings and does not directly edit the implementer's branch. A QA PASS is not production approval.

## 7. ChatGPT senior code review

When independent QA passes, the Team Lead asks ChatGPT to review the actual change.

ChatGPT should:
1. Read the original approved requirement document and card.
2. Inspect the actual PR/diff and relevant project files.
3. Check whether the implementation meets the acceptance criteria and respects architecture, security, maintainability, and operational constraints.
4. Consider QA findings and verify whether they were addressed.
5. Report PASS, FAIL, or NEEDS CLARIFICATION with evidence and exact locations.
6. Provide a concise review handoff for the card.

If ChatGPT cannot access the PR/repository, it must clearly state that it has not independently inspected the code and request the necessary access or evidence. A summary from another agent alone is not equivalent to a code review.

If changes are required, the card moves to Revision and returns to the original developer. After fixes, independent QA repeats before ChatGPT reviews again.

If ChatGPT passes, the card remains In Review while the coordinator records that senior review passed and arranges test integration/deployment.

## 8. Test integration, deployment, and human acceptance

The test environment is deployed from `develop`.

After independent QA and ChatGPT senior review pass:

1. The coordinator confirms the feature PR targets `develop`, required checks pass, and the change is ready for test integration.
2. The Team Lead explicitly authorizes that specific feature PR to merge into `develop`. An AI reviewer's PASS is a recommendation, not merge authorization. The authorized operator merges it through the project's approved process; branch protection and required checks remain in force.
3. The test site is deployed from the resulting `develop` commit, automatically or through the authorized deployment process.
4. Record the deployed commit SHA, test URL, deployment result, and relevant health check in the card.
5. The Team Lead performs manual acceptance testing against the approved criteria and records ACCEPTED or REJECTED.

Claude.ai performs the deployment only if it actually has the required access to the deployment platform and its controls (for example, Render). Otherwise it coordinates an authorized operator and records the confirmed result. Do not claim a deployment succeeded without evidence.

If human testing fails, record expected versus actual behavior and reproduction steps. The default response is to fix forward on `develop`: move the card to Revision, have the original developer correct the change, and repeat independent QA and senior review as appropriate before redeployment. Do not promote the rejected change to production. Revert the integration from `develop` when necessary to restore a stable test branch, such as when the change causes a serious regression, security issue, or blocks other work; record the revert commit and decision on the card. The Team Lead or authorized operator decides whether to fix forward or revert based on impact.

If human testing passes, the Team Lead records explicit acceptance. The card can then move to Ready to Merge. This state means the change is eligible for production promotion; it does not itself authorize the release. The Team Lead must separately authorize the specific promotion to `master`.

## 9. Branching and environment strategy

| Branch | Purpose | Environment |
|---|---|---|
| `master` | Production-ready source | Production |
| `develop` | Integration and test source | Test site |
| `feature/<short-name>` | Isolated task implementation | No automatic deployment by default |

Required flow:

```text
master (production)
  ^
  | approved production promotion
  |
develop (test)
  ^
  | feature PRs
  |
feature/<task-name> (developer branch)
```

Rules:
- Feature branches are created from the latest `develop`.
- Feature PRs target `develop`, never `master`.
- Developers do not push directly to `develop` or `master`.
- Developers do not merge their own feature PRs.
- A feature PR may be merged into `develop` only after required reviews/checks pass and the Team Lead explicitly authorizes that specific test-integration merge.
- Test deployment uses the exact integrated `develop` commit.
- Production promotion happens only after Team Lead acceptance and explicit authorization.
- Production promotion should normally be a reviewed PR from `develop` to `master`, or a documented release process appropriate to the project.

### Important: promote only accepted work

A `develop` branch may contain multiple features, including work not yet accepted by the Team Lead. Before promoting `develop` to `master`, inspect the complete commit/change set that would be released. Do not accidentally release unrelated, unfinished, or untested work.

If `develop` contains unaccepted changes, use a controlled release branch or another project-approved promotion strategy that includes only accepted work. Record the exact release commit and included cards.

Rejected test integrations remain in Revision and are not eligible for production promotion. Fix forward on `develop` by default; revert the integration when needed to restore test-branch stability or contain a serious regression/security issue. Record the chosen action and exact commits on the card.

### Production promotion

1. Team Lead confirms which cards/features are accepted and explicitly authorizes the specific release.
2. ChatGPT inspects the exact proposed release commit/change set and required CI results.
3. ChatGPT prepares or reviews the promotion PR/process.
4. ChatGPT performs the merge only after explicit Team Lead approval for that specific promotion.
5. Verify production deployment and basic health.
6. Record the production commit and deployment result.
7. Move related cards to Done only when required integration and deployment verification is complete.

A general instruction to help with merges is not blanket authorization for future production merges. Approval is specific to the release and revision under review.

## 10. Card comment conventions

Use clear role attribution. Examples:

**Developer handoff**
```text
[Claude Code | Dev1 | Developer]
Status: Implementation complete
Branch: feature/<name>
PR: <link>
Commit: <sha>
Summary: ...
Tests run and results: ...
Known limitations: ...
GitHub actor: <GitHub user or App identity>
Initiating developer: Dev1
Next: Independent QA by Dev2 (Antigravity).
```

**QA report**
```text
[Antigravity | Dev2 | QA]
Result: PASS / FAIL / NEEDS CLARIFICATION
Reviewed PR/commit: ...
Checks run: ...
Findings:
- [High] <file:line> — evidence, reproduction, expected result
Limitations: ...
Next: ...
```

**ChatGPT review**
```text
[ChatGPT | Senior Code Review]
Result: PASS / FAIL / NEEDS CLARIFICATION
Reviewed PR/commit: ...
Findings: ...
Risks / limitations: ...
Next: ...
```

**Test deployment**
```text
[Claude.ai | Test Deployment]
Environment: Test
URL: ...
Branch: develop
Commit: ...
Deployment result: ...
Health check: ...
Next: Human acceptance testing.
```

**Human acceptance**
```text
[Human | Team Lead]
Result: ACCEPTED / REJECTED
Tested commit: ...
Test scenarios: ...
Findings: ...
Production promotion authorized: YES / NO
```

Never put secrets, credentials, private keys, or sensitive personal data in cards or comments. Preserve existing labels if an API replaces the complete label list.

## 11. Definition of successful handoff

Every handoff must identify:
- Sender and intended recipient.
- Card and requirement document.
- Current state and requested next state/action.
- Exact repository, branch, PR, and commit where relevant.
- Work performed and evidence.
- Checks actually run and their results.
- Known limitations, unresolved findings, and blockers.

The coordinator keeps the card current when it has access. When it does not, the sender provides a ready-to-copy status update for the authorized person.

## 12. Safeguards

- One primary DevN developer and a different DevN QA reviewer per task; record both identifiers.
- Every DevN uses isolated credentials, OAuth token storage, browser profile, and workspace. Shared OAuth client IDs do not permit shared user sessions.
- Record the actual GitHub user/App actor and the initiating DevN for source-control actions.
- QA checks the actual source diff and exact commit.
- QA does not directly edit the developer's branch by default.
- Review PASS, human test acceptance, and production authorization are separate decisions.
- No agent bypasses branch protections or human approval.
- Test deployment and production deployment are separately verified.
- Do not promote unaccepted work merely because it is present in `develop`.
- No agent reports a Git, Kanban, or deployment-platform action as complete without confirmation from the relevant system.
- Record decisions and evidence in shared artifacts so the process does not depend on private chat history.

## 13. Context loading and documentation boundaries

The AI Development Team repository is not a required context dependency for every development task. Each participant should begin with the individual project's own instructions and load only the context needed for the assigned work.

### For developers and reviewers

- Read the project's own `README`, `CLAUDE.md`, `AGENTS.md`, architecture notes, and relevant code as applicable to the task and the tools being used.
- Read the approved feature requirements and the specific card being worked on.
- Do not load the entire AI Development Team repository for ordinary implementation, bug fixes, or project-specific code reviews.
- Consult only the relevant central guide when a shared team rule is needed—for example, role boundaries, independent QA, workflow transitions, branching, or handoff requirements.
- Use the central requirements template only when creating a new requirements artifact.

### Sources of truth

| Concern | Source of truth |
|---|---|
| Project architecture, coding conventions, build/run instructions, environments, and technical decisions | Individual project repository |
| Feature behavior, scope, and acceptance criteria | Approved feature requirements document |
| Shared team roles, governance, quality gates, and collaboration principles | AI Development Team onboarding standard |
| Shared workflow states, handoff rules, and branching conventions | AI Development Team workflow standard, as adopted by the project |
| Task status, discussion, assignments, and handoff activity | Project work tracker |
| Actual code and review history | Project repository, commits, and pull requests |

A project's own instructions may link to the central standards repository, but that link does not mean every agent must retrieve those documents on every task. Project-specific instructions remain authoritative for project-specific technical matters. If a project intentionally adopts a different workflow or branching policy, record that variation in the project repository and follow the approved project policy.

**Operating principle: project context by default; team context only when needed.**
