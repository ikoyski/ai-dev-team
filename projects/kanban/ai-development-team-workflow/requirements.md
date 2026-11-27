# Feature Requirements — Kanban Support for AI Development Team Workflow

> This document proposes improvements to the existing Webtools Kanban application so it can coordinate the team's established human-and-AI software delivery workflow. It is the requirements baseline for related Kanban cards and pull requests once approved by the Human Team Lead.

## Metadata

- **Project:** kanban
- **Repository:** https://github.com/ikoyski/kanban (frontend); related services include the Kanban backend and Webtools Kanban MCP
- **Feature:** ai-development-team-workflow
- **Status:** In Review — Team Lead approval pending
- **Prepared by:** ChatGPT — Chief Architect / Requirements Author
- **Product owner:** Human Team Lead
- **Created:** 2026-11-26
- **Last updated:** 2026-11-27
- **Related Kanban cards:** To be created by Claude.ai after this plan is reviewed
- **Related PRs:** [ai-dev-team PR #1](https://github.com/ikoyski/ai-dev-team/pull/1)
- **Related architecture decisions:** Preserve the existing workflow states; use a per-board opt-in development workflow profile; keep human merge, test-acceptance, and production-promotion decisions restricted to the authenticated Team Lead acting through the Kanban web application; bind review and acceptance evidence to exact PR-head revisions; deliver the manual workflow MVP before optional summaries and filters.

## 1. Problem statement

The Kanban application already supports multi-board collaboration, Owner/Editor/Viewer permissions, cards with priorities and due dates, labels, rich-text descriptions and comments, archive/restore, search, and AI assistant access through the separate Kanban MCP service.

The AI development team has since established a more explicit delivery process involving a human Team Lead, a project coordinator, developers, independent QA reviewers, senior code review, test integration, human acceptance, and separately authorized production promotion.

The current general-purpose card model and collaboration interactions do not yet make all of that process easy to represent or verify. In particular, the team needs clearer ownership for developer and independent QA roles, dependencies, acceptance criteria, handoff evidence, review checkpoints, and links between requirements, Kanban work, GitHub changes, and deployed revisions.

Without these capabilities, important context is likely to remain in free-form comments or separate conversations. This makes coordination harder for both people and AI agents, and increases the risk of inaccurate status reports, missed handoffs, or work progressing without the required evidence.

## 2. Objective and desired outcome

Evolve Kanban into a workflow-aware collaboration tracker that remains useful as a general-purpose Kanban application while explicitly supporting the team's AI-assisted software delivery process.

The pilot should demonstrate that the team can take an approved requirements document, decompose it into cards, assign separate developer and QA responsibilities, implement changes on feature branches, conduct independent QA and senior review, integrate to the test environment, obtain human acceptance, and track separately authorized production release — with traceable evidence at each step.

Success means:
- Every implementation card has an accountable developer and a different named QA reviewer before it enters Ready.
- Cards can link to the approved requirement document and identify the relevant requirement and acceptance-criteria IDs.
- Dependencies, blockers, handoffs, review results, and test evidence are visible on the card.
- GitHub branches, PRs, commits, and deployment revisions can be associated with the work.
- The existing workflow states are retained; review/deployment substages are represented as card metadata and activity, not additional mandatory columns.
- AI assistants can perform supported coordination actions through MCP without bypassing the user's existing Kanban permissions or claiming unverified actions.
- Human approval gates remain explicit and cannot be inferred from an AI review PASS or a status change alone.

## 3. Background and current behavior

For this project, the Human Team Lead is the human account holder who retains final authority. The proposed authorization role mapping is the board Owner role, subject to Task 1 verifying the backend's actual role checks and confirming that this mapping is sufficiently narrow. ChatGPT is a distinct Chief Architect / Senior Technical Reviewer participant, not a DevN developer and not Claude.ai; its reviews must be attributed to ChatGPT. If ChatGPT cannot authenticate as a separate Kanban actor, record its review as a separately attributed review artifact rather than recording it as an action by another participant.

The current application includes:
- Multiple boards, board membership, and Owner/Editor/Viewer roles.
- Columns and draggable cards.
- Card priority, due date, labels, rich-text description, and rich-text comments.
- Card archive/restore, board search, board JSON import/export, and responsive UI.
- A REST API backed by PostgreSQL.
- A separate stateless Webtools Kanban MCP service through which compatible AI clients can list boards and manage supported board/card/comment operations under the user's authorization.

The frontend repository is `ikoyski/kanban`. Authentication and persistence are provided by separate services; MCP is also a separate service. Feature implementation must inspect the corresponding backend and MCP repositories before deciding where schema, API, or tool changes belong. Do not place backend or MCP responsibilities in the browser frontend.

The team's standard delivery workflow is:

```text
Backlog -> Ready -> In Progress -> In Review
                         ^             |
                         |             +-> Revision -> In Progress
                         |             |
                         +-------------+ (after fixes)
                                       |
                                       v
                                 Ready to Merge -> Done

Any active state may become Blocked and return to the appropriate state when resolved.
```

The meanings of the important review stages are:
1. Developer implements on a feature branch created from the latest `develop`, pushes commits, and opens a PR targeting `develop`.
2. A different agent independently performs QA against the approved requirements and actual PR/commit.
3. ChatGPT performs senior technical review after independent QA passes. Both QA and senior-review evidence identify the exact PR head commit SHA reviewed.
4. Only the Human Team Lead, using the authenticated human account through the Kanban web application, may authorize merging a specific PR head SHA into `develop`. The authorization records that exact SHA and becomes stale if the PR head changes before merge. This decision cannot be created, changed, or reversed through MCP, including when an MCP caller supplies a Team Lead name or other attribution metadata.
5. The test environment is deployed from the resulting `develop` commit and the Team Lead performs manual acceptance testing.
6. The Team Lead records ACCEPTED or REJECTED for the exact tested revision through the web application. After acceptance, the card may move to Ready to Merge.
7. The Team Lead separately authorizes production promotion to `master` through the web application for a specific release commit SHA. The authorization records that exact SHA and becomes stale if the release candidate changes. Done means the authorized production release and required verification are complete.

The application must not reinterpret these states or weaken the team's Git and release controls.

## 4. Proposed behavior

### 4.1 Workflow-aware cards

Cards should support structured delivery metadata in addition to their existing title, description, priority, due date, labels, and comments:
- Work item type (for example: Feature, Bug, Task, Refactor, Documentation, Test/QA).
- Developer / implementation owner.
- Independent QA reviewer.
- Requirement document URL and relevant requirement/acceptance-criteria identifiers.
- Dependencies on other cards, with a visible blocked-by / blocks relationship.
- GitHub repository, feature branch, pull request, and relevant commit references.
- Test deployment environment/URL and deployed commit reference when applicable.
- Explicit review and acceptance checkpoint outcomes.

Participant assignment is responsibility metadata, not a grant of application access. Active DevN instances in the team's developer roster must map to their own distinct authenticated Kanban user identities so actions can be attributed to the correct developer; an agent/client label alone is not an identity. The catalog may also represent human or external contributors who do not sign in, where appropriate. Board membership and Owner/Editor/Viewer permissions remain separate and authoritative for access.

### 4.2 Review and handoff tracking without extra columns

Keep the existing workflow columns/states. Within In Review, show the current checkpoint and its status, such as:
- Awaiting independent QA / QA in progress / QA passed / QA failed.
- Awaiting senior review / senior review passed / changes requested.
- Awaiting Team Lead authorization to merge into develop.
- Awaiting test deployment / test deployment verified.
- Awaiting Team Lead acceptance / accepted / rejected.
- Awaiting Team Lead production-promotion authorization / production release verified.

Represent each handoff as an attributable activity entry with a timestamp, checkpoint, outcome, summary, and relevant evidence links. A normal comment should remain available for discussion; structured handoff entries should make important transitions easy to scan and query.  Workflow evidence must identify the exact PR head SHA to which it applies. When that SHA changes, prior evidence remains historical and must not automatically satisfy gates for the new revision. For Phase 1 manual authorization, the web confirmation screen must prominently display the exact SHA being authorized and require the Team Lead to affirm that it matches the current GitHub PR head or release candidate before submission. The application must clearly warn that this is a human comparison, not automated GitHub verification. GitHub branch protection, required checks, and dismissal of stale approvals remain the hard repository-side controls; Kanban's manual confirmation is not a substitute for them.

### 4.3 GitHub and deployment traceability

Allow cards to record the relevant GitHub and deployment references. The first release may use explicit user/agent entry or permitted MCP updates for non-authoritative repository, PR, commit, and deployment references; protected Team Lead authorization and acceptance decisions are never MCP-editable. Automatic synchronization with GitHub or deployment providers is a later enhancement.

The card should make it possible to answer:
- Which requirement does this task implement?
- Which branch and PR contain the implementation?
- Which exact commit was reviewed and QA-tested?
- Which commit is running in the test environment?
- Was human acceptance recorded for that test revision?
- Which production release contains the accepted change?

Do not imply that a PR is merged, a deployment succeeded, or a release is complete merely because a URL or status value was entered. Where feasible, later integrations should verify these facts from their source systems.

### 4.4 Safe AI/MCP coordination

Expose the new structured card fields, dependency relationships, and handoff/checkpoint operations through the separate Kanban MCP service where appropriate. MCP tools must continue to act through the authenticated user's existing permissions and backend authorization.

AI agents should be able to perform ordinary coordination and implementation-workflow operations only after the operation is confirmed by the API. MCP may read human decisions and may perform permitted routine updates, but it must not create, edit, or reverse: (a) Team Lead authorization to merge a PR into `develop`, (b) Team Lead ACCEPTED/REJECTED test decisions, or (c) Team Lead authorization for production promotion. Those decision actions are available only through the Kanban web application to the authenticated Team Lead account. The backend must enforce this restriction using the authenticated actor and a positive allowlist of trusted web OAuth client IDs. For the current Kanban deployment, only the registered `kanban-web` client is eligible to invoke these protected decision actions. MCP client IDs (including `kanban-claude` and `kanban-antigravity`) and any unrecognized/new client IDs must be denied by default. Do not implement this as a denylist of known MCP clients. The backend must validate the client ID from a cryptographically verified access-token claim or equivalent trusted server-side authentication context; never trust an MCP-supplied header or caller-provided client attribution. Task 1 must verify the actual token claim and current Auth Server behavior.

The system should distinguish:
- The authenticated account that performed the API operation.
- The authenticated OAuth client/application context through which the operation was made.
- Optional client/agent attribution supplied for operational context.
- The person or agent assigned responsibility for the work.

Client-supplied agent labels are descriptive metadata, not proof of identity. Do not treat an AI agent name as a trusted identity or allow it to elevate permissions.

## 5. Functional requirements

### A. Work ownership and task definition

1. **FR-1 — Work item type:** A card shall support a configurable or predefined work item type while preserving existing card behavior and data.
2. **FR-2 — Developer assignment:** A card shall identify one primary developer responsible for implementation.
3. **FR-3 — Independent QA assignment:** A card shall identify one QA reviewer. The application shall flag or prevent the same participant from being assigned as both developer and independent QA for the same card.
4. **FR-4 — Participant catalog:** The application shall support a participant catalog for assigning human and AI contributors. Each active DevN instance in the team's developer roster shall map to its own distinct authenticated Kanban user identity for reliable actor attribution. The catalog may also represent external/non-login contributors where appropriate. Participant assignment shall not itself grant board access or permissions; authentication and board membership remain separate.
5. **FR-5 — Requirement traceability:** A card shall store a link to its authoritative requirements document and allow references to requirement IDs and acceptance-criteria IDs.
6. **FR-6 — Acceptance criteria:** A card shall support a structured, checkable list of task-level acceptance criteria, with completion state and optional evidence/comment.
7. **FR-7 — Dependencies:** A card shall support links to prerequisite/blocking cards and display both its blockers and the cards it blocks. The system shall prevent self-dependencies and detect or reject dependency cycles.
8. **FR-8 — Blocker details:** When a card is Blocked, the card shall make the blocker reason, responsible party, impact, and next action visible. Clearing a blocker shall not silently mark the task complete.

### B. Review, handoff, and acceptance

9. **FR-9 — Review checkpoint:** A card in In Review shall record its current review/deployment checkpoint without requiring additional workflow columns.
10. **FR-10 — QA outcome:** The assigned independent QA reviewer shall be able to record PASS or FAIL against a specific PR/commit, including checks performed, findings, limitations, and evidence links.
11. **FR-11 — Senior review outcome:** The card shall record the senior review outcome and the exact revision reviewed. A PASS shall not itself authorize a merge.
12. **FR-12 — Team Lead authorization:** The card shall distinguish a Team Lead's explicit authorization to merge a specific PR head SHA into `develop` from AI review results and general comments. The authorization shall record the PR identifier and exact head SHA, may be created, changed, or reversed only by the authenticated Team Lead through the Kanban web application, and becomes stale if the PR head SHA changes before merge. MCP clients shall not have a tool or mutation path for this decision.
13. **FR-13 — Test deployment evidence:** The card shall record the test environment, deployed commit SHA, deployment time/result, and verification evidence where applicable.
14. **FR-14 — Human acceptance:** The Team Lead shall be able to record ACCEPTED or REJECTED against the tested revision, with notes and evidence, only through the Kanban web application while authenticated as the Team Lead. MCP clients shall not be able to create, change, or reverse this decision. Only an accepted test result for the current eligible revision may qualify the card for Ready to Merge.
15. **FR-15 — Production release evidence and authorization:** The card shall record the Team Lead's explicit production-promotion authorization bound to the exact release candidate commit SHA, the production release reference, and verification evidence before it is considered Done. The authorization decision shall be recorded only through the Kanban web application by the authenticated Team Lead and becomes stale if the release candidate SHA changes. MCP may read the decision and may record permitted non-authoritative delivery evidence but shall not create or alter the authorization.
16. **FR-16 — Structured handoff:** A participant shall be able to record a handoff containing the sender, intended recipient/role, checkpoint, summary, requested next action, outcome, and supporting links. The system shall retain the handoff in the card's activity history.
17. **FR-17 — Revision cycle:** When a review fails or changes are requested, the card shall record findings and return to the original developer for correction. Subsequent review evidence shall identify the new PR/commit revision.
18. **FR-18 — Activity history:** The card shall provide a chronological, append-oriented history of important events, including status changes, assignments, dependency changes, review outcomes, handoffs, approval decisions, and recorded deployment/release evidence.

### C. GitHub and delivery references

19. **FR-19 — GitHub references:** A card shall support repository, branch, PR, and commit references. A task may have more than one commit and may need to record the exact commit reviewed or deployed.
20. **FR-20 — Deployment references:** A card shall support test and production deployment references separately, including environment, URL, revision, and verification status.
21. **FR-21 — Evidence clarity:** Manually entered references and assertions shall be visibly distinguishable from facts verified by a future integration.

### D. Workflow and MCP support

22. **FR-22 — State validation:** The application shall validate required card metadata before allowing a card to enter Ready. At minimum, an implementation task requires clear scope/acceptance criteria, resolved dependencies, a developer, and a different QA reviewer.
23. **FR-23 — In Review validation:** The application shall require a reviewable implementation reference (normally a PR targeting `develop`) before a task enters In Review, while allowing documented exceptions for task types that do not produce code.
24. **FR-24 — Ready to Merge validation:** A code task shall not qualify for Ready to Merge until independent QA and senior review have passed for the applicable PR head SHA, the Team Lead's web-recorded authorization to merge that specific PR into `develop` is recorded, the resulting test deployment is verified, and the Team Lead's human acceptance is recorded through the web application for that exact tested revision.
25. **FR-25 — Done validation:** A code task shall not qualify for Done until the separately authorized production promotion and required production verification are recorded. Non-code task types may use an appropriate completion rule.
26. **FR-26 — MCP workflow support:** Supported MCP operations shall be able to read and update permitted structured work fields, dependencies, developer/QA assignments, handoffs, and non-human review/checkpoint evidence, subject to backend authorization and validation. MCP shall be able to read human approval decisions but shall not create, change, or reverse Team Lead merge authorization, human test acceptance/rejection, or production-promotion authorization.
27. **FR-27 — Confirmed operations:** MCP responses and agent handoffs shall reflect the actual API result. Failed, rejected, or unconfirmed writes must not be reported as successful.
28. **FR-28 — Existing permissions and human-only decision enforcement:** All new web and MCP operations shall enforce existing board roles and server-side authorization. The backend shall additionally restrict Team Lead merge authorization, human test acceptance/rejection, and production-promotion authorization to the authenticated Team Lead acting through the Kanban web application. Requests through MCP clients shall be rejected for these decision mutations even if the caller supplies Team Lead attribution data. UI-only restrictions are insufficient.
29. **FR-29 — General-purpose operation:** The features shall remain usable for ordinary Kanban projects. AI-specific participants and review checkpoints shall be optional unless a board explicitly enables the development workflow profile.

### E. Usability and administration

30. **FR-30 — Workflow profile:** A board owner shall be able to enable a software-development workflow profile that exposes the relevant fields and validation rules while leaving ordinary boards lightweight.
31. **FR-31 — Card summary (Phase 2):** Card list views should provide a compact indication of owner, QA reviewer, blockers, current review checkpoint, and key delivery status without making cards visually overwhelming. This is deferred from the Phase 1 pilot.
32. **FR-32 — Search and filtering (Phase 2):** Users should be able to filter/search cards by developer, QA reviewer, work item type, dependency/blocker status, review checkpoint, and relevant delivery status. This is deferred from the Phase 1 pilot.
33. **FR-33 — Responsive experience:** New fields and checkpoint/history views shall remain usable on mobile and desktop.
34. **FR-34 — Import/export:** Board JSON export/import shall preserve new structured fields and relationships, with validation and safe handling of unknown or older fields.

## 6. Non-functional requirements

- **NFR-1 — Backward compatibility:** Existing boards and cards without new metadata shall continue to load and operate. Database migrations and API changes shall be backward-compatible where practical.
- **NFR-2 — Authorization:** Backend authorization remains authoritative for web and MCP requests. No new field or workflow profile may bypass Owner/Editor/Viewer permissions.
- **NFR-3 — Audit integrity:** Important history events shall not be silently overwritten as ordinary editable comments. Corrections should be represented by subsequent events or an explicitly audited correction mechanism.
- **NFR-4 — Attribution:** Store the authenticated actor and timestamp for system-recorded actions. Optional agent/client attribution must be clearly marked as unverified descriptive metadata unless independently authenticated.
- **NFR-5 — Data integrity:** Dependency relationships shall be validated for existence, board boundaries, self-reference, and cycles. QA, senior-review, deployment, human-acceptance, merge-authorization, and production-authorization evidence shall reference the exact PR head SHA or deployed/release revision to which it applies. A changed revision invalidates prior gate evidence and authorizations by default; only the documented independent QA and senior-review mechanical-rebase delta-check paths may issue fresh review evidence for the new SHA. A new explicit human authorization is always required for a changed merge or production-release SHA.
- **NFR-6 — Security:** Validate and safely render all user- and agent-supplied text/URLs. Continue using the existing rich-text sanitization boundary. Do not store access tokens, OAuth secrets, or other credentials in card metadata or activity entries.
- **NFR-7 — Reliability:** Failed writes must leave the user with a clear error and must not display optimistic success as confirmed completion. Preserve retry-safe behavior.
- **NFR-8 — Performance:** Board loading and card rendering must remain responsive for boards with substantially more activity and relationship metadata than today. Avoid loading an unbounded full audit history for every card in the board view.
- **NFR-9 — Maintainability:** Keep frontend, backend, and MCP responsibilities in their existing service boundaries. Prefer explicit API contracts and focused modules over duplicating workflow rules across clients.
- **NFR-10 — Accessibility:** New controls, status indicators, dialogs, and history views shall support keyboard navigation and accessible labels/announcements.
- **NFR-11 — Provider neutrality:** Do not hard-code behavior to Claude, ChatGPT, Gemini, Antigravity, or a specific model provider. Participant names and integrations should be configurable and provider-neutral.
- **NFR-12 — Revision-bound QA evidence:** Every QA outcome shall identify the exact PR head SHA tested. A changed SHA makes the previous QA result stale by default. Only an independently performed and documented mechanical-rebase delta-check may establish a fresh PASS for the new SHA without repeating the full QA suite.

## 7. Scope

### In scope — Phase 1 pilot

- A per-board, opt-in software-development workflow profile.
- Structured developer and independent QA assignments.
- Requirement and acceptance-criteria references.
- Task acceptance criteria and dependencies.
- Review checkpoints and structured handoffs.
- Explicit human authorization and acceptance records.
- GitHub branch/PR/commit and deployment references.
- Card activity history for important workflow events.
- Web UI, backend/API, and Kanban MCP support for the agreed features.
- Board import/export compatibility and regression tests.
- A pilot using this Kanban improvement project and subsequent real AI-team tasks.

### Deferred to Phase 2

- FR-31 compact workflow-aware card-list summaries.
- FR-32 dedicated workflow filters for developer, QA reviewer, work type, dependency/blocker, checkpoint, and delivery status.

### Out of scope

- Replacing GitHub as the source of truth for source code, commits, PRs, or CI checks.
- Replacing Render or other deployment platforms.
- Automatically merging PRs, pushing to `develop`/ `master`, or promoting production from Kanban.
- Allowing AI agents to authorize their own merges or production releases.
- Making Kanban the identity provider or creating a new AI-agent authentication system.
- Treating a participant label as a user account, board membership, or permission grant.
- Mandatory extra columns for QA, senior review, deployment, or human testing.
- Provider-specific AI agent orchestration, model selection, prompt management, or chat features.
- Full two-way GitHub/Render synchronization in the initial pilot.
- Replacing existing board roles or changing the central Webtools authentication architecture.

## 8. Technical design and constraints

This section sets architectural constraints, not a prescriptive implementation. The implementation team must inspect the current frontend, backend, database migrations, API contracts, and MCP tools before proposing detailed changes.

1. **Service boundaries:** The browser frontend remains a client. Persisted fields, validation, dependency integrity, authorization, and authoritative activity records belong in the backend. MCP exposes supported operations by calling the backend; it must not become a second source of workflow truth.
2. **Workflow profile:** Prefer an opt-in board-level profile/configuration so the existing general-purpose Kanban experience remains simple. Profile settings should define which fields and transition validations apply.
3. **Participant representation:** Design a participant reference/catalog that supports authenticated board members, including the team's active DevN instances, each mapped to its own distinct Kanban user identity for actor attribution. It may also represent named external/non-login contributors where appropriate. The participant reference itself is not an access-control grant; board membership and permissions remain authoritative.
4. **History model:** Consider a dedicated structured activity/event representation for important workflow events instead of encoding every checkpoint as free-form rich-text comments. Preserve ordinary comments for discussion.
5. **Revision-aware evidence:** QA, senior review, deployment, and acceptance records must identify the exact PR/commit or revision they apply to. If the implementation changes, stale evidence must not silently satisfy gates for the new revision.
6. **Dependency model:** Dependencies should be explicit card relationships within a board, with clear behavior for archived/deleted cards and cycle prevention.
7. **API and MCP:** Extend backend DTOs/endpoints and MCP tools only as needed. Keep authorization and business validation centralized in the backend. MCP tool descriptions should explain required fields, side effects, and confirmation behavior.
8. **Migration:** Existing cards should receive safe defaults (for example, no workflow profile and empty optional metadata). Avoid destructive resets or requiring users to recreate boards/cards.
9. **Incremental delivery:** Deliver a useful manual-tracking MVP first. Add source-system synchronization only after the underlying workflow data model and manual process are proven.
10. **No premature automation:** A manually recorded Team Lead decision is an auditable workflow record, not an automatic GitHub permission or merge action. Any future integration must still respect GitHub branch protections and the explicit authorization model.
11. **Human decision channel:** Merge authorization, test ACCEPTED/REJECTED, and production-promotion authorization are web-application-only decisions. The backend must positively allow only the registered `kanban-web` OAuth client for these mutations and deny every other or unknown client ID by default, regardless of caller-supplied identity or agent metadata. This is an allowlist, not a denylist of known MCP clients.
12. **Revision and rebase policy:** The PR head SHA is the authoritative identifier for QA and review evidence. Any SHA change invalidates prior evidence by default. A delta-check is permitted only for a mechanical, conflict-free rebase where the effective PR patch is demonstrably unchanged, no conflict resolution or other step changes the implementation or contract, and relevant CI checks pass. The independent QA reviewer must document the old and new SHAs, the comparison performed, affected checks rerun, and issue a fresh QA result bound to the new SHA. If the effective patch changes, shared contracts/hot files are affected, or equivalence is uncertain, full QA is required.

## 9. Security and privacy considerations

- Continue to use the existing Webtools authentication and backend authorization model.
- Validate all mutations on the backend, including those initiated through MCP.
- A board role (Owner/Editor/Viewer) controls what a signed-in user may do. Developer/QA assignment does not grant access or additional permissions.
- Record the authenticated user/account that made an action. If an AI client name is also recorded, treat it as client-supplied metadata unless there is a verified identity mechanism.
- Do not expose private repository details, PR links, deployment URLs, or activity history to users who lack access to the board.
- Prevent stored XSS through safe rendering and URL validation; continue sanitizing rich-text content.
- Avoid storing secrets, bearer tokens, environment variables, or sensitive deployment configuration in cards, comments, or handoff records.
- Consider abuse cases where an AI client attempts to mark its own QA as passed, impersonate the Team Lead, fabricate deployment evidence, or move a card past a gate. Server-side rules and audit attribution must prevent an untrusted client assertion from becoming privileged authorization.
- The backend must reject every protected decision mutation unless both the authenticated human actor is the authorized Team Lead and the cryptographically verified OAuth client ID is on the explicit trusted web-client allowlist (currently `kanban-web`). All MCP clients and unknown/future client IDs are denied by default. This is positive allowlisting, not a denylist of known MCP client IDs. Task 1 must verify the signed client claim and issuer validation path.
- The Team Lead's approval must be attributable to the authenticated human account and explicit action. AI review results are advisory and must never be accepted as human approval.

## 10. Dependencies and sequencing

### Wave 0 — Architecture and contract freeze (Task 1; sequential)

1. Inspect the actual frontend, backend, database migrations, API contracts, and MCP tools; confirm the proposed repository/service split rather than relying on assumptions.
2. Resolve the technical design for task-level acceptance criteria (dedicated entity versus structured card field), the participant catalog (including DevN and distinct ChatGPT reviewer attribution), and the minimum checkpoint types. Verify signed OAuth client-context claims from the Auth Server and the backend's Team Lead/Board Owner role enforcement; document any Auth Server gap separately without expanding scope silently.
3. Define the per-board workflow-profile configuration, dependency and activity models, revision-bound evidence, human-only web decision operations, and MCP read/write boundaries.
4. Publish the API contract, MCP tool signatures, migration/version allocation plan, dependency graph, task-level acceptance criteria, and file-ownership map. Identify hot files and specify which task owns each one.
5. Claude.ai reviews the design. No implementation begins until the contract and ownership map are stable.

### Wave 1 — Backend foundation (Tasks 2 and 3; sequential integration)

6. Implement persistence and additive migrations, then server-side workflow validation and authorization. Use one primary writer for shared backend hot files and merge one PR at a time where changes overlap. Do not start a dependent task from an unmerged branch that changes the same contract or files.
7. Implement and test the trusted web-client/Team Lead restriction for protected approval decisions before exposing those actions in the UI or MCP.

### Wave 2 — Client and transfer capabilities (after backend contract is merged to develop/test)

8. Implement frontend workflow editing, checkpoints, history, and GitHub/deployment references against the frozen contract. Keep task-level UI acceptance criteria and tests within each implementation card.
9. Implement MCP read/write support only for the permitted operations; protected human decisions are readable but not mutable through MCP.
10. Implement Phase 1 import/export compatibility, including preservation of all newly supported fields and relationships. FR-31 and FR-32 are deferred to Phase 2.
11. Parallel work is allowed only when file ownership and dependency boundaries show no overlapping writers. Review each PR's touched-file list against other open PRs and integrate shared-repository changes one PR at a time.

### Wave 3 — Cross-cutting verification and pilot (Tasks 9 and 10)

12. Task 9 focuses on cross-cutting security, authorization, migration/integration regression, end-to-end workflow, and release-gate verification. It must not be used as a substitute for feature-level tests and acceptance criteria in Tasks 2–8.
13. Deploy the accepted `develop` revision to the test environment, then run the pilot. Human acceptance and production-promotion decisions remain separate Team Lead actions.

### Conflict and QA evidence controls

- One active writer per hot file. The Wave 0 ownership map must identify owners for shared UI entry points, card modal/rendering code, backend card/entity/DTO/service/controller/auth files, MCP tool declarations, and migration versions.
- Reserve additive Flyway migration versions in the technical plan; only the assigned backend migration owner may add or reorder migrations.
- Keep separate clones/worktrees and isolated runtime/test data where parallel agents could otherwise collide. Do not share mutable working directories or databases for concurrent migration work.
- Before opening/merging a PR, provide `git diff --name-only develop...HEAD`; compare it with all open PRs for overlapping paths and semantic/API dependencies.
- Do not branch a task from another unmerged feature branch by default. If a stacked dependency is unavoidable, document it explicitly and rebase/test after the prerequisite merges.
- Merge overlapping work into `develop` one PR at a time. Rebase against the latest `develop`, run CI, and perform the required QA evidence refresh before the Team Lead authorizes the merge.
- QA evidence is keyed to the exact PR head SHA. Full re-QA is the default after a SHA change. Only the documented mechanical-rebase exception permits an independent delta-check and fresh QA attestation for the new SHA.
- Keep the developer and QA reviewer different for every implementation card. A rebase delta-check is performed by the independent QA reviewer, not the developer.

The project must not enter Ready for implementation until the Team Lead approves this requirements revision, the architecture/data model is reviewed, and task dependencies/ownership are clear.

## 11. Acceptance criteria

- [ ] **AC-1:** Existing boards/cards created before the feature continue to load and support current operations without data loss.
- [ ] **AC-2:** A board owner can enable/disable the software-development workflow profile without changing the board's existing columns.
- [ ] **AC-3:** An implementation card can identify a developer and a different independent QA reviewer; the same participant cannot satisfy both roles.
- [ ] **AC-4:** A card can link to its approved requirements document and reference specific requirement and acceptance-criteria IDs.
- [ ] **AC-5:** A card can maintain task-level acceptance criteria and show completion state.
- [ ] **AC-6:** Users can create and view blocker/dependency relationships; invalid self-links and dependency cycles are rejected.
- [ ] **AC-7:** A Blocked card visibly records why it is blocked, who owns the next action, and what will unblock it.
- [ ] **AC-8:** In Review can show the current QA/review/deployment checkpoint without adding mandatory workflow columns.
- [ ] **AC-9:** QA and senior review outcomes identify the exact PR/commit reviewed and include a summary plus evidence/findings.
- [ ] **AC-10:** A Team Lead authorization to merge a specific PR head SHA into `develop` is recorded distinctly from reviewer PASS results. The authorization becomes stale if the PR head SHA changes before merge. The backend accepts the decision only from the authenticated Team Lead through the Kanban web application; equivalent MCP mutations are rejected.
- [ ] **AC-11:** Test deployment evidence identifies the deployed commit, environment, and verification result.
- [ ] **AC-12:** Human acceptance is recorded against the tested revision through the web application by the authenticated Team Lead. Rejected or stale acceptance evidence cannot qualify a newer revision for Ready to Merge, and MCP mutations attempting to create or alter the decision are rejected.
- [ ] **AC-13:** Production-promotion authorization is bound to the exact release candidate commit SHA; release evidence and verification are recorded separately. The authorization becomes stale if the release candidate SHA changes. Only the authenticated Team Lead through the web application can record the authorization; MCP cannot create or alter it. Kanban does not perform an unauthorized merge or deployment.
- [ ] **AC-14:** Important assignment, status, checkpoint, handoff, approval, and deployment events appear in chronological activity history with authenticated actor and timestamp.
- [ ] **AC-15:** Web and MCP operations enforce the same base backend permissions and validation rules, with the additional channel-specific restriction that MCP cannot mutate Team Lead approval decisions as detailed in AC-22.
- [ ] **AC-16:** MCP reports a write as successful only after the backend confirms it; errors and permission denials are surfaced accurately.
- [ ] **AC-17:** Board JSON export/import preserves all new supported metadata and relationships, with backward compatibility for older exports.
- [ ] **AC-18 (Phase 2):** Users can filter by developer, QA reviewer, work type, blocker/dependency, and checkpoint.
- [ ] **AC-19:** Automated tests cover backend validation/authorization, frontend behavior, MCP operations, migrations, and relevant regression paths.
- [ ] **AC-20:** The pilot completes at least one end-to-end task through requirement linkage, assignment, implementation, independent QA, senior review, test integration, human acceptance, and separately authorized production release, with evidence recorded on the cards.
- [ ] **AC-21:** QA PASS evidence is bound to an exact PR head SHA. A changed SHA invalidates the prior QA gate by default; a mechanical-rebase exception is accepted only when an independent QA reviewer documents patch equivalence, old/new SHAs, required CI reruns, and a fresh result for the new SHA. Any behavioral, API/contract, hot-file, or uncertain change requires full QA. Senior review is independently SHA-bound and follows its own full-review-by-default / documented mechanical-rebase delta-review policy; QA delta evidence cannot substitute for senior review.
- [ ] **AC-22:** MCP can read the Team Lead's merge, test-acceptance, and production-promotion decisions but cannot create, edit, or reverse them. Backend authorization rejects these operations from MCP clients even when the request supplies Team Lead identity or agent attribution fields. Merge and production authorizations are SHA-bound and become stale when their respective target SHA changes.
- [ ] **AC-23:** Before recording merge or production authorization, the web UI prominently displays the exact target SHA and requires the Team Lead to confirm that it matches the current GitHub PR head or release candidate. The UI clearly identifies this as a manual comparison; GitHub branch protection, required checks, and stale-approval dismissal remain the hard controls.

## 12. Testing requirements

- **Backend unit/integration tests:** participant assignment validation; distinct developer/QA; workflow profile behavior; transition gates; dependency validation and cycle prevention; revision-aware review/acceptance evidence; activity attribution; board-role authorization.
- **Database migration tests:** upgrade an existing database with populated boards/cards; verify defaults and preservation of existing records.
- **API tests:** create/read/update cards with new metadata; invalid payloads; stale revisions; unauthorized mutations; dependency edge cases; import/export compatibility.
- **Frontend Playwright tests:** enable/disable workflow profile; edit assignments and criteria; display dependencies and blockers; add handoffs/checkpoints; record eligible review/acceptance interactions; responsive interactions; old-board regression. Dedicated FR-31/FR-32 summary and filter tests are deferred with those features to Phase 2.
- **MCP tests:** verify new read/write tools and parameters; permission denial; invalid transitions; failed writes; accurate confirmation; authenticated actor attribution; no privilege escalation from participant labels.
- **Security tests:** stored XSS and unsafe URLs; unauthorized cross-board access; attempts to impersonate Team Lead approval; attempts to self-approve QA; attempts to bypass transition validation.
- **End-to-end pilot:** use a non-production board and test deployment. Preserve evidence for each checkpoint and verify the deployed commit matches the accepted revision.
- **Evidence:** PRs should include test commands/results, relevant CI links, migration notes, and screenshots or traces for meaningful UI flows. QA reports must state exact PR head SHA reviewed and any unverified areas.
- **Revision/rebase tests:** Verify that QA evidence for an earlier SHA cannot satisfy a gate for a newer SHA. Test the mechanical-rebase delta-check path, including recorded old/new SHAs, patch-equivalence evidence, rerun checks, and fresh QA attestation; verify that changed effective patches require full QA.
- **Human decision-channel tests:** Verify that authenticated Team Lead web actions can record the three protected decisions, while MCP requests cannot create, change, or reverse them, including requests that spoof Team Lead names, participant IDs, or client attribution.

Do not use production credentials or live production data in automated tests.

## 13. Deployment, migration, and rollback

- Deploy backend schema/API changes before frontend or MCP clients that require the new contract, while maintaining compatibility with existing clients.
- Use additive database migrations. Do not wipe or reset existing Kanban data.
- Deploy frontend and MCP changes only after compatible backend behavior is available in the test environment.
- Verify existing boards, cards, members, permissions, comments, archive/restore, import/export, and AI MCP operations after deployment.
- Record the exact `develop` commit and test deployment result on the pilot cards.
- If a migration or new workflow gate causes a regression, disable the workflow profile or roll back the affected service using the established deployment procedure. Do not delete user data to recover.
- Production promotion remains a separate Team Lead-authorized release after test acceptance.

## 14. Suggested task breakdown

Claude.ai should review this plan, identify sensible task boundaries, dependencies, and estimates, then create cards on the appropriate Kanban project board. Assign one developer and a different QA reviewer to each implementation card. Cards should initially remain in Backlog until the Team Lead approves this requirements revision and all Ready prerequisites are satisfied.

| Task group | Description / acceptance focus | Suggested sequencing |
|---|---|---|
| 1. Architecture and data model | Inspect frontend/backend/MCP; define workflow profile, participant references, dependencies, checkpoints, activity events, revision-aware evidence, and API contracts. Produce a reviewed technical design. | First; blocks implementation |
| 2. Backend persistence and migrations | Add backward-compatible persistence for workflow profile and structured card metadata, participants, dependencies, and activity records. | After task 1 |
| 3. Backend workflow rules and authorization | Implement server-side validation for assignments, dependencies, state gates, review/acceptance evidence, and role enforcement, including the MCP/web decision boundary. | After Task 2; sequential integration |
| 4. Frontend card workflow experience | Add workflow profile settings, structured card fields, criteria, dependencies, checkpoint display, handoffs, history, and delivery references. Include feature-level tests and acceptance criteria. | Wave 2, after API contract stabilizes |
| 5. GitHub/deployment reference UX | Add explicit repository/branch/PR/commit and deployment references, distinguishing manual records from verified integration data. Include feature-level tests and acceptance criteria. | Wave 2, alongside Task 4 only where file ownership permits |
| 6. MCP workflow support | Extend MCP read/write operations for permitted fields, dependencies, checkpoints, and handoffs; expose human decisions read-only and verify protected mutations are rejected. Include tool-level tests. | Wave 2, after backend contract |
| 7. Search, filters, and board summaries | Deliver FR-31 summaries and FR-32 workflow filters. | Phase 2 only; does not block Phase 1 pilot acceptance |
| 8. Import/export and compatibility | Preserve all new Phase 1 metadata and relationships in JSON transfer and validate older imports/exports. Include migration/import regression tests. | Wave 2, after data model |
| 9. Cross-cutting regression, security, and E2E verification | Verify integration across repositories/services, human-only decision enforcement, revision-aware gates, migration safety, security, and full lifecycle behavior. Feature-level tests remain part of Tasks 2–8. | Wave 3; throughout where appropriate |
| 10. Pilot deployment and workflow validation | Deploy to test, run a real AI-team task through the full lifecycle, record evidence, collect feedback, and resolve pilot gaps. | Wave 3, after implementation and QA |

The coordinator should split these groups into appropriately sized cards. Avoid one oversized card per entire group if the work can be independently implemented and reviewed. Each card must link to this requirements document and identify the applicable FR/AC references. Developer and QA assignments must be different.

## 15. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Over-customizing Kanban for one team | Makes ordinary boards complicated | Use an opt-in workflow profile; keep current behavior as the default |
| Workflow logic duplicated across frontend and MCP | Inconsistent validation or permission bypass | Centralize authoritative rules in the backend |
| AI participant labels treated as trusted identity | Impersonation or false approvals | Separate assignment metadata from authenticated actor and authorization |
| Review evidence becomes stale after new commits | Unreviewed code may pass a gate | Bind outcomes to exact PR/commit revisions and invalidate stale evidence |
| Too much information on cards | Poor usability, especially on mobile | Keep card summaries compact; put detailed evidence in a history/checkpoint view |
| Dependency cycles or deleted prerequisite cards | Work can become permanently blocked or misleading | Validate cycles and define deletion/archive behavior |
| GitHub/deployment facts entered manually are inaccurate | False status reporting | Label manual assertions; defer verification to later source integrations |
| Schema/API changes disrupt current clients | Existing boards or MCP integrations break | Additive migrations, compatibility tests, staged deployments |
| Scope becomes too large for a pilot | Delayed delivery and unclear value | Deliver manual workflow tracking first; defer full synchronization and automation |

## 16. Decisions and remaining design work

The following product/workflow decisions are agreed for this requirements baseline. The remaining Wave 0 items are technical design choices, not permission to change the agreed behavior.

| Decision | Agreed resolution | Follow-up |
|---|---|---|
| Workflow profile scope | Enable the software-development workflow profile per board; ordinary boards remain lightweight by default. | Task 1 defines the profile configuration and defaults. |
| Human-only approval channel and Team Lead role | Merge authorization, test ACCEPTED/REJECTED, and production-promotion authorization are web-application-only actions by the authenticated Team Lead. The board Owner role is the proposed authority mapping, but Task 1 must verify current backend role semantics and define the least-privilege Team Lead authorization check explicitly; board ownership alone must not be assumed to prove the required identity without source verification. MCP may read these decisions but may not create, edit, or reverse them. | Task 1 verifies and specifies the authenticated actor/board role enforcement. |
| QA after a rebase | Full re-QA is the default whenever the PR head SHA changes. A delta-check is allowed only for a mechanical, conflict-free rebase with demonstrably unchanged effective patch, no implementation/contract changes, and relevant CI checks passing. Independent QA must document old/new SHAs, patch comparison, checks rerun, and issue fresh QA evidence for the new SHA. Any semantic, contract, hot-file, or uncertain change requires full QA. | Task 1 defines the evidence representation; backend validation and QA procedures enforce revision binding. |
| Senior-review rebase policy | Senior review is bound to the exact PR head SHA. A changed SHA requires full senior re-review by default. A delta review is permitted only for a verified mechanical, conflict-free rebase with unchanged effective patch, no semantic/API/contract/hot-file changes, and passing relevant CI. ChatGPT must independently inspect the old/new revision comparison, record both SHAs and the rationale, and issue fresh senior-review evidence for the new SHA. QA delta evidence does not substitute for senior review. | Task 1 defines evidence and the review procedure; ChatGPT performs the senior-review decision. |
| Revision identity and authorization binding | The PR head commit SHA is authoritative for QA and senior review. Merge authorization is bound to the exact PR head SHA; production-promotion authorization is bound to the exact release commit SHA. Any change makes the corresponding authorization stale and requires fresh human authorization. Deployment and human acceptance evidence identify the exact deployed/tested revision. | Task 1 defines storage, stale-state behavior, and validation. |
| Phase 1 scope | Keep FR-31 card summaries and FR-32 workflow filters out of the Phase 1 pilot; defer them to Phase 2. Keep FR-34 in Phase 1 so all newly supported workflow data and relationships survive import/export. | Coordinator labels/schedules Phase 2 work separately; Task 8 covers Phase 1 data preservation. |
| Release sequencing | Backend persistence, validation, and API contract must be integrated before dependent frontend/MCP work. Parallel work is permitted only with explicit file ownership and no conflicting writers; shared files/contracts are integrated sequentially. Cross-cutting E2E/security verification follows core integration. | Task 1 produces the dependency graph, file-ownership map, and migration allocation plan. |
| Task-level tests | Every implementation card includes its own tests and acceptance criteria. Task 9 is reserved for cross-cutting security, integration, regression, and E2E verification rather than tests omitted by earlier tasks. | Claude.ai applies this when decomposing the task groups into cards. |
| Task 1 ownership and review | ChatGPT is the architecture/design owner; Claude Code performs independent technical QA against the actual repositories and design. Claude.ai coordinates and reviews requirements traceability, task sequencing, ownership, and handoff readiness; it does not duplicate the technical QA. | Coordinator records the pairing on the Task 1 Backlog card; the Team Lead approves the task before it enters Ready. |
| Task acceptance-criteria representation | The product requirement is settled: criteria are structured, checkable, traceable, and revision/evidence-aware. Whether to store each criterion as a dedicated entity or as a structured field remains an implementation design choice. | Resolve in Task 1 before migrations are allocated. |
| Participant catalog | Active DevN instances use distinct authenticated Kanban user identities for audit attribution. The catalog may also represent appropriate external/non-login contributors. Assignment never grants board access or authority. | Resolve the participant data model and UX in Task 1; align with the team's DevN identity standard. |
| Checkpoint types | The required workflow stages are QA, senior review, Team Lead merge authorization, test deployment/verification, Team Lead acceptance/rejection, and production authorization/release verification. The exact extensible type taxonomy and stored representation are technical design choices. | Resolve in Task 1 before API/MCP contracts are frozen. |
| GitHub fact verification | Phase 1 may record repository/branch/PR/commit and deployment details manually, but must label manual assertions distinctly from verified facts. | Task 1 confirms current integration permissions; automatic verification remains a later enhancement. |
| Activity history | Important workflow and approval events are append-oriented and auditable. Corrections are represented by subsequent events or an audited correction mechanism, not silent edits/deletions. | Task 1 defines event model and permitted correction behavior. |

The default implementation allocation for this pilot is Dev1 (Claude Code) for backend and MCP work, and Dev2 (Antigravity) for frontend work; each task must still name a different independent QA reviewer, and the coordinator may adjust assignments for capacity or access while preserving that separation. Use the standardized `dev:devN` and `qa:devN` labels for identity-based assignment; tool-family labels are supplemental only. The coordinator may reuse existing labels. Creating missing labels is a tracker-administration action and must be explicitly authorized by the Team Lead before execution.

Task 1 must inspect the actual `kanban`, backend, MCP, and authentication-service behavior before finalizing the technical design. In particular, verify whether the access token contains a cryptographically signed, trusted OAuth client identifier (or equivalent claim) that the backend can validate. Never trust an MCP-supplied header or caller-provided client attribution as proof of client context. If the current Auth Server does not issue a suitable signed claim, document the gap and propose a separate, explicitly scoped Auth Server change; do not silently expand this project's scope. Also verify how the backend identifies the authenticated Team Lead and whether Board Owner is the correct least-privilege role mapping. The repository/service split, touched-file ownership, migration versions, API contracts, and MCP signatures must be confirmed from source rather than assumed from this requirements document.

## 17. Approval

- **Team Lead decision:** Pending — Approved / Changes requested / Rejected
- **Approved by:** <human approver>
- **Date:** <YYYY-MM-DD>
- **Approved revision/commit:** <commit SHA or link>
- **Notes:** This document is a proposed requirements baseline. Claude.ai may review it and prepare Backlog cards, but implementation cards must not enter Ready until the Team Lead approves the requirements and task prerequisites are satisfied.
