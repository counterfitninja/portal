# Feature Specification: Portal - Team Productivity Portal

**Feature Branch**: `main` (existing branch; no branch-creation hook configured)

**Created**: 2026-10-13

**Status**: Draft

**Input**: User description: "Develop Portal, a team productivity portal. It allows multiple teams to have a single pane of glass to monitor for the Product and technology teams. It will allow different integration with third-party tool sets and build our own CMDB along with integration with ADO, New Relic and Azure SRE." Clarification: ADO and New Relic are read-only with links to source tools; users must also be able to ask SRE questions.

## Clarifications

### Session 2026-10-13

- Q: How long should Portal retain a user's private SRE conversation so they can return to it and ask follow-up questions? → A: Keep only for the active session.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Monitor Team Delivery and Service Health (Priority: P1)

As a Product or technology team member, I want one team-scoped overview of delivery progress, owned services, and operational issues so I can identify what needs attention without visiting several tools.

**Why this priority**: A shared view of delivery and service health is Portal's central value.

**Independent Test**: With representative delivery and health records available for two teams, select a team, identify its delivery risks and unhealthy services, and navigate to supporting source records.

**Acceptance Scenarios**:

1. **Given** a user belongs to two teams with different services and work items, **When** they switch the selected team, **Then** the overview shows only that team's records and explicitly shared services.
2. **Given** available delivery and operational information, **When** a member opens the overview, **Then** they see work-item counts by source state, blocked or overdue items where supported by the source, service health, active alerts or incidents, and each section's source and last successful refresh time.
3. **Given** a user can access several teams, **When** they select a combined view, **Then** information is grouped by team and a shared service is identifiable as one service with multiple owners rather than unrelated duplicate services.
4. **Given** one source is unavailable or has no data, **When** the overview loads, **Then** other sections remain usable and the affected section distinguishes unavailable, stale, and empty information from a healthy state.
5. **Given** a visible delivery or operational record, **When** a member selects its source link, **Then** they are directed to the corresponding source record without Portal modifying it.

---

### User Story 2 - Maintain a Shared CMDB (Priority: P1)

As a technology team owner, I want a configuration management database (CMDB) of services and supporting resources, with ownership and dependencies, so Product and technology teams can understand responsibility and potential impact.

**Why this priority**: Reliable ownership and dependency information makes the monitoring view actionable and establishes Portal's own maintained inventory.

**Independent Test**: Without any external integration enabled, create two services and a resource, assign owners and dependencies, and find the resulting inventory and impact information.

**Acceptance Scenarios**:

1. **Given** an authorized team owner, **When** they create or edit a configuration item, **Then** Portal stores its unique identity, name, type, description, owning team or teams, environment, lifecycle status, and optional external references; missing required fields are identified and prevent saving.
2. **Given** existing items, **When** an owner records that one item depends on another, **Then** both the dependency and its direction are visible from the relevant item details, including direct dependants.
3. **Given** an inventory containing several teams and environments, **When** a member searches or filters by name, type, owner, environment, or lifecycle status, **Then** they see matching items within their permitted scope.
4. **Given** an external record is mapped to an item, **When** the source is refreshed repeatedly, **Then** the mapping remains attached to that item without creating duplicates or overwriting Portal-maintained ownership and dependency information.
5. **Given** an item is retired, **When** its owner archives it, **Then** its history and relationships remain inspectable, it is excluded from active inventory by default, and an authorized owner can restore it.
6. **Given** an unauthorized edit, duplicate external mapping within a connection, or a self-dependency, **When** a user attempts to save, **Then** Portal rejects the change with an explicit explanation and leaves the prior record unchanged.

---

### User Story 3 - Connect and Govern Third-Party Sources (Priority: P2)

As a Portal administrator, I want to connect Azure DevOps (ADO), New Relic, and Azure SRE Agent and map them to teams and CMDB items so the overview reflects the correct source information.

**Why this priority**: These named sources deliver the initial integrated experience, while connection governance prevents misleading or unauthorized information.

**Independent Test**: Configure each named connection against a representative accessible source, map its scope, refresh it, simulate failure, and verify visibility and recovery without requiring other sources to succeed.

**Acceptance Scenarios**:

1. **Given** an administrator with valid source access, **When** they configure and test a connection, **Then** Portal reports success or an actionable failure and permits assignment of its authorized source scope to teams.
2. **Given** a connected ADO project, **When** delivery information is refreshed, **Then** users see work-item identity, title, source state, assignee when provided, and source links; blocked status, target dates, iteration, and recent pipeline outcomes are shown when provided, otherwise marked as not supplied.
3. **Given** a connected New Relic scope, **When** operational information is refreshed, **Then** users see monitored service identity, source-reported health when available, active alerts, available performance indicators with units and observation times, and source links; unavailable fields are not inferred as healthy or zero.
4. **Given** a connected Azure SRE Agent scope, **When** available incident and investigation records are refreshed, **Then** users see their source identity, summary, status, related service references when supplied, and source links where available.
5. **Given** a connection with expired access or a failed refresh, **When** an administrator inspects it, **Then** Portal shows the failure reason and last successful refresh, retains previously retrieved information as stale, and offers a refresh retry.
6. **Given** a connection is disabled, **When** members view affected information, **Then** Portal stops refreshing it, marks retained information as disconnected, and prevents new SRE questions through that connection.
7. **Given** a source service cannot be matched confidently, **When** its information is retrieved, **Then** it is listed as unmapped for administrator review rather than silently assigned to a CMDB item.

---

### User Story 4 - Ask SRE Questions in Context (Priority: P2)

As a technology or Product team member, I want to ask SRE questions from Portal about a permitted service, incident, or investigation so I can understand operational issues and decide what to do next.

**Why this priority**: The user explicitly requires an interactive SRE capability beyond passive monitoring.

**Independent Test**: With a permitted Azure SRE Agent connection and representative service context, ask a question, read a response, ask a follow-up, and verify handling of unavailable evidence and unauthorized context.

**Acceptance Scenarios**:

1. **Given** a member selects an accessible team and service or incident, **When** they submit an SRE question, **Then** Portal shows the selected context, submits the question to the associated Azure SRE Agent, and displays a pending state followed by the response or an explicit failure.
2. **Given** a successful answer, **When** the member reads it, **Then** they can distinguish the agent's assessment from source-reported facts and inspect supporting references when supplied; absent references and unavailable evidence are explicitly identified.
3. **Given** a conversation with a previous response, **When** the member asks a follow-up, **Then** the conversation retains its permitted context and prior exchanges for the active Portal session only; choosing a different team or service starts a separate conversation.
4. **Given** a member requests remediation or a change to an external system, **When** Portal processes the request, **Then** it does not execute the change, states that the capability is advisory, and may provide recommended next steps or links for action outside Portal.
5. **Given** an inaccessible service or context outside the selected team's authorized source scope, **When** a member attempts a question or opens a conversation, **Then** Portal denies access without disclosing restricted information or sending restricted context to the agent.
6. **Given** an unavailable agent, incomplete response, or failed question, **When** the request ends, **Then** Portal presents an explicit failure or incomplete-response state rather than a fabricated answer, preserves the question, and offers retry without automatic duplicate submission.

---

### User Story 5 - Manage Team Access and Accountability (Priority: P1)

As a Portal administrator or team owner, I want access governed by team membership and changes attributable to their author so shared visibility does not expose restricted information or obscure responsibility.

**Why this priority**: Multi-team visibility is only viable when users can trust its access boundaries and ownership.

**Independent Test**: Use administrator, team-owner, and member accounts across two teams to verify permitted reads and writes, denied cross-team access, and recorded changes.

**Acceptance Scenarios**:

1. **Given** a signed-in member, **When** they use Portal, **Then** they can read permitted team information and ask scoped SRE questions but cannot change memberships, connections, or CMDB records.
2. **Given** a team owner, **When** they update team membership or a CMDB item, **Then** the change is permitted only for teams they manage; changing a shared item requires ownership authority for all its owning teams.
3. **Given** an administrator, **When** they create a team, assign owners, configure connections, or inspect cross-team information, **Then** Portal permits the action and records administrative changes.
4. **Given** recorded membership, CMDB, mapping, or connection changes, **When** an authorized owner or administrator reviews history, **Then** they can identify who changed what and when without seeing connection secrets.
5. **Given** membership is revoked, **When** the former member next requests that team's information or a related conversation, **Then** access is denied even through previously saved links.

### Edge Cases

- A new team has no CMDB items or connections: show an explicit empty state and authorized setup actions, not a healthy summary.
- A service belongs to several teams: show all permitted owners and preserve one canonical item; restricted owners' details are not disclosed.
- Source records contain duplicate names or conflicting health signals: use scoped external identities for matching and display each source's signal separately rather than choosing an unexplained overall status.
- Refreshes repeat or arrive out of order: retain the latest known source version or observation time, avoid duplicates, and do not replace newer information with older observations.
- A source record disappears: mark its source reference as no longer present; do not delete its CMDB item automatically.
- A dependency graph contains a cycle: preserve valid relationships and display impact information without repeated items or unbounded traversal.
- A source denies access or limits requests: show the affected scope and refresh failure without exposing secrets; never broaden permissions automatically.
- Membership or source scope changes during an SRE conversation: recheck access before each question and conversation read; previously available context does not grant continuing access.
- An SRE answer is empty, incomplete, or unsupported by evidence: identify that condition; do not turn it into a confirmed cause or healthy service state.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Portal MUST require users to sign in and enforce administrator, team-owner, and member permissions as defined in User Story 5 for every read, change, and SRE interaction.
- **FR-002**: Administrators MUST be able to create teams, assign owners, and manage membership; team owners MUST be able to manage membership only within their managed teams.
- **FR-003**: Users MUST be able to switch among permitted teams and select a combined view limited to those teams, with shared services identifiable without duplicate canonical items.
- **FR-004**: Portal MUST provide a team overview containing delivery progress, service health, and active operational issues, with source attribution, source links where available, and last successful refresh times.
- **FR-005**: Portal MUST distinguish current, stale, unavailable, disconnected, and empty source information. Information becomes stale when its configured refresh interval has elapsed without success; the default interval MUST be five minutes.
- **FR-006**: Portal MUST keep unaffected source sections usable when another source fails and MUST NOT infer healthy status or zero values from missing information.
- **FR-007**: Authorized owners and administrators MUST be able to create, read, update, archive, and restore CMDB items. Identity, name, type, at least one owning team, environment, and lifecycle status MUST be required; description and external references MUST be supported.
- **FR-008**: The CMDB MUST support services, applications, and infrastructure resources, team ownership, directed dependencies, direct dependants, and external record mappings.
- **FR-009**: Users MUST be able to search and filter permitted CMDB items by name, type, owning team, environment, and lifecycle status, with archived items excluded by default.
- **FR-010**: Portal MUST reject invalid CMDB changes, self-dependencies, and duplicate external mappings within the same connection without changing the prior valid record. Cyclic dependencies MUST remain inspectable without repeated items or unbounded traversal.
- **FR-011**: Portal MUST preserve Portal-maintained ownership, descriptions, and dependencies during source refreshes and MUST NOT delete a CMDB item solely because its external record disappears.
- **FR-012**: Administrators MUST be able to configure, test, enable, disable, and retry connections independently for ADO, New Relic, and Azure SRE Agent, assign source scopes to teams, and set refresh intervals.
- **FR-013**: The ADO integration MUST expose the delivery fields and optional source-supported indicators specified in User Story 3, preserving source states and identifying fields not supplied by the source.
- **FR-014**: The New Relic integration MUST expose service identity, available health signals, active alerts, available performance indicators with units and observation times, and source references, identifying fields not supplied by the source.
- **FR-015**: The Azure SRE Agent integration MUST expose accessible incident and investigation information supplied by the connected scope and support users submitting SRE questions from Portal.
- **FR-016**: Administrators MUST be able to inspect unmapped external records and associate them with permitted CMDB items. Ambiguous records MUST NOT be mapped automatically.
- **FR-017**: Refreshes MUST be repeatable without duplicate records, preserve source attribution, and avoid replacing newer observations with older information.
- **FR-018**: Connection failures MUST include an actionable failure reason and last successful refresh time where one exists; retained information MUST be marked stale or disconnected rather than presented as current.
- **FR-019**: Connection secrets MUST NOT appear in team views, histories, SRE context, or user-visible failure messages; only administrators MUST be able to manage connection access.
- **FR-020**: Users MUST be able to submit an SRE question with an explicit permitted team and optional service, incident, or investigation context, see pending and terminal states, and ask follow-up questions within the same conversation.
- **FR-021**: SRE conversations MUST be private to their submitting user, constrained by current team and source access, and readable only while that access remains valid. Conversation history MUST be available only for the active Portal session and MUST NOT be retained after it ends. Changing team or selected context MUST start a separate conversation.
- **FR-022**: SRE responses MUST be labeled as agent assessments, show supporting references when supplied, and explicitly identify absent references, incomplete responses, and unavailable evidence.
- **FR-023**: Portal MUST NOT create or update ADO work items, alter New Relic configuration, execute remediation, or permit the SRE question capability to change external systems in this release. Advisory questions and follow-ups are the only permitted interactive external capability.
- **FR-024**: Failed SRE questions MUST show an explicit failure, preserve the question for user-initiated retry, and MUST NOT be automatically resubmitted.
- **FR-025**: Portal MUST record actor, time, affected entity, and changed non-secret values for team membership, CMDB, external mapping, and connection configuration changes. Owners MUST see history only for their managed teams; administrators MUST see all change history.
- **FR-026**: Archived CMDB records and their relationships and history MUST remain inspectable by authorized users and MUST be excluded from active views unless explicitly requested.
- **FR-027**: A shared CMDB item MUST be editable by an administrator or an owner authorized for every owning team; adding or removing an owning team MUST follow the same rule.

### Key Entities *(include if feature involves data)*

- **User**: A signed-in person with an administrative role or team memberships; owns private SRE conversations.
- **Team**: A Product or technology group with a name, owners, members, source scopes, and owned configuration items.
- **Membership**: A user's association with a team and member or owner role; controls current access.
- **Configuration Item**: A canonical service, application, or infrastructure resource with identity, metadata, owners, environment, lifecycle, and external references.
- **Dependency**: A directed relationship between two configuration items used to explain reliance and potential impact.
- **Integration Connection**: A configured external source, its access configuration, permitted team scopes, enabled state, refresh interval, and connection status.
- **External Mapping**: An association between a connection-scoped external record identity and one CMDB item.
- **Delivery Record**: Source-attributed work-item or pipeline information associated with a team and optionally a configuration item.
- **Operational Record**: Source-attributed health, alert, incident, investigation, or performance information with observation time and optional CMDB association.
- **SRE Conversation**: A user's scoped sequence of questions and agent responses during the active Portal session, including context, request states, and evidence references.
- **Change History Entry**: An attributable, timestamped record of a non-secret governance or inventory change.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 90% of representative Product and technology users can identify a selected team's delivery risk, an unhealthy service, and its permitted owner within two minutes using Portal alone.
- **SC-002**: At least 90% of representative team owners can create a service, assign ownership, and add a dependency within five minutes without assistance.
- **SC-003**: With 20 teams, 5,000 configuration items, and 100 simultaneous users, at least 95% of team-view, CMDB-search, and item-detail requests show usable information within three seconds, excluding time spent waiting for a new external refresh or SRE answer.
- **SC-004**: Under normal source availability and the default refresh interval, at least 95% of source changes eligible for retrieval appear in Portal within ten minutes of becoming available to the connected scope.
- **SC-005**: In all acceptance tests covering access denial, failed refreshes, disabled connections, and missing evidence, Portal prevents unauthorized disclosure and clearly identifies unavailable or stale information instead of reporting a healthy or successful result.
- **SC-006**: At least 90% of representative users can submit a contextual SRE question and a follow-up without leaving Portal or assistance; every completed request displays either an agent response with its evidence availability or an explicit failure.
- **SC-007**: In all tested requests for external changes, including remediation phrased as SRE questions, Portal performs no external mutation and explains its advisory-only boundary.
- **SC-008**: Across ten repeated refreshes of unchanged source information, no duplicate CMDB items or external mappings are created, and all Portal-maintained ownership and dependencies remain unchanged.
- **SC-009**: At least 80% of representative pilot users rate Portal's usefulness for understanding team delivery and operational health at four or higher on a five-point scale.

## Assumptions

- The first release serves multiple teams within one organization; separate customer organizations and cross-organization tenancy are outside this specification.
- ADO means Azure DevOps. Azure SRE is interpreted as Azure SRE Agent because the user explicitly wants to ask SRE questions.
- The user's read-only preference applies to source monitoring and external changes. Asking SRE questions is an explicit exception for advisory interaction, not authorization for investigation workflows with side effects or remediation.
- Source access for SRE questions can be restricted to advisory use. If the connected agent cannot guarantee no external mutations, administrators must not enable the question capability for that connection.
- The organization supplies a trusted sign-in system, valid third-party access, representative source data, and authorized source scopes. Specific identity and hosting choices belong to planning.
- Initial connections cover ADO delivery visibility, New Relic operational visibility, and Azure SRE Agent information and questions. Additional third-party integrations are future scope, not a requirement for a generic connector marketplace in this release.
- Initial CMDB population is manual creation and explicit mapping of external records; automatic infrastructure discovery, full asset lifecycle procurement, and inventory reconciliation workflows are future scope.
- Portal does not replace source tools, implement task management, send new incident notifications, or become an incident-response execution system in this release.
- Each source remains authoritative for its reported delivery and operational signals; Portal is authoritative for its manually maintained inventory, ownership, and dependencies.
- Owners, members, and administrators are initial roles; arbitrary custom role design and public unauthenticated views are outside this release.
- Change-history retention and organizational compliance policies must be established during planning before production use; access and secret-protection requirements in this specification apply regardless of those choices.
- The project constitution sets Azure-native hosting, Entra ID/OIDC, managed identity, Key Vault, API-first, observability, and audit requirements; confirm concrete architecture and applicable scope during planning.
