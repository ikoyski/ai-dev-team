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
- **Last updated:** 2026-11-26
- **Related Kanban cards:** To be created by Claude.ai after this plan is reviewed
- **Related PRs:** None
- **Related architecture decisions:** Preserve the existing workflow states; project context by default; human retains final acceptance and release authority.

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
3. ChatGPT performs senior technical review after independent QA passes.
4. Only the Human Team Lead authorizes the specific PR merge into `develop`.
5. The test environment is deployed from the resulting `develop` commit and the Team Lead performs manual acceptance testing.
6. After acceptance, the card may move to Ready to Merge. The Team Lead separately authorizes production promotion to `master`.
7. Done means the authorized production release and required verification are complete.

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

Participant assignment is responsibility metadata, not a grant of application access. A developer or QA participant may be a person or an AI agent. Board membership and Owner/Editor/Viewer permissions remain separate and authoritative for access.

### 4.2 Review and handoff tracking without extra columns

Keep the existing workflow columns/states. Within In Review, show the current checkpoint and its status, such as:
- Awaiting independent QA / QA in progress / QA passed / QA failed.
- Awaiting senior review / senior review passed / changes requested.
- Awaiting Team Lead authorization to merge into develop.
- Awaiting test deployment / test deployment verified.
- Awaiting Team Lead acceptance / accepted / rejected.

Represent each handoff as an attributable activity entry with a timestamp, checkpoint, outcome, summary, and relevant evidence links. A normal comment should remain available for discussion; structured handoff entries should make important transitions easy to scan and query.

### 4.3 GitHub and deployment traceability

Allow cards to record the relevant GitHub and deployment references. The first release may use explicit user/agent entry or MCP updates; automatic synchronization with GitHub or deployment providers is a later enhancement.

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

AI agents should be able to report their own coordination actions only after the operation is confirmed by the API. The system should distinguish:
- The authenticated account that performed the API operation.
- Optional client/agent attribution supplied for operational context.
- The person or agent assigned responsibility for the work.

Client-supplied agent labels are descriptive metadata, not proof of identity. Do not treat an AI agent name as a trusted identity or allow it to elevate permissions.

## 5. Functional requirements

### A. Work ownership and task definition

1. **FR-1 — Work item type:** A card shall support a configurable or predefined work item type while preserving existing card behavior and data.
2. **FR-2 — Developer assignment:** A card shall identify one primary developer responsible for implementation.
3. **FR-3 — Independent QA assignment:** A card shall identify one QA reviewer. The application shall flag or prevent the same participant from being assigned as both developer and independent QA for the same card.
4. **FR-4 — Participant catalog:** The application shall support selecting human and AI participants as responsibility labels without requiring every AI participant to have a Kanban login or board membership. The design shall avoid conflating assignment with authorization.
5. **FR-5 — Requirement traceability:** A card shall store a link to its authoritative requirements document and allow references to requirement IDs and acceptance-criteria IDs.
6. **FR-6 — Acceptance criteria:** A card shall support a structured, checkable list of task-level acceptance criteria, with completion state and optional evidence/comment.
7. **FR-7 — Dependencies:** A card shall support links to prerequisite/blocking cards and display both its blockers and the cards it blocks. The system shall prevent self-dependencies and detect or reject dependency cycles.
8. **FR-8 — Blocker details:** When a card is Blocked, the card shall make the blocker reason, responsible party, impact, and next action visible. Clearing a blocker shall not silently mark the task complete.

### B. Review, handoff, and acceptance

9. **FR-9 — Review checkpoint:** A card in In Review shall record its current review/deployment checkpoint without requiring additional workflow columns.
10. **FR-10 — QA outcome:** The assigned independent QA reviewer shall be able to record PASS or FAIL against a specific PR/commit, including checks performed, findings, limitations, and evidence links.
11. **FR-11 — Senior review outcome:** The card shall record the senior review outcome and the exact revision reviewed. A PASS shall not itself authorize a merge.
12. **FR-12 — Team Lead authorization:** The card shall distinguish a Team Lead's explicit authorization to merge a specific PR into `develop` from AI review results and general comments.
13. **FR-13 — Test deployment evidence:** The card shall record the test environment, deployed commit SHA, deployment time/result, and verification evidence where applicable.
14. **FR-14 — Human acceptance:** The Team Lead shall be able to record ACCEPTED or REJECTED against the tested revision, with notes and evidence. Only an accepted test result may qualify the card for Ready to Merge.
15. **FR-15 — Production release evidence:** The card shall record the production release reference and verification evidence before it is considered Done, in accordance with the project's release policy.
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
24. **FR-24 — Ready to Merge validation:** A code task shall not qualify for Ready to Merge until independent QA and senior review have passed, the Team Lead's test-integration authorization is recorded, the relevant test deployment is verified, and human acceptance is recorded for that tested revision.
25. **FR-25 — Done validation:** A code task shall not qualify for Done until the separately authorized production promotion and required production verification are recorded. Non-code task types may use an appropriate completion rule.
26. **FR-26 — MCP parity:** Supported MCP operations shall be able to read and update the new structured fields and record handoffs/checkpoints, subject to the same backend authorization and validation rules as the web application.
27. **FR-27 — Confirmed operations:** MCP responses and agent handoffs shall reflect the actual API result. Failed, rejected, or unconfirmed writes must not be reported as successful.
28. **FR-28 — Existing permissions:** All new web and MCP operations shall enforce the existing board roles and server-side authorization. UI-only restrictions are insufficient.
29. **FR-29 — General-purpose operation:** The features shall remain usable for ordinary Kanban projects. AI-specific participants and review checkpoints shall be optional unless a board explicitly enables the development workflow profile.

### E. Usability and administration

30. **FR-30 — Workflow profile:** A board owner shall be able to enable a software-development workflow profile that exposes the relevant fields and validation rules while leaving ordinary boards lightweight.
31. **FR-31 — Card summary:** Card list views shall provide a compact indication of owner, QA reviewer, blockers, current review checkpoint, and key delivery status without making cards visually overwhelming.
32. **FR-32 — Search and filtering:** Users shall be able to filter/search cards by developer, QA reviewer, work item type, dependency/blocker status, review checkpoint, and relevant delivery status.
33. **FR-33 — Responsive experience:** New fields and checkpoint/history views shall remain usable on mobile and desktop.
34. **FR-34 — Import/export:** Board JSON export/import shall preserve new structured fields and relationships, with validation and safe handling of unknown or older fields.

## 6. Non-functional requirements

- **NFR-1 — Backward compatibility:** Existing boards and cards without new metadata shall continue to load and operate. Database migrations and API changes shall be backward-compatible where practical.
- **NFR-2 — Authorization:** Backend authorization remains authoritative for web and MCP requests. No new field or workflow profile may bypass Owner/Editor/Viewer permissions.
- **NFR-3 — Audit integrity:** Important history events shall not be silently overwritten as ordinary editable comments. Corrections should be represented by subsequent events or an explicitly audited correction mechanism.
- **NFR-4 — Attribution:** Store the authenticated actor and timestamp for system-recorded actions. Optional agent/client attribution must be clearly marked as unverified descriptive metadata unless independently authenticated.
- **NFR-5 — Data integrity:** Dependency relationships shall be validated for existence, board boundaries, self-reference, and cycles. Review/acceptance evidence shall reference the relevant revision.
- **NFR-6 — Security:** Validate and safely render all user- and agent-supplied text/URLs. Continue using the existing rich-text sanitization boundary. Do not store access tokens, OAuth secrets, or other credentials in card metadata or activity entries.
- **NFR-7 — Reliability:** Failed writes must leave the user with a clear error and must not display optimistic success as confirmed completion. Preserve retry-safe behavior.
- **NFR-8 — Performance:** Board loading and card rendering must remain responsive for boards with substantially more activity and relationship metadata than today. Avoid loading an unbounded full audit history for every card in the board view.
- **NFR-9 — Maintainability:** Keep frontend, backend, and MCP responsibilities in their existing service boundaries. Prefer explicit API contracts and focused modules over duplicating workflow rules across clients.
- **NFR-10 — Accessibility:** New controls, status indicators, dialogs, and history views shall support keyboard navigation and accessible labels/announcements.
- **NFR-11 — Provider neutrality:** Do not hard-code behavior to Claude, ChatGPT, Gemini, Antigravity, or a specific model provider. Participant names and integrations should be configurable and provider-neutral.

## 7. Scope

### In scope

- A board-level software-development workflow profile.
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
3. **Participant representation:** Design a lightweight participant reference that can represent a board member or a named external/AI contributor. Do not require an AI contributor to authenticate as a separate Kanban user. Do not use this reference for access control.
4. **History model:** Consider a dedicated structured activity/event representation for important workflow events instead of encoding every checkpoint as free-form rich-text comments. Preserve ordinary comments for discussion.
5. **Revision-aware evidence:** QA, senior review, deployment, and acceptance records must identify the exact PR/commit or revision they apply to. If the implementation changes, stale evidence must not silently satisfy gates for the new revision.
6. **Dependency model:** Dependencies should be explicit card relationships within a board, with clear behavior for archived/deleted cards and cycle prevention.
7. **API and MCP:** Extend backend DTOs/endpoints and MCP tools only as needed. Keep authorization and business validation centralized in the backend. MCP tool descriptions should explain required fields, side effects, and confirmation behavior.
8. **Migration:** Existing cards should receive safe defaults (for example, no workflow profile and empty optional metadata). Avoid destructive resets or requiring users to recreate boards/cards.
9. **Incremental delivery:** Deliver a useful manual-tracking MVP first. Add source-system synchronization only after the underlying workflow data model and manual process are proven.
10. **No premature automation:** A manually recorded Team Lead decision is an auditable workflow record, not an automatic GitHub permission or merge action. Any future integration must still respect GitHub branch protections and the explicit authorization model.

## 9. Security and privacy considerations

- Continue to use the existing Webtools authentication and backend authorization model.
- Validate all mutations on the backend, including those initiated through MCP.
- A board role (Owner/Editor/Viewer) controls what a signed-in user may do. Developer/QA assignment does not grant access or additional permissions.
- Record the authenticated user/account that made an action. If an AI client name is also recorded, treat it as client-supplied metadata unless there is a verified identity mechanism.
- Do not expose private repository details, PR links, deployment URLs, or activity history to users who lack access to the board.
- Prevent stored XSS through safe rendering and URL validation; continue sanitizing rich-text content.
- Avoid storing secrets, bearer tokens, environment variables, or sensitive deployment configuration in cards, comments, or handoff records.
- Consider abuse cases where an AI client attempts to mark its own QA as passed, impersonate the Team Lead, fabricate deployment evidence, or move a card past a gate. Server-side rules and audit attribution must prevent an untrusted client assertion from becoming privileged authorization.
- The Team Lead's approval must be attributable to the authenticated human account and explicit action. AI review results are advisory and must never be accepted as human approval.

## 10. Dependencies and sequencing

1. Inspect current backend schema/API, frontend card state/rendering, and MCP tool contracts.
2. Agree on the workflow-profile, participant, dependency, checkpoint, and activity data model before parallel implementation.
3. Implement backend persistence, validation, and authorization before exposing mutations through frontend and MCP.
4. Add frontend card editing, summaries, filters, history, and transition guidance against the agreed API.
5. Add MCP read/write support after the backend contract is stable.
6. Add import/export compatibility and automated regression tests.
7. Deploy to the test environment and use the Team Lead's acceptance process before any production promotion.

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
- [ ] **AC-10:** A Team Lead authorization to merge a specific PR into `develop` is recorded distinctly from reviewer PASS results.
- [ ] **AC-11:** Test deployment evidence identifies the deployed commit, environment, and verification result.
- [ ] **AC-12:** Human acceptance is recorded against the tested revision. Rejected or stale acceptance evidence cannot qualify a newer revision for Ready to Merge.
- [ ] **AC-13:** Production promotion and verification evidence are recorded separately; Kanban does not perform an unauthorized merge or deployment.
- [ ] **AC-14:** Important assignment, status, checkpoint, handoff, approval, and deployment events appear in chronological activity history with authenticated actor and timestamp.
- [ ] **AC-15:** Web and MCP operations enforce the same backend permissions and validation rules.
- [ ] **AC-16:** MCP reports a write as successful only after the backend confirms it; errors and permission denials are surfaced accurately.
- [ ] **AC-17:** Board JSON export/import preserves all new supported metadata and relationships, with backward compatibility for older exports.
- [ ] **AC-18:** Users can filter by developer, QA reviewer, work type, blocker/dependency, and checkpoint.
- [ ] **AC-19:** Automated tests cover backend validation/authorization, frontend behavior, MCP operations, migrations, and relevant regression paths.
- [ ] **AC-20:** The pilot completes at least one end-to-end task through requirement linkage, assignment, implementation, independent QA, senior review, test integration, human acceptance, and separately authorized production release, with evidence recorded on the cards.

## 12. Testing requirements

- **Backend unit/integration tests:** participant assignment validation; distinct developer/QA; workflow profile behavior; transition gates; dependency validation and cycle prevention; revision-aware review/acceptance evidence; activity attribution; board-role authorization.
- **Database migration tests:** upgrade an existing database with populated boards/cards; verify defaults and preservation of existing records.
- **API tests:** create/read/update cards with new metadata; invalid payloads; stale revisions; unauthorized mutations; dependency edge cases; import/export compatibility.
- **Frontend Playwright tests:** enable/disable workflow profile; edit assignments and criteria; display dependencies and blockers; add handoffs/checkpoints; record review/acceptance; filters; responsive interactions; old-board regression.
- **MCP tests:** verify new read/write tools and parameters; permission denial; invalid transitions; failed writes; accurate confirmation; authenticated actor attribution; no privilege escalation from participant labels.
- **Security tests:** stored XSS and unsafe URLs; unauthorized cross-board access; attempts to impersonate Team Lead approval; attempts to self-approve QA; attempts to bypass transition validation.
- **End-to-end pilot:** use a non-production board and test deployment. Preserve evidence for each checkpoint and verify the deployed commit matches the accepted revision.
- **Evidence:** PRs should include test commands/results, relevant CI links, migration notes, and screenshots or traces for meaningful UI flows. QA reports must state exact PR/commit reviewed and any unverified areas.

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
| 3. Backend workflow rules and authorization | Implement server-side validation for assignments, dependencies, state gates, review/acceptance evidence, and role enforcement. | After/alongside task 2 |
| 4. Frontend card workflow experience | Add workflow profile settings, structured card fields, criteria, dependencies, checkpoint display, handoffs, history, and delivery references. | After API contract stabilizes |
| 5. GitHub/deployment reference UX | Add explicit repository/branch/PR/commit and deployment references, distinguishing manual records from verified integration data. | Alongside task 4 |
| 6. MCP workflow support | Extend MCP read/write operations for supported fields, dependencies, checkpoints, and handoffs; verify permissions and confirmed outcomes. | After backend contract |
| 7. Search, filters, and board summaries | Add workflow-aware filters and compact card indicators while preserving existing board usability. | After core UI |
| 8. Import/export and compatibility | Preserve new metadata in JSON transfer and validate older imports/exports. | After data model |
| 9. Automated regression and security tests | Add backend, migration, frontend, MCP, authorization, XSS, and transition-gate coverage. | Throughout; final verification |
| 10. Pilot deployment and workflow validation | Deploy to test, run a real AI-team task through the full lifecycle, record evidence, collect feedback, and resolve pilot gaps. | After implementation and QA |

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

## 16. Open questions and decisions

| Question / decision | Owner | Status | Resolution |
|---|---|---|---|
| Should the software-development workflow profile be enabled per board or offered as a board template? | Team Lead / Architecture | Open | Recommend per-board profile initially; decide during task 1 design. |
| What participant catalog UX is simplest for named AI agents and human contributors who are not Kanban members? | Architecture / UX | Open | Must not grant access or require a login; finalize in data-model design. |
| Should task-level acceptance criteria be a dedicated checklist entity or structured card field? | Architecture | Open | Choose based on API, audit, and import/export needs. |
| Which GitHub facts can be verified automatically with currently available integration permissions? | Team Lead / Coordinator | Open | MVP records links/revisions manually; verify integration feasibility separately. |
| What is the minimum set of checkpoint types required for the first pilot? | Team Lead / Coordinator | Open | Use the stages in section 4.2 as the proposed baseline; simplify only with approval. |
| Should activity history support edits/deletions for non-critical events? | Architecture / Security | Open | Important workflow events must be append-oriented and auditable. |

## 17. Approval

- **Team Lead decision:** Pending — Approved / Changes requested / Rejected
- **Approved by:** <human approver>
- **Date:** <YYYY-MM-DD>
- **Approved revision/commit:** <commit SHA or link>
- **Notes:** This document is a proposed requirements baseline. Claude.ai may review it and prepare Backlog cards, but implementation cards must not enter Ready until the Team Lead approves the requirements and task prerequisites are satisfied.
