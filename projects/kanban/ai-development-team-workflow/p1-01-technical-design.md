# P1-01 — Kanban AI Development Workflow Architecture and Technical Design

**Status:** Draft for independent technical QA and coordinator review  
**Work item:** P1-01 Task 1 — Architecture, data model, API contract and file-ownership map  
**Design owner:** ChatGPT — Architect  
**Independent technical QA:** Dev1 — Claude Code  
**Coordinator review:** Claude.ai — traceability, sequencing, ownership and handoff readiness  
**Scope:** Design only. No implementation code, schema change, deployment, merge, or Kanban mutation is included in this PR.

## 1. Executive decisions

This design retains Kanban as a general-purpose board application and adds a per-board, opt-in software-development workflow profile. Existing columns and ordinary board behavior remain unchanged. Workflow data and authoritative validation belong in the backend. The browser frontend presents the workflow. MCP remains a stateless adapter over the same backend contract.

The main design decisions are:

1. **Acceptance criteria:** use a normalized, individually addressable criterion record per card criterion, not one unstructured JSON field. This supports stable source references, per-item completion, ordering, evidence, audit history and import/export remapping.
2. **Participants:** use a board-scoped participant catalog. Authenticated contributors link to existing board members and their stable Kanban user IDs. External/non-login participants may be listed but cannot authenticate, act, approve or receive permissions through catalog membership.
3. **Checkpoints:** use an extensible, controlled taxonomy for independent QA, senior review, merge authorization, test deployment, test acceptance, production authorization and production verification. These remain card metadata and activity, not new mandatory columns.
4. **Team Lead:** do not equate every Board Owner with the Team Lead. Multiple Owners are permitted. Store one explicit Team Lead user ID in the board workflow profile, and require that user to remain an OWNER.
5. **Protected decisions:** merge-to-develop authorization, test ACCEPTED/REJECTED and production-promotion authorization require both the authenticated mapped Team Lead and a cryptographically verified OAuth client ID equal to kanban-web. Unknown and MCP client IDs are denied by default. These operations have no MCP mutation tools.
6. **Senior review:** only the designated senior-reviewer account or the mapped Team Lead may record a senior-review outcome. Developers and QA reviewers cannot do so. A Team Lead relay of ChatGPT's review records the Team Lead as the authenticated actor and ChatGPT as the attributed participant; it must not claim ChatGPT was the authenticated actor.
7. **Import/export:** extend the existing board JSON format with versioned workflow metadata. The current import path recreates cards and columns, so the implementation must remap source IDs to newly generated IDs before restoring relationships. Legacy exports remain importable.
8. **Migrations:** reserve backend Flyway V3–V6 for P1-02 through P1-05. V1 and V2 are immutable. All new migrations are additive and preserve existing data.
9. **Security gap:** the inspected Auth Server does not issue a signed access-token client_id claim. A separately approved Auth Server change is required before the backend can enforce the requested positive allowlist. The documented MCP direct-backend path also does not match the backend's current gateway-header actor resolver; this trust path must be investigated before protected decisions are implemented.
10. **Branches:** all four service repos default to master and contain develop. The target policy is feature branches from develop, PRs into develop for test integration, and a separately authorized promotion from develop to master for production. Current CI does not yet consistently validate develop; details are in §3.

This is a design proposal for Dev1 QA and Claude.ai coordination review. The security gaps are explicit prerequisites, not permission to silently expand P1-05.

## 2. Baseline and repository inspection

### 2.1 Requirements baseline

The design uses the supplied approved requirements reference:

- Path: projects/kanban/ai-development-team-workflow/requirements.md
- Revision supplied in the task: c850fc38e4684fa2d177bae60ab68197bbdfc70b
- Retrieved file blob SHA: 8031e0b71b6653cba36e37f4cdec0017cf5c6cf4

There is a metadata inconsistency in the supplied document itself: its header says “In Review — Team Lead approval pending”, and §17 still says “Team Lead decision: Pending”. The task explicitly identifies the supplied revision as approved, so this design treats it as the requested baseline. The design does not alter that document or invent an approval event. The Team Lead/coordinator should reconcile the approval metadata or link the recorded approval before implementation cards rely on the document as an approved baseline.

### 2.2 Source access confirmation

I confirmed source access, not merely README access. The following source material was readable and inspected.

| Repository | Default branch | Default-branch SHA inspected | Source inspected |
|---|---|---|---|
| ikoyski/kanban | master | e9ea2a7b5f311e7a40cdb2a91c26cd5d9fbe1da4 | API client, import/export orchestration, card actions, card UI/modal and Playwright workflow |
| ikoyski/webtools-kanban-backend | master | 383b819c30149c8fd3ee5d10709e6d5e7483d45d | V1/V2 Flyway SQL, entities, request/response DTOs, controllers, services, role checks, actor resolver and Maven workflow |
| ikoyski/webtools-kanban-mcp | master | e88c4aa0d3187bf83ec4f29d6d50e56c5678d7f9 | JWT decoder/filter, request token context, backend API client, modern tool repository, legacy tool catalog/config and Maven workflow |
| ikoyski/webtools-auth | master | 6b9e294945b61a7379838e0d48f9f3f873e9f755 | Authorization server config, JWT customizer, JWK config, registered-client seeder, SecurityConfig, database migrations and Maven workflow |

Repository links at the inspected revisions:

- https://github.com/ikoyski/kanban/tree/e9ea2a7b5f311e7a40cdb2a91c26cd5d9fbe1da4
- https://github.com/ikoyski/webtools-kanban-backend/tree/383b819c30149c8fd3ee5d10709e6d5e7483d45d
- https://github.com/ikoyski/webtools-kanban-mcp/tree/e88c4aa0d3187bf83ec4f29d6d50e56c5678d7f9
- https://github.com/ikoyski/webtools-auth/tree/6b9e294945b61a7379838e0d48f9f3f873e9f755

All four source repositories and the required source files were readable. The MCP repository has no Flyway migration directory; persistence migrations are in the backend repository. No blocker is raised for source access.

### 2.3 Current architecture confirmed from code

| Component | Confirmed ownership and source locations |
|---|---|
| Frontend | src/js/api.js is the shared REST client. src/js/app/card-actions.js and src/js/ui/card-detail.js coordinate and render card detail. src/components/modals/card-detail-modal.html is the card modal. src/js/app/data-transfer.js handles browser import/export orchestration. |
| Backend | REST controllers are under src/main/java/com/ikoyki/webtools/kanban/backend/controller. Entities, repositories, DTOs and services own persistence and business rules. BoardAccessService enforces board membership and OWNER/EDITOR/VIEWER role checks. |
| MCP | Stateless adapter. ModernKanbanToolRepository declares modern tools; LegacyMcpConfig declares legacy tools; KanbanApiClient owns backend API requests, validation and error mapping. MCP forwards the caller's bearer token instead of using a shared service account. |
| Auth Server | AuthorizationServerConfig configures token generation and claims; JwkConfig configures RSA signing and JWKS; ClientSeeder creates registered OAuth clients. |

The backend and Auth Server are Spring Boot 3.4.2 / Java 21 services. The backend uses Flyway and Springdoc OpenAPI. The MCP server is a separate Spring Boot 4 / Java 21 service. The repository contains no standalone OpenAPI contract file; the proposed contract in §8 is the design source for API documentation and DTO/controller implementation.

## 3. Branch model and current branch state

### 3.1 Per-repository branch comparison

The default branch was verified from repository metadata. Branch presence, commit SHAs and ancestry were checked using GitHub branch and compare endpoints with base=develop and head=master.

| Repository | Default branch | develop exists | master SHA | develop SHA | Relationship |
|---|---|---|---|---|---|
| kanban frontend | master | Yes | e9ea2a7b5f311e7a40cdb2a91c26cd5d9fbe1da4 | e9ea2a7b5f311e7a40cdb2a91c26cd5d9fbe1da4 | Identical; 0 ahead / 0 behind |
| Kanban backend | master | Yes | 383b819c30149c8fd3ee5d10709e6d5e7483d45d | 383b819c30149c8fd3ee5d10709e6d5e7483d45d | Identical; 0 ahead / 0 behind |
| Kanban MCP | master | Yes | e88c4aa0d3187bf83ec4f29d6d50e56c5678d7f9 | e88c4aa0d3187bf83ec4f29d6d50e56c5678d7f9 | Identical; 0 ahead / 0 behind |
| Auth Server | master | Yes | 6b9e294945b61a7379838e0d48f9f3f873e9f755 | 1835b2ece046306fbddecdee11e44a9df10b186b | master is ahead by 1 commit; develop is behind by 1 |

For Auth Server, develop at 1835b2e is the merge base and master at 6b9e294 is the one-commit-ahead merge commit titled “Merge develop: public-client refresh token fix”. Thus the feature work already on develop is included in master; master has the merge commit on top. This is not a divergent-history conflict.

The three Kanban repositories currently have identical master and develop pointers. Their integration branches exist but have not yet received a separate commit. This is an initial synchronized state, not evidence that develop is absent.

### 3.2 Target branch and environment policy

Use the same policy in all four repositories:

- master is the production/release branch and remains the GitHub default branch. Production releases are promoted here only after the Team Lead's separate production authorization.
- develop is the integration and test branch. Feature PRs target develop; the test environment should run the integrated develop revision.
- feature/<task-name> branches are created from the latest develop. Developers do not push directly to develop or master.
- No feature PR targets master directly.
- A feature PR is merged to develop only after CI, independent QA, senior review and explicit Team Lead authorization for the exact PR head SHA.
- Test acceptance is recorded against the deployed develop commit.
- Production promotion is a separate reviewed PR or controlled release from develop to master, using only accepted changes. The Team Lead authorizes the exact release commit SHA separately.
- Developers do not merge their own PRs. GitHub branch protection and required checks remain hard controls. Kanban records the human decision; it does not replace GitHub controls.

If develop contains unaccepted work, do not blindly merge the full develop tip to master. Use a reviewed release branch or another controlled promotion strategy that contains only accepted work, and record the exact release SHA and included cards.

### 3.3 Current CI configuration is not yet aligned

Source inspection found the following:

| Repository | Current GitHub Actions triggers | Production image publishing |
|---|---|---|
| kanban frontend | Playwright workflow push/PR targets master only | No image publish in this workflow |
| Kanban backend | Maven workflow push/PR targets master only | Docker image ikoyski/webtools-kanban-backend:latest published on push to master only |
| Kanban MCP | Maven workflow push/PR targets master only | Docker image ikoyski/webtools-kanban-mcp:latest published on push to master only |
| Auth Server | Maven workflow push/PR targets develop and master | Docker image ikoyski/webtools-auth:latest published on push to master only |

Therefore, current PRs targeting develop do not trigger frontend/backend/MCP CI workflows. Auth Server already runs its Maven/Sonar CI against both develop and master, and its Docker publish remains master-only. The target model should be applied consistently: CI verification on push and PR to both develop and master; production image publishing only on master. Test deployments must use a distinct test deployment/service or test tag and must not overwrite the production latest image.

The frontend README documents a Render test URL and test API/Auth environment, but no checked-in render.yaml was found in the inspected source tree. The exact Render service branch settings were not verified in this task because the Render workspace was not selected for read access. Thus master-as-production and develop-as-test are the approved target branch convention and are supported by the Docker publish rules; the live Render branch-to-service mapping remains an operational verification item. Do not claim that current Render services are already configured to deploy those exact branches.

**Proposed card update:** add an implementation task to align CI triggers with develop while keeping production artifact publishing restricted to master. At minimum, update the frontend Playwright workflow and backend/MCP Maven workflows. Auth's workflow already has the correct branch triggers, though its behavior should be retained and tested. Add verification of the live Render test/prod branch mapping as an explicit operational prerequisite to the first test deployment.

## 4. Current-state findings that constrain design

### 4.1 Board Owner is not uniquely the Team Lead

The backend BoardRole enum has VIEWER, EDITOR and OWNER. BoardAccessService.requireAtLeast compares the membership role. BoardMemberService permits an OWNER to add members and update other members' roles; more than one OWNER can exist. Therefore OWNER is a board-administration role, not a unique Team Lead identity.

**Decision:** store one team_lead_user_id in the board workflow profile. That user must have current OWNER membership on the same board. Protected decisions require the authenticated actor to equal this exact mapped user and the user's current role to remain OWNER. A different OWNER is not a Team Lead solely by being an OWNER. Transfer of the Team Lead mapping is a separate audited OWNER action. The mapped Team Lead cannot be removed or demoted until the mapping is transferred.

### 4.2 Backend actor resolution currently assumes trusted gateway headers

AuthUserArgumentResolver reads X-User-Id, X-User-Email, X-User-Display-Name and X-User-Avatar-Url. Its source comments say the API Gateway replaces these values after JWT validation. UserProvisioningService similarly assumes the central UUID comes from a trusted gateway. The inspected backend POM does not include the OAuth2 resource-server starter, and its source tree contains no backend JWT resource-server security configuration.

The MCP README says MCP-to-backend traffic bypasses the public API Gateway and calls the backend over the private service route. McpConfig forwards the original Authorization header, but its interceptor does not itself add X-User-* identity headers. This is a source-level trust-path mismatch. It does not prove the deployed route is insecure, because the runtime gateway and Config Server configuration were not inspected, but source alone does not prove the backend obtains identity context correctly for direct MCP traffic.

**Required resolution before P1-05:** inspect live gateway and Config Server routing and prove one of these secure arrangements:
- the backend validates the bearer JWT itself and extracts actor/client claims from the verified token; or
- a trusted gateway validates the token, strips all caller-supplied identity/client headers, injects verified context, and is the only reachable route to the backend.

Private network reachability alone is not authorization. The backend must not trust externally supplied X-User-Id, X-User-Email or client-ID headers. Direct service access must not bypass the trusted authentication boundary. This investigation may need a separately scoped card; do not silently fold unapproved code changes into P1-05.

### 4.3 Auth Server does not issue a signed OAuth client_id claim

The inspected Auth Server token customizer sets the stable central identity UUID in uid, email and optional profile claims, and resource URI(s) in aud. The Spring Authorization Server 1.4.1 JwtGenerator supplies standard issuer, subject, timestamps and token identifiers. The customizer does not add an access-token client_id claim.

The customizer uses the registered OAuth client when constructing the audience, but then replaces the default client-id audience with resource URI audience(s). The aud claim is therefore the resource audience, not a safe substitute for OAuth client identity.

The current registered client seeder defines kanban-web, kanban-claude and kanban-antigravity; client configuration does not itself provide the backend with a signed client identifier in the access token.

Source references:
- Auth Server AuthorizationServerConfig: https://github.com/ikoyski/webtools-auth/blob/6b9e294945b61a7379838e0d48f9f3f873e9f755/src/main/java/com/ikoyski/webtools/authserver/config/AuthorizationServerConfig.java
- Auth Server ClientSeeder: https://github.com/ikoyski/webtools-auth/blob/6b9e294945b61a7379838e0d48f9f3f873e9f755/src/main/java/com/ikoyski/webtools/authserver/config/ClientSeeder.java
- Auth Server JWK setup: https://github.com/ikoyski/webtools-auth/blob/6b9e294945b61a7379838e0d48f9f3f873e9f755/src/main/java/com/ikoyski/webtools/authserver/config/JwkConfig.java
- Spring Authorization Server 1.4.1 JwtGenerator: https://github.com/spring-attic/spring-authorization-server/blob/1.4.1/oauth2-authorization-server/src/main/java/org/springframework/security/oauth2/server/authorization/token/JwtGenerator.java
- Spring Boot 3.4.2 dependency management: https://github.com/spring-projects/spring-boot/blob/v3.4.2/spring-boot-project/spring-boot-dependencies/build.gradle

**Result:** the current signed claim set cannot satisfy AC-24. Do not infer client ID from aud, request parameters, agent labels, MCP headers or caller-supplied attribution.

**Separate scoped change required:** add a signed access-token claim such as client_id, populated only from the server-side registered client in JwtEncodingContext (for example, context.getRegisteredClient().getClientId()). Preserve uid, issuer, scopes and resource audiences. Test authorization-code issuance and refresh-token issuance for kanban-web, kanban-claude, kanban-antigravity and an unrecognized registered client. The backend allowlist must still accept only kanban-web; all missing/unknown claims deny. This Auth Server modification is out of P1-01 and P1-05 scope until separately approved.

## 5. Resolved data and workflow decisions

### 5.1 Board workflow profile

A profile is optional and board-scoped. No profile or enabled=false means ordinary Kanban behavior; workflow-specific validation is inactive. Enabling/disabling the profile is OWNER-only.

The profile contains:
- enabled flag and schema/profile version
- one team_lead_user_id
- designated senior_reviewer_participant_id
- mappings from existing board column IDs to canonical workflow states
- created/updated timestamps and actor references

Do not add mandatory columns. Preserve the existing Backlog, Ready, In Progress, In Review, Revision, Blocked and Done columns. Map column IDs to canonical workflow states rather than relying on mutable titles. “Ready to Merge” is logical eligibility within the review lifecycle, not a new mandatory column.

### 5.2 Participant catalog

Use a board-scoped participant catalog with:
- stable participant ID and board ID
- participant key (for example team-lead, claude-coordinator, dev1, dev2, chatgpt-architect)
- type: HUMAN, AI_AGENT or EXTERNAL
- nullable linked Kanban user ID
- display name, tool family, active status and timestamps

Rules:
- Authenticated HUMAN and AI_AGENT participants link to an existing board member and stable Kanban user ID.
- Each active DevN has its own participant record and distinct authenticated Kanban identity. OAuth client ID identifies the client application type, not the DevN person/runtime.
- EXTERNAL participants may have no user ID. They are descriptive assignees only and cannot act, authenticate or satisfy authenticated QA/approval gates.
- Catalog membership never grants board access. Board membership and OWNER/EDITOR/VIEWER role remain authoritative.
- ChatGPT is the designated senior-review participant. Under the current relay arrangement the authenticated actor is the Team Lead, ChatGPT is attributed participant, and the origin is TEAM_LEAD_RELAY.
- Claude.ai is the coordinator; the catalog entry does not imply repository access.
- Participant name, agent label or request-body user ID is never proof of identity.

### 5.3 Acceptance criteria: normalized records

Use a dedicated card_acceptance_criterion table/entity instead of a JSON-only list.

Fields:
- criterion ID, card ID and optional source AC ID
- text, display order, required flag
- checked state and checked-by actor/timestamp
- optional evidence note and links

A criterion checkbox represents task progress only. It is not the Team Lead test acceptance decision. Test ACCEPTED/REJECTED is a separate protected decision bound to a deployed revision.

This entity design is preferred because individual criteria need stable IDs, independent editing, traceable source references, ordering, evidence, activity events, and import/export ID remapping.

### 5.4 Checkpoint taxonomy

Use a controlled, extensible checkpoint type vocabulary for Phase 1:

| Type | Purpose | Who can record |
|---|---|---|
| INDEPENDENT_QA | QA PASS/FAIL, checks, findings, limitations, evidence and exact PR head SHA | Assigned QA participant only |
| SENIOR_REVIEW | Senior review result, findings and exact PR head SHA | Designated senior reviewer or Team Lead only |
| MERGE_AUTHORIZATION | Permission to merge a specific PR head SHA into develop | Mapped Team Lead through kanban-web only |
| TEST_DEPLOYMENT | Test environment, deployed SHA, result/time and evidence | Authorized operator/editor; provenance recorded |
| TEST_ACCEPTANCE | Human ACCEPTED/REJECTED against exact tested revision | Mapped Team Lead through kanban-web only |
| PRODUCTION_AUTHORIZATION | Permission to promote exact release candidate SHA | Mapped Team Lead through kanban-web only |
| PRODUCTION_VERIFICATION | Production release and verification evidence | Authorized operator; provenance recorded |

Use outcome values appropriate to the type, e.g. IN_PROGRESS, PASS, FAIL, CHANGES_REQUESTED, AUTHORIZED, ACCEPTED, REJECTED, VERIFIED and STALE. Store every attempt as a new record. Never overwrite prior evidence. A changed target SHA makes the old evidence/authorization stale by comparison with the current target.

### 5.5 Senior-review recording rights and ChatGPT relay

The backend rule for SENIOR_REVIEW is:
- allow the authenticated designated senior reviewer user; or
- allow the authenticated mapped Team Lead.

A developer or QA reviewer cannot record a senior review even if they are an EDITOR or another OWNER. The backend must compare the authenticated user against the profile and role; a self-asserted reviewer name is not enough.

For ChatGPT's current relay arrangement, the Team Lead records the result in the Kanban web app with this prefix:

[ChatGPT | Architect/Reviewer | recorded by Team Lead]

Persist the real actor as the authenticated Team Lead; persist ChatGPT as attributed participant; set origin to TEAM_LEAD_RELAY; bind the review to the exact PR head SHA. Never write a fabricated authenticated identity for ChatGPT. If ChatGPT later receives a separate authenticated workflow channel, it must use its own verified participant/user identity.

### 5.6 Protected Team Lead decisions

Three separate protected decisions exist:
1. merge authorization to develop
2. test acceptance or rejection
3. production-promotion authorization

Each write is accepted only when:
1. JWT signature, issuer, expiry and expected resource audience have been validated by the trusted backend security boundary;
2. actor identity is extracted from verified token/gateway context, never the request body or untrusted headers;
3. actor user ID equals the workflow profile's team_lead_user_id;
4. the mapped Team Lead still has OWNER membership on that board;
5. the OAuth client ID comes from a cryptographically verified access-token claim and equals exactly kanban-web;
6. the target PR/test/release revision is present and the decision is bound to the exact SHA;
7. the web UI displays the exact SHA and requires the manual comparison confirmation described by AC-23.

All MCP clients and unknown/future OAuth client IDs are denied by default. This is a positive allowlist, not a denylist. No user/body/header parameter can override actor, client ID, role, origin or decision authority. MCP may read decisions, but must never create, edit or reverse them. A correction or reversal is a new append-only record that supersedes the earlier decision.

The current Auth Server claim gap and gateway trust mismatch mean this rule is a design target, not an already implemented or verified live control.

### 5.7 Revision-aware evidence

Bind:
- QA and senior review to exact PR head SHA
- merge authorization to exact PR head SHA
- test deployment and acceptance to exact deployed/tested commit SHA
- production authorization and verification to exact release candidate/release SHA

A changed SHA makes relevant evidence stale by default. Only the requirements' narrowly defined mechanical, conflict-free rebase delta-check may refresh QA or senior review without a full repeat. It must identify old/new SHAs, demonstrate unchanged effective patch, show required CI reruns and issue fresh evidence for the new SHA. QA delta evidence never substitutes for senior review. A changed merge or production target SHA always requires a new Team Lead authorization.

## 6. Proposed persistence model and Flyway reservation

The following are proposed table names and responsibilities, not existing implementation. Backend V1 and V2 are immutable.

| Reserved version | Card | Tables / persistence | Scope |
|---|---|---|---|
| V3__kanban_workflow_profile_participants_card_metadata.sql | P1-02 | board_workflow_profile, board_workflow_state_mapping, board_participant, card_work_item, card_acceptance_criterion | Per-board profile; Team Lead and senior reviewer mapping; participant catalog; card type, developer/QA, requirement refs and criteria |
| V4__kanban_dependencies_activity_delivery_references.sql | P1-03 | card_dependency, card_activity_event, card_delivery_reference | Same-board dependency edges; append-oriented history; GitHub/deployment refs with manual/verified provenance |
| V5__kanban_revision_bound_checkpoints.sql | P1-04 | card_checkpoint_record | QA/senior/deployment checkpoint evidence with outcome, exact revision, actor, summary, findings, limitations and links |
| V6__kanban_protected_team_lead_decisions.sql | P1-05 | card_protected_decision | Append-only merge, test acceptance/rejection and production authorization records bound to exact SHA and verified actor/client context |

### 6.1 Data constraints and behaviors

- board_workflow_profile: one profile per board. It references the designated Team Lead and senior reviewer.
- board_workflow_state_mapping: maps existing column IDs to canonical state keys. Column must belong to same board.
- board_participant: unique participant key per board. Authenticated participant references a board member; external participant has no user ID.
- card_work_item: one-to-one card extension for work type, developer, QA, requirement URL and requirement/AC references.
- card_acceptance_criterion: one row per criterion; preserve order/source AC/evidence.
- card_dependency: directed prerequisite relationship; same board, no self edge, no cycles. Serialize dependency writes per board or otherwise prevent concurrent writes from introducing a cycle.
- card_activity_event: append-oriented actor/time/event record with optional verified OAuth client, optional participant attribution, revision SHA and structured event payload.
- card_delivery_reference: repository/branch/PR/commit/test deployment/production release; environment, SHA, provenance and verification state. User/MCP submissions are MANUAL; only future trusted integration may mark VERIFIED.
- card_checkpoint_record: append-only review/deployment checkpoint attempts.
- card_protected_decision: append-only protected human decisions; corrections/reversal supersede but do not delete old decisions.

Use FKs and indexes for board/card/participant/criterion/decision retrieval. Where the existing model lacks a composite board/card key, enforce same-board relationships transactionally and add DB constraints where practical. Do not truncate or recreate existing board/card data.

### 6.2 Backward compatibility and rollback

- No profile row or enabled=false preserves ordinary Kanban behavior.
- Existing cards remain valid with no workflow metadata or new child records.
- Keep existing API endpoints and response fields compatible with the current frontend and MCP.
- Make new fields optional for legacy boards.
- Upgrade tests must start from a populated V2 database and preserve boards, columns, cards, memberships, comments and archive state.
- Disable the workflow profile or roll back the affected service through the established deployment process if the new workflow gates regress; never delete data to recover.
- Do not edit V1/V2. Do not allocate these reserved version numbers to another change.

## 7. Workflow gate rules

Backend validation is authoritative; frontend validation is usability support only.

| Operation | Required condition when profile enabled |
|---|---|
| Enable profile | Board OWNER; column mappings valid; designated Team Lead is an OWNER; senior reviewer participant is valid |
| Backlog → Ready | Clear scope and task criteria; dependencies resolved; active developer and different authenticated QA participant assigned; non-code exception explicit |
| Ready → In Progress | Assigned developer accepts/starts the task; authenticated actor recorded |
| In Progress → In Review | Reviewable PR/reference exists (normally PR targeting develop); developer handoff and evidence recorded |
| QA result | Authenticated actor is assigned QA; exact PR head SHA; checks/findings/limitations/evidence |
| Senior review | Authenticated actor is designated senior reviewer or mapped Team Lead; actor must not be developer/QA; exact PR head SHA |
| Merge authorization | Mapped Team Lead + verified client_id=kanban-web; exact PR head SHA; UI manual comparison confirmation |
| Test deployment | Environment, deployed SHA, time/result, evidence and provenance |
| Test acceptance | Mapped Team Lead + verified client_id=kanban-web; exact tested SHA; ACCEPTED or REJECTED |
| Ready to Merge eligibility | QA and senior review passed for current PR head SHA; exact merge authorization; test deploy verified; Team Lead accepted the deployed revision |
| Production authorization | Mapped Team Lead + verified client_id=kanban-web; exact release candidate SHA; UI manual SHA confirmation |
| Done | Separate production authorization and required production verification recorded; a non-code task can use its documented completion rule |

Do not create a Ready to Merge column. This is a logical eligibility state within the existing card workflow.

## 8. Proposed REST API contract

All paths are relative to the existing backend /v1 base URL. Use typed DTOs, validation, board membership checks and service-level authorization. Actor and OAuth client context come only from verified server-side authentication context, never from a request body.

| Method and path | Request / result | Authorization |
|---|---|---|
| GET /boards/{boardId}/workflow-profile | Read enabled state, column mapping, Team Lead and senior reviewer | Board member |
| PUT /boards/{boardId}/workflow-profile | Set enabled state, mapping and designated users/participants | OWNER; Team Lead must be existing OWNER |
| GET /boards/{boardId}/participants | List board participants | Board member |
| POST /boards/{boardId}/participants | Add existing board member as participant or create descriptive EXTERNAL participant | OWNER; does not add membership |
| PATCH /boards/{boardId}/participants/{participantId} | Update display/tool metadata or active status | OWNER |
| GET /cards/{cardId}/workflow | Read work type, developer/QA and requirement refs | Board member |
| PUT /cards/{cardId}/workflow | Set work type, developer, QA, requirement URL and FR/AC references | EDITOR; enforce distinct developer and QA |
| GET /cards/{cardId}/acceptance-criteria | List criterion IDs, text, source refs, order, checked state and evidence | Board member |
| POST /cards/{cardId}/acceptance-criteria | Add {sourceAcId?, text, order, required} | EDITOR |
| PATCH /cards/{cardId}/acceptance-criteria/{criterionId} | Edit text/order or check/evidence state | EDITOR; audit actor |
| GET /cards/{cardId}/dependencies | List prerequisite cards and cards blocked by this card | Board member |
| POST /cards/{cardId}/dependencies | Add {dependsOnCardId}; reject cross-board, self and cyclic edges | EDITOR |
| DELETE /cards/{cardId}/dependencies/{dependsOnCardId} | Remove relationship and append activity event | EDITOR |
| GET /cards/{cardId}/checkpoints | Read checkpoint attempts and stale state | Board member |
| POST /cards/{cardId}/checkpoints/qa | {prHeadSha, outcome, summary, checks[], findings[], limitations[], evidenceUrls[]} | Assigned QA only |
| POST /cards/{cardId}/checkpoints/senior-review | {prHeadSha, outcome, summary, findings[], evidenceUrls[]} | Designated senior reviewer or Team Lead only |
| POST /cards/{cardId}/checkpoints/test-deployment | {environment, deployedSha, url?, result, verifiedAt?, evidenceUrls[]} | Authorized operator/editor; provenance explicit |
| POST /cards/{cardId}/checkpoints/production-verification | {environment, releaseSha, releaseUrl?, result, evidenceUrls[]} | Authorized operator/editor; provenance explicit |
| POST /cards/{cardId}/handoffs | {recipientParticipantId, checkpointType, summary, nextAction, outcome, evidenceUrls[]} | EDITOR; sender derived from auth context |
| GET /cards/{cardId}/activity | Paginated chronological workflow history | Board member |
| GET /cards/{cardId}/delivery-references | Read manual/verified GitHub and deployment refs | Board member |
| POST /cards/{cardId}/delivery-references | {referenceType, url, revisionSha?, environment?}; server sets MANUAL provenance | EDITOR |
| GET /cards/{cardId}/protected-decisions | Read merge, test acceptance and production decisions | Board member |
| POST /cards/{cardId}/decisions/merge-authorization | {prUrl, prHeadSha, manualShaComparisonConfirmed, reason?} | Mapped Team Lead + verified kanban-web only |
| POST /cards/{cardId}/decisions/test-acceptance | {testedSha, result: ACCEPTED or REJECTED, notes?, evidenceUrls[]} | Mapped Team Lead + verified kanban-web only |
| POST /cards/{cardId}/decisions/production-authorization | {releaseSha, releaseUrl?, manualShaComparisonConfirmed, reason?} | Mapped Team Lead + verified kanban-web only |

Protected decisions have no update/delete endpoint. A reversal or correction is a new append-only decision referencing the prior record. Reject client-supplied actor ID, client ID, role, origin or approval attribution when presented as authoritative values.

The merge and production authorization UI must show the exact target SHA prominently and require the Team Lead to confirm the manual comparison. Label this as manual comparison, not GitHub verification.

## 9. Proposed MCP tool signatures and boundary

MCP remains stateless. ModernKanbanToolRepository and LegacyMcpConfig must expose the same supported tools; KanbanApiClient owns outbound requests, input validation and error mapping. Backend owns authorization, transitions, persistence and business rules.

Existing tools remain. Add these tools:

| MCP tool signature | Purpose |
|---|---|
| get_workflow_profile(boardId) | Read profile and state mapping |
| list_participants(boardId) | Read participant catalog |
| get_card_workflow(cardId) | Read structured work fields and requirement links |
| update_card_workflow(cardId, workItemType?, developerParticipantId?, qaParticipantId?, requirementDocumentUrl?, requirementRefs?, acceptanceCriteriaRefs?) | Update permitted metadata |
| list_acceptance_criteria(cardId) | Read criteria and completion |
| add_acceptance_criterion(cardId, text, sourceAcId?, order?, required?) | Add one criterion |
| update_acceptance_criterion(cardId, criterionId, text?, order?, checked?, evidenceNote?) | Update criterion or check state; not human acceptance |
| list_card_dependencies(cardId) | Read blockers and blocked cards |
| add_card_dependency(cardId, dependsOnCardId) | Add same-board validated dependency |
| remove_card_dependency(cardId, dependsOnCardId) | Remove edge and record activity |
| list_card_checkpoints(cardId) | Read checkpoint history/stale state |
| record_qa_result(cardId, prHeadSha, outcome, summary, checks[], findings[], limitations[], evidenceUrls[]) | Record only when actor is assigned QA |
| record_senior_review(cardId, prHeadSha, outcome, summary, findings[], evidenceUrls[]) | Backend allows designated senior reviewer or Team Lead; Dev1/Dev2 as developers/QA are rejected |
| record_handoff(cardId, recipientParticipantId, checkpointType, summary, nextAction, outcome, evidenceUrls[]) | Append structured handoff |
| list_card_activity(cardId) | Read chronological activity |
| add_delivery_reference(cardId, referenceType, url, revisionSha?, environment?) | Add a MANUAL reference; MCP cannot mark VERIFIED |
| list_protected_decisions(cardId) | Read Team Lead decisions |

There are intentionally no MCP mutation tools for merge authorization, test acceptance/rejection, or production authorization. The backend must also reject crafted REST mutations from MCP and unknown clients; omission from the tool catalog is defense in depth, not the authorization boundary. MCP may read protected decisions.

Tool writes are reported as successful only after the backend confirms success. Errors and permission denials must be returned as errors, not optimistic success. Never pass client-supplied agent or actor names as authoritative identity.

## 10. Import/export design and ownership

### 10.1 Existing location confirmed

Import/export is implemented across frontend and backend:
- Frontend orchestration: src/js/app/data-transfer.js
- Frontend API wrapper: src/js/api.js (exportBoard and importBoard)
- Backend endpoints: BoardController.exportBoard and BoardController.importBoard
- Backend import persistence: BoardService.importBoard
- Backend export DTO mapping: BoardMapper
- Backend input DTO: ImportBoardRequest

The current backend import deletes existing cards and columns for the target board and recreates them from the supplied maps. Newly generated IDs must therefore not be assumed to match the IDs in the import file.

### 10.2 Proposed versioned transfer behavior

Export:
- Add top-level schemaVersion=2.
- Preserve the existing name/columns/cards structure.
- Add a workflow extension with profile, participant references, card work fields, criteria, dependency edges, checkpoint/activity evidence and delivery references.
- Never export/import board membership, role grants, secrets, OAuth tokens or permission-bearing identity data. The exported role is caller context, not a portable permission.
- Export authenticated participant identity as a reference; do not interpret it as a request to create membership in the target board.

Import:
1. Missing schemaVersion means legacy format; import it with workflow disabled and no workflow metadata.
2. Validate the entire document and all references before deleting/recreating existing board content.
3. Build source column ID → new column ID and source card ID → new card ID maps as the backend recreates columns/cards.
4. Restore metadata, criteria and dependencies only after the new cards exist; rewrite every relationship using the maps.
5. Resolve authenticated participants only to existing target-board members. Never auto-add users or grant board roles. An unresolved authenticated participant must cause a clear validation error or deliberate Team Lead remapping.
6. Preserve external participant keys as descriptive entries only.
7. Revalidate dependencies after remapping; reject cross-board, self and cyclic edges.
8. Preserve MANUAL vs VERIFIED provenance. Import cannot fabricate a VERIFIED claim or create a current protected Team Lead authorization.
9. Old exports remain importable. Unknown future schema versions must fail clearly or follow an explicit forward-compatibility rule; do not silently discard recognized workflow metadata.
10. Replace/rebuild the board transactionally so validation/persistence failure does not leave a partially replaced board.

P1-10 owns the implementation and regression tests after P1-03 is integrated.

## 11. File-ownership map: one writer per hot file

The following is the proposed single-writer allocation, aligned with the existing card split and repository access matrix. These are task-level write assignments, not claims that files have already changed.

### 11.1 Frontend — Dev2 owns Phase 1 frontend changes

| File or file group | Responsibility | Single writer / task |
|---|---|---|
| src/js/api.js | Shared REST client | Dev2; P1-06 then P1-07 then P1-08, sequential |
| src/components/modals/card-detail-modal.html | Card detail modal structure | Dev2; P1-06 → P1-07 → P1-08, sequential |
| src/js/ui/card-detail.js | Card modal rendering/events | Dev2; same sequential UI ownership window |
| src/js/app/card-actions.js | Card edit/action orchestration | Dev2; same sequential UI ownership window |
| src/js/ui/modals.js, src/js/app/events.js, src/js/app/board-interactions.js, src/js/ui/app-state.js | Shared modal/event/state hot paths | Dev2; only touched under the same sequential UI ownership window |
| New workflow profile/participant UI modules | Profile and assignment views | Dev2; P1-06 |
| New dependency/checkpoint/history UI modules | Blockers, handoffs, checkpoints and activity | Dev2; P1-07 |
| GitHub/deployment refs and Team Lead decision UI | Manual refs, SHA display and protected decision screens | Dev2; P1-08 |
| .github/workflows/playwright.yml and tests/e2e | Frontend CI and browser tests | Dev2 owns writes; Dev1 reviews |

P1-06, P1-07 and P1-08 must be sequential because they share the card modal, REST wrapper and shared UI entry points. One active writer only.

### 11.2 Backend — Dev1 is the sole Phase 1 backend writer

| File or file group | Responsibility | Single writer / task |
|---|---|---|
| src/main/resources/db/migration/V3 through V6 | Ordered Flyway history | Dev1; one migration/PR at a time, P1-02 → P1-03 → P1-04 → P1-05 |
| entity/, repository/, new workflow DTOs | Workflow data model | Dev1; P1-02 → P1-04 sequential |
| BoardAccessService.java and AuthUserArgumentResolver.java | Authorization and actor-context boundary | Dev1; sensitive changes only after trust-path prerequisite is resolved |
| CardService.java and BoardService.java | Card and board domain behavior | Dev1; coordinate P1-03/P1-04; P1-10 import work after P1-03 |
| CardController.java, BoardController.java and new workflow controllers | Backend REST endpoints | Dev1; contract first, implementation P1-02 → P1-05 |
| BoardMapper.java and ImportBoardRequest.java | Board/card mapping and import DTO | Dev1; contract/data model first; P1-10 owns import-specific modifications |
| .github/workflows/maven.yml | Backend CI branch filters | Dev1; branch/CI alignment card |
| Backend unit/integration tests | Domain, API, authorization and migration tests | Dev1; independent QA Dev2 |

Use focused workflow services/controllers where practical. Do not put the complete workflow feature into one existing CardController. Dependent branches must be based on latest develop after prerequisite merges, not on another unmerged feature branch by default.

### 11.3 MCP — Dev1 owns adapter and both protocol catalogs

| File | Responsibility | Single writer / task |
|---|---|---|
| src/main/java/com/ikoyski/webtools/kanban/mcp/client/KanbanApiClient.java | Backend REST requests, validation and error mapping | Dev1; P1-09 |
| src/main/java/com/ikoyski/webtools/kanban/mcp/tools/ModernKanbanToolRepository.java | Modern MCP tools and schemas | Dev1; P1-09 |
| src/main/java/com/ikoyski/webtools/kanban/mcp/config/LegacyMcpConfig.java | Legacy MCP tool catalog and caller-token context | Dev1; P1-09 |
| .github/workflows/maven.yml | MCP CI branch filters | Dev1; branch/CI alignment card |
| MCP tests | Input validation, modern/legacy parity and permission failures | Dev1; independent QA Dev2 |

Every new MCP tool must be present consistently in the modern and legacy catalogs. Preserve caller-token forwarding; do not introduce a shared MCP service account.

### 11.4 Auth Server — no code task approved by P1-01

The signed client-ID claim is missing. Do not change AuthorizationServerConfig or token behavior in this P1-01 design PR or silently add Auth Server work to P1-05. Create a separate Auth Server card after explicit Team Lead approval, with its own developer, QA, scope, tests and PR.

### 11.5 Shared files and integration discipline

Before each PR, provide git diff --name-only develop...HEAD and compare touched paths with all open PRs. Only one active writer per shared hot file. Rebase against latest develop after prerequisites merge, run CI and refresh QA/senior review evidence for the exact new SHA. Never use a stacked branch without explicitly documenting its dependency and revalidation plan.

## 12. Dev1 / Dev2 repository access matrix

This is the target least-privilege plan. The Dev1/Dev2 GitHub accounts are being provisioned, so this table does not claim permissions have already been granted.

| Repository | Dev1 — Claude Code | Dev2 — Antigravity | Reason |
|---|---|---|---|
| ikoyski/kanban | Read | Write | Dev2 owns frontend P1-06–P1-08; Dev1 reviews |
| ikoyski/webtools-kanban-backend | Write | Read | Dev1 owns backend P1-02–P1-05 and P1-10 |
| ikoyski/webtools-kanban-mcp | Write | Read | Dev1 owns MCP P1-09 |
| ikoyski/webtools-auth | Read | Read | No Auth Server implementation is approved in P1-01; any claim change needs separate scope |

For both accounts: no Admin; no direct pushes to develop/master; no force push or branch deletion on protected branches; no self-merge; PRs target develop; CI and independent review required; master is protected for production promotion. GitHub permissions must be granted at Write or Read only as shown, not Admin.

The Team Lead retains repository administration and human approval. ChatGPT and Claude.ai use Team Lead GitHub credentials under the accepted arrangement. Consequently, GitHub author identity, commit author fields, PR review and merge actor are not treated as proof of AI authorship or human authorization. Attribution is explicit in the Kanban/PR handoff; protected human decisions remain separate authenticated web decisions. Claude.ai has no repository access and coordinates only. Dev1 and Dev2 use their own separately provisioned GitHub identities and isolated credential contexts, not the Team Lead session.

## 13. Dependency graph and sequencing

### 13.1 Existing card dependency graph

~~~text
P1-01 Architecture and contract
  |
  v
P1-02 V3 profile / participants / work-item / criteria persistence
  |
  v
P1-03 V4 dependencies / activity / delivery references
  |
  v
P1-04 V5 workflow gates / revision-bound checkpoint evidence
  |
  v
P1-05 V6 protected Team Lead decisions
  |                         \
  |                          \-> P1-09 MCP workflow support
  v
P1-06 frontend profile / assignments / criteria
  |
  v
P1-07 frontend dependencies / blockers / checkpoints / history
  |
  v
P1-08 frontend delivery refs / protected decision screens

P1-03 -> P1-10 import/export preservation

P1-06, P1-07, P1-08, P1-09, P1-10
  |
  v
P1-11 cross-cutting regression / security / E2E
  |
  v
P1-12 pilot deployment and workflow validation

P1-13 search / filters / card summaries (Phase 2; not a Phase 1 dependency)
~~~

P1-09 can run in parallel with frontend work after P1-05 is merged because it is a separate repository, provided the API contract is frozen. P1-10 follows P1-03 and may proceed independently only where it does not overlap open backend schema files. P1-06/P1-07/P1-08 remain sequential because they share UI hot files.

### 13.2 Proposed card changes

These are proposals only. This PR does not create or edit Kanban cards.

1. **P1-01 block list:** change “Blocks P1-02 to P1-13” to “Blocks P1-02 to P1-12”. P1-13 is Phase 2 and its own description says it does not block Phase 1 pilot acceptance.
2. **P1-05 prerequisites:** add dependency on the separately approved Auth Server signed-client-claim task and the resolved backend actor/client trust-path investigation. Do not add Auth Server code into P1-05.
3. **P1-05 scope:** keep the backend allowlist and Team Lead check, but explicitly block implementation until the verified client_id claim and trusted backend actor/client context are available.
4. **P1-11 scope:** test the Auth claim, direct MCP/backend path, spoofed headers, non-Team-Lead Owner rejection, all known MCP clients and an unknown client ID. Keep feature-level tests in P1-02 through P1-10.
5. **CI alignment:** add a task before implementation work depends on develop CI. The task updates frontend Playwright and backend/MCP Maven workflows to run on develop and master, retaining Docker image publishing only on master. Auth workflow already triggers both branches. Verify test deployment uses develop and production service uses master.
6. **P1-10:** retain dependency on P1-03; specify import ID remapping, transactional replacement, authenticated participant resolution, preserving relationships and legacy import compatibility.

### 13.3 Additional scoped cards proposed

**Proposed security card A — Auth Server: add signed OAuth client_id claim**

- Repository: webtools-auth only.
- Set a signed access-token claim from the server-side RegisteredClient during token creation/refresh.
- Preserve uid, email/profile, issuer, scope and resource audience behavior.
- Test kanban-web, kanban-claude, kanban-antigravity and unknown registered clients, including refresh issuance.
- No database migration is expected for a token-customizer-only change.
- Blocks P1-05. Requires explicit Team Lead approval before implementation.

**Proposed security card B — Prove and close backend actor/client trust path**

- Inspect backend, MCP and actual API Gateway/Config Server runtime settings.
- Verify which component validates JWTs, how X-User-* headers are stripped/replaced, and whether direct backend access can bypass that boundary.
- Resolve the mismatch between MCP's documented direct backend calls and backend's gateway-header-based AuthUser resolver.
- Backend must extract identity/client context from verified JWT claims or a demonstrably trusted gateway-only context. Spoofed external headers must be rejected.
- Add tests for spoofed X-User-Id, X-User-Email and client-ID headers, direct backend access and missing/invalid claims.
- Blocks P1-05. Any implementation scope must be separately approved if it extends existing cards.

**Proposed CI card C — Align integration and production branch CI**

- Repositories: kanban frontend, backend and MCP.
- Change GitHub Actions push and pull_request filters to include develop and master.
- Preserve Docker Hub publishing only on push to master. Do not publish test branch artifacts as latest production images.
- Auth Server already triggers CI for both branches; confirm it remains that way.
- Verify develop checks are required for feature PRs into develop and master checks for promotions.
- Add/verify distinct test deployment mapping for develop and production mapping for master.
- This is needed before feature PRs depend on develop CI. Owner/reviewer follows the Dev1/Dev2 repository access matrix; coordinator confirms exact assignment.

**Operational verification item — Render branch mapping**

The frontend README references a Render test site, but no render.yaml or Render service branch setting was visible in the checked-in source. Confirm in the Render control plane that each test service tracks develop and each production service tracks master (or document an approved alternative). Do not infer branch configuration from the service URL.

## 14. Traceability: FR / AC to P1-02 through P1-13

This table maps requirements to implementation and verification cards. P1-11 cross-cutting tests do not replace feature-level tests in P1-02 through P1-10.

| Requirement | Primary implementation card(s) | Verification / pilot card(s) | Notes |
|---|---|---|---|
| FR-1 work item type | P1-02, P1-06 | P1-11 | Structured field and UI |
| FR-2 developer assignment | P1-02, P1-06 | P1-11 | Assignment does not grant access |
| FR-3 independent QA assignment | P1-02, P1-04, P1-06 | P1-11 | Backend enforces distinct participants |
| FR-4 participant catalog | P1-02, P1-06 | P1-11 | DevN identities and ChatGPT relay |
| FR-5 requirement traceability | P1-02, P1-06 | P1-11 | URL and FR/AC links |
| FR-6 task acceptance criteria | P1-02, P1-06 | P1-11 | Normalized criterion records |
| FR-7 dependencies | P1-03, P1-07, P1-09 | P1-11 | Backend validation, UI and MCP |
| FR-8 blocker details | P1-03, P1-07 | P1-11 | Reason, owner, impact, next action |
| FR-9 review checkpoint | P1-04, P1-07, P1-09 | P1-11 | No extra mandatory columns |
| FR-10 QA outcome | P1-04, P1-09 | P1-11 | Assigned QA, exact PR SHA |
| FR-11 senior review | P1-04, P1-09 | P1-11 | Designated senior reviewer or Team Lead only |
| FR-12 merge authorization | P1-05, P1-08 | P1-11 | Signed claim prerequisite; SHA bound |
| FR-13 test deployment evidence | P1-03, P1-04, P1-08, P1-09 | P1-11 | Manual vs verified provenance |
| FR-14 human acceptance | P1-05, P1-08 | P1-11 | Team Lead and kanban-web only |
| FR-15 production authorization | P1-05, P1-08 | P1-11, P1-12 | Explicit auth plus release verification |
| FR-16 structured handoff | P1-03, P1-04, P1-07, P1-09 | P1-11 | Append-only activity |
| FR-17 revision cycle | P1-04, P1-07, P1-09 | P1-11 | New SHA requires fresh evidence |
| FR-18 activity history | P1-03, P1-07, P1-09 | P1-11 | Append-oriented events |
| FR-19 GitHub references | P1-03, P1-08, P1-09 | P1-11 | Manual references in Phase 1 |
| FR-20 deployment references | P1-03, P1-08, P1-09 | P1-11, P1-12 | Test and production distinct |
| FR-21 manual vs verified facts | P1-03, P1-08, P1-09 | P1-11 | No false VERIFIED state |
| FR-22 Ready validation | P1-04 | P1-11 | Backend gate, profile enabled |
| FR-23 In Review validation | P1-04 | P1-11 | PR or non-code exception |
| FR-24 Ready to Merge validation | P1-04, P1-05 | P1-11 | QA/senior/human/deploy/acceptance gates |
| FR-25 Done validation | P1-04, P1-05 | P1-11, P1-12 | Production authorization/verification |
| FR-26 MCP workflow support | P1-09 | P1-11 | Permitted writes, human decisions read-only |
| FR-27 confirmed MCP operations | P1-09 | P1-11 | Backend response is authoritative |
| FR-28 permissions and human-only decisions | P1-05, P1-09 | P1-11 | Positive kanban-web allowlist |
| FR-29 general-purpose operation | P1-02, P1-06 | P1-11 | Profile disabled by default |
| FR-30 workflow profile | P1-02, P1-06 | P1-11 | Per-board opt-in |
| FR-31 card summary (Phase 2) | P1-13 | Phase 2 verification | Deferred |
| FR-32 workflow filters (Phase 2) | P1-13 | Phase 2 verification | Deferred |
| FR-33 responsive experience | P1-06, P1-07, P1-08 | P1-11 | Feature-level browser tests |
| FR-34 import/export | P1-10 | P1-11 | Relationships and legacy exports |

### Acceptance criteria traceability

| AC | Primary implementation card(s) | Verification / pilot |
|---|---|---|
| AC-1 existing boards/cards load | P1-02, P1-03, P1-10 | P1-11 |
| AC-2 profile toggle, no column changes | P1-02, P1-06 | P1-11 |
| AC-3 different developer and QA | P1-02, P1-04, P1-06 | P1-11 |
| AC-4 requirement and FR/AC refs | P1-02, P1-06 | P1-11 |
| AC-5 task-level checkable criteria | P1-02, P1-06 | P1-11 |
| AC-6 dependency validation/cycles | P1-03, P1-07, P1-09 | P1-11 |
| AC-7 blocker detail | P1-03, P1-07 | P1-11 |
| AC-8 checkpoint with no new columns | P1-04, P1-07, P1-09 | P1-11 |
| AC-9 QA/senior review exact revision | P1-04, P1-09 | P1-11 |
| AC-10 SHA-bound merge authorization | P1-05, P1-08 | P1-11 |
| AC-11 deployed SHA/evidence | P1-03, P1-04, P1-08 | P1-11 |
| AC-12 Team Lead acceptance | P1-05, P1-08 | P1-11 |
| AC-13 production authorization/release | P1-05, P1-08 | P1-11, P1-12 |
| AC-14 chronological actor/time history | P1-03, P1-07 | P1-11 |
| AC-15 base permissions plus MCP restriction | P1-05, P1-09 | P1-11 |
| AC-16 confirmed MCP writes/errors | P1-09 | P1-11 |
| AC-17 import/export legacy compatibility | P1-10 | P1-11 |
| AC-18 Phase 2 filters | P1-13 | Phase 2 |
| AC-19 automated tests | P1-02 through P1-10 feature tests | P1-11 cross-cutting |
| AC-20 full lifecycle pilot | P1-12 | Team Lead acceptance |
| AC-21 exact SHA and rebase policy | P1-04, P1-05, P1-09 | P1-11 |
| AC-22 MCP reads but never mutates human decisions | P1-05, P1-09 | P1-11 |
| AC-23 manual exact-SHA confirmation | P1-08 | P1-11 |
| AC-24 signed OAuth client ID and Team Lead | Auth claim prerequisite, trust-path prerequisite, P1-05 | P1-11 |

## 15. Operational limits and review requests

- Repository source access is confirmed for all required source files. The default branch SHAs, develop branch presence and ahead/behind state are in §§2–3.
- The Dev1/Dev2 access matrix is a target plan, not proof that GitHub permissions have been provisioned. The coordinator/Team Lead must confirm after accounts exist.
- No Render workspace was selected, so the live service branch configuration was not inspected. The target mapping is master=production and develop=test; live service mapping remains an operational verification item.
- No runtime API Gateway or Config Server settings were inspected. The actor/client trust-path finding in §4.2 must be validated in the runtime environment.
- ChatGPT has no Kanban MCP access in this session. No claim is made that P1-01 was moved or commented on by ChatGPT. The Team Lead should record the move/comment using the prefix in the task instructions.
- The Team Lead accepted use of Team Lead GitHub credentials for ChatGPT/Claude.ai. This design never treats GitHub identity, PR review, commit author or merge as proof of AI authorship or human decision authorization.
- No implementation code or migration was added to any of the four service repositories.

### Dev1 — independent technical QA

Review this design against the approved requirements revision and inspected source baselines. Focus on:
- Team Lead/OWNER distinction and least privilege
- Missing signed client_id claim and separate Auth Server card
- Backend/MCP actor trust path, header spoofing and direct service access
- Flyway migration order, relational constraints and backward compatibility
- API/MCP consistency and modern/legacy tool parity
- revision/SHA staleness, append-only history and protected decision rules
- import/export ID remapping and transaction safety
- branch comparison, CI mismatch and file ownership

QA report must include the PR head SHA and PASS / FAIL / NEEDS CLARIFICATION, findings, evidence, checks performed and limitations. QA is read-only.

### Claude.ai — coordinator review

Review:
- FR/AC traceability and task boundaries
- P1-02 to P1-13 dependency sequencing
- removal of P1-13 from P1-01's Phase 1 block list
- single-writer assignments for hot UI/backend/MCP/migration files
- separate security and CI prerequisite cards
- card scope and owner/QA changes before any implementation task enters Ready

Claude.ai has no repository access under the current arrangement; the Team Lead provides this PR and source links for coordinator review and records the review result in Kanban.

### Team Lead

1. Reconcile the requirements file approval metadata with the task's approved baseline designation.
2. Review and approve/adjust the architecture decisions and proposed scoped follow-up cards.
3. Have the coordinator update P1-01 dependency/block metadata: P1-13 is Phase 2 and must not block Phase 1 work.
4. Ensure CI on develop and master is in place before feature PRs depend on develop checks.
5. Confirm the live Render service mappings before the first test deployment.
6. Keep P1-05 protected decision implementation blocked until the signed client claim and verified backend actor/client trust path are resolved and separately approved.

**Draft only. No service source was changed, no Kanban status was changed, and nothing was merged.**
