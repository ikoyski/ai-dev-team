# Developer Instance Identity and Access

**Status:** Shared operating standard  
**Scope:** All projects and all human/AI-assisted development environments  
**Related standards:** [AI Development Team Onboarding](AI-DEVELOPMENT-TEAM-ONBOARDING.md) · [Workflow and Branching](AI-DEVELOPMENT-WORKFLOW-AND-BRANCHING.md)

## 1. Purpose

This standard gives every independently operating developer instance a stable identity that can be traced across the work tracker, source-control activity, tool access, and review handoffs. It is designed for an open-ended developer pool: Dev1, Dev2, Dev3, … DevN. There is no predefined maximum number of developers.

A developer identity represents an independently operating contributor instance, not a particular model, vendor, or tool.

## 2. Developer IDs

- Assign each independently operating developer instance a unique, stable ID: **Dev1**, **Dev2**, … **DevN**. Use the lowercase form `devN` in machine-readable labels and the display form `DevN` in prose and handoffs.
- Allocate the next unused number when onboarding a new independent instance. Do not reuse a retired ID.
- Keep the ID when the same instance is upgraded, changes model, changes provider routing, or is reconfigured.
- Assign a new ID when a second independent runtime/workspace is provisioned, even when it uses the same tool and model as an existing developer.
- Track status as Active, Paused, or Retired. Retain retired IDs in the access-controlled roster for audit history.
- Tool, model, provider, role, and project assignment are attributes of DevN; none is the identity itself.

Examples:
- Dev1 — Claude Code (OpenRouter may be used as its model/provider route).
- Dev2 — Antigravity.
- Dev3 — a second, independently provisioned Claude Code instance.
- Dev4 — a second, independently provisioned Antigravity instance.

The roster is open-ended. Examples do not reserve a maximum team size or require alternating tool types.

## 3. Identity layers

Each active DevN must have a separate, attributable identity at each system where it acts.

| Layer | Required identity and purpose |
|---|---|
| Team identity | Stable DevN identifier used in assignments, handoffs, reviews, and audit records |
| Google / SSO | A unique account for the instance, provisioned under the identity provider's applicable policies; it must map to a distinct Kanban user |
| Kanban | A distinct Kanban user ID and individually granted board membership/permissions |
| MCP OAuth application | The OAuth `client_id` for the MCP client application type; this identifies the connecting application, not the DevN user |
| GitHub | A separately attributable, organization-approved GitHub identity or automation identity mapped to DevN; record the actual GitHub actor that performs each operation |
| Runtime | An isolated workspace, browser profile, and credential/token store for the instance |

Do not use a shared human account as the identity for multiple DevN instances. The human Team Lead retains a separate human identity and remains the only authority for human-only approvals.

The Human Team Lead uses their own human identity and the separately registered web client (for the current Kanban integration, `kanban-web`). This identity is never shared with DevN instances.

### 3.1 Google / SSO and Kanban identity

- Provision one unique Google/SSO account per DevN where Google is the configured identity provider. Prefer managed organizational accounts when available and follow the provider's account policies.
- Each account must resolve to a distinct Kanban user record and stable backend user ID. Verify this by signing in and confirming the resulting user identity; a different display name or email alias alone is not enough.
- Add each DevN user to only the project boards and roles required for its work.
- Developer accounts must not receive Team Lead, owner, merge-authorization, acceptance, or production-authorization privileges.
- Do not share browser profiles, login sessions, refresh tokens, or access-token caches between DevN instances.

### 3.2 MCP OAuth client registration

OAuth client registrations are grouped by **MCP client application/integration**, not by developer instance.

For the Kanban MCP integration:

| Developer instance | MCP client application | OAuth client ID | Authenticated Kanban identity |
|---|---|---|---|
| Dev1 — Claude Code | Claude Code MCP integration | `kanban-claude` | Dev1's own Google/SSO account |
| Dev2 — Antigravity | Antigravity MCP integration | `kanban-antigravity` | Dev2's own Google/SSO account |
| Dev3 — another Claude Code instance | Claude Code MCP integration | `kanban-claude` | Dev3's own Google/SSO account |
| Dev4 — another Antigravity instance | Antigravity MCP integration | `kanban-antigravity` | Dev4's own Google/SSO account |
| DevN | Existing supported MCP client type | Matching existing client ID | DevN's own Google/SSO account |

Rules:

1. Reuse the existing client ID when onboarding another instance of an already-supported MCP client type.
2. Each DevN must complete its own OAuth authorization flow and keep its tokens in its own isolated runtime.
3. Never copy an OAuth token, refresh token, browser session, or PKCE session between instances.
4. A new DevN using an existing MCP client type should not require a new OAuth client registration or application code change, provided the existing registration supports that client's redirect URI, resource, and scopes.
5. Add a new OAuth client registration only when introducing a genuinely new MCP client integration or when a distinct registration is justified by a documented security or policy requirement.
6. The OAuth client ID identifies the client application; the authenticated token subject/user ID identifies the actor. Audit and authorization must use the authenticated user identity, not infer the user from the client ID.

### 3.3 GitHub identity and attribution

Every DevN needs attributable GitHub activity, but the implementation must comply with GitHub's current terms, the organization's policy, and the available GitHub plan.

- Map each DevN to a distinct, explicitly approved GitHub attribution identity. Where permitted by GitHub and organization policy, use a dedicated user/machine identity for the instance; a separately configured GitHub App identity is another option.
- Do not create extra personal accounts merely to bypass platform rules. Do not share the Team Lead's personal credentials, personal access tokens, SSH private keys, or browser session across agents.
- If separate GitHub actors are provisioned per DevN, record the mapping and use the corresponding least-privilege credentials for that instance.
- If one shared GitHub App performs actions for several DevN instances, record both the actual GitHub App actor and the initiating DevN in the PR body, commit metadata/trailers, or an attributable handoff. A shared App login is not a distinct GitHub actor per DevN; document this as a reduced-attribution arrangement rather than claiming separate GitHub accounts.
- Configure Git author name/email consistently with the approved identity. Where supported, use verified commit signing. Git author fields alone are self-asserted metadata and must not be treated as authentication.
- Preserve the distinction between developer activity and human authorization. The Team Lead uses their own human GitHub identity for human-only review/approval and production authorization.

## 4. Kanban assignment and audit conventions

The work tracker must distinguish a specific developer instance from its tool family.

Use these labels when the tracker supports labels:
- `dev:dev1`, `dev:dev2`, … `dev:devN` — primary developer.
- `qa:dev1`, `qa:dev2`, … `qa:devN` — independent QA reviewer.
- Optional tool-family labels such as `agent:claude-code` or `agent:antigravity` may supplement, but must not replace, the DevN identity labels.

If the tracker has native owner/reviewer fields, use them and keep the DevN identity in the associated user mapping. If it does not, record the developer and QA labels or an equivalent structured card description. Labels are descriptive metadata and do not grant permissions.

Every task must have exactly one primary DevN owner and a different DevN reviewer. The reviewer must not be the same instance that implemented the change. Prefer a different tool/provider for higher-risk changes when practical, but independence is defined by a different contributor instance and separate work context—not merely a different product name.

Use explicit attribution in comments and handoffs, for example:
- `[Claude Code | Dev1 | Developer]`
- `[Antigravity | Dev2 | QA]`
- `[Claude Code | Dev3 | QA]`

Do not use only labels such as `Claude Code` or `Antigravity` when multiple instances of that tool are active.

## 5. Runtime and credential isolation

Each DevN must have its own:
- Repository clone or worktree and writable workspace.
- Operating-system/container user or equivalently isolated home/configuration directory.
- Browser profile and Google/SSO login session.
- MCP OAuth token store and local MCP client configuration.
- GitHub credential context, SSH key, or App authorization as approved.
- Git author configuration and, where used, signing identity.
- Tool-specific state that could contain credentials, conversation history, or uncommitted changes.

Never mount another DevN's writable workspace or credential directory into an instance unless a specific, approved workflow requires read-only access. Do not store secrets in this repository, task cards, comments, prompts, or developer registry entries. Store credentials in the approved secret manager or runtime-specific protected storage.

## 6. Onboarding a new DevN

The coordinator or authorized operator must complete these steps before the instance receives implementation work:

1. Allocate the next unused DevN ID and add a non-secret roster entry.
2. Record the agent application, harness, provider/model route, primary skills, and status.
3. Provision a unique Google/SSO account and verify it creates a distinct Kanban user.
4. Add that user to only the required boards and assign least-privilege permissions.
5. Select the existing MCP OAuth client ID that matches the agent application type. Do not create a client ID solely because the developer number is new.
6. Complete an independent OAuth login and verify that MCP actions execute as the new Kanban user, not the Team Lead or another DevN.
7. Assign an approved GitHub identity/automation method, repository permissions, and DevN attribution. Confirm which identity GitHub records as the actor.
8. Provision isolated workspace, browser profile, token storage, and Git credentials.
9. Assign an independent QA pairing and verify that the reviewer is a different DevN instance.
10. Record the completed identity/access mapping in the access-controlled team roster and confirm the instance is ready.

If the new agent uses an MCP client type not already registered, treat that as an integration onboarding task: define the redirect URI, resource, scopes, client authentication and security requirements, then update the Auth Server configuration/implementation through the normal review process.

## 7. Roster and privacy

Maintain an access-controlled roster using [Developer Instance Registration Template](templates/DEVELOPER-INSTANCE-REGISTRATION-TEMPLATE.md). The roster should contain only the information needed for ownership, access administration, and audit.

Do not commit actual passwords, tokens, private keys, recovery codes, OAuth secrets, or other credentials. Do not place personal account details or stable provider subject identifiers in this public standards repository; keep operational mappings in the approved private operations store.

## 8. Offboarding and rotation

When a DevN is paused or retired:
- Remove or suspend board membership and other project access as appropriate.
- Revoke its OAuth grants and tokens; rotate or revoke dedicated credentials and keys.
- Remove GitHub repository access or App permissions that are no longer required.
- Archive its workspace and preserve task/PR/commit references required for audit.
- Mark the ID Paused or Retired; never recycle it.
- Reassign open tasks and identify a new independent QA reviewer before work resumes.

## 9. Acceptance checklist

A DevN is ready for assignment only when:
- [ ] A unique, stable DevN ID is recorded.
- [ ] The instance has an isolated workspace and credential store.
- [ ] Its Google/SSO login maps to a distinct Kanban user ID.
- [ ] Its Kanban board membership and permissions are least-privilege.
- [ ] It uses the correct shared MCP OAuth client for its application type and an independent token store.
- [ ] MCP operations are attributed to the expected Kanban user.
- [ ] Its GitHub identity/automation actor and DevN attribution method are documented.
- [ ] GitHub permissions and local Git identity are configured appropriately.
- [ ] A different DevN is assigned as independent QA for its work.
- [ ] No human Team Lead credentials or approval privileges are shared with the developer instance.
