# 1. GitLab

## 1.1 General Planning

1. General planning operates at the GitLab group level and covers cross-project coordination
1. All planning artifacts (scopes, roadmaps, cross-project dependencies) MUST be managed at the group level
1. Project-level planning (Section 1.2) handles execution within a single repository

### 1.1.1. Epics

1. An epic MUST represent a scope document as defined in Planning Section 1
1. The epic description MUST contain or link to the full scope document (problem statement, success criteria, boundaries, assumptions, constraints, stakeholders, impact, dependencies, risks)
1. Epics MUST be created before any child issues are raised
1. One epic per scope — splitting a scope across multiple epics is PROHIBITED
1. Epic labels MUST include the project code (e.g., `AUTH`, `PIPE`, `PLAT`)
1. Epic status MUST be kept current — stale epics (no activity for 14 days without explanation) MUST be flagged in standups
1. Closing an epic MUST require sign-off from the stakeholder identified in the scope document

### 1.1.2. Epic Boards

1. A single epic board MUST be maintained at the group level for cross-project visibility
1. Board columns MUST reflect epic lifecycle stages:
   1. `Draft` — scope document in progress, not yet approved
   1. `Approved` — scope approved, work may begin
   1. `In Progress` — child issues are actively being worked
   1. `Review` — all child issues complete, awaiting stakeholder sign-off
   1. `Done` — stakeholder sign-off received
1. Epics MUST NOT remain in `Draft` for more than 10 business days without escalation
   > Review: confirm escalation path and draft duration limit

### 1.1.3. Issues

1. Group-level issues MUST be used for cross-project concerns that do not belong to a single repository (e.g., infrastructure requests, cross-service integration tasks, process changes)
1. Group-level issues MUST reference the parent epic they belong to
1. Issues without a parent epic MUST be triaged within 5 business days — orphaned issues are PROHIBITED

### 1.1.4. Issue Boards

1. A group-level issue board MUST be maintained for cross-project issue tracking
1. Board columns MUST align with the workflow stages defined in Section 1.2.2
1. Filters MUST be available by project, assignee, and milestone

### 1.1.5 Roadmap

1. The GitLab roadmap view MUST be used to visualise epic timelines across the group
1. Every epic MUST have a start date and due date derived from the scope document's three-point estimate (using the expected estimate)
1. Roadmap MUST be reviewed at the start of every iteration to identify scheduling conflicts and dependency risks
1. Roadmap changes MUST be communicated to all stakeholders identified in the affected scope documents

### 1.1.6 Iterations

1. Iterations MUST be defined at the group level so that all projects within the group share the same cadence
1. Iteration length MUST be consistent within a group
   > Review: confirm iteration length (1 week, 2 weeks)
1. Every iteration MUST have a defined goal linked to one or more epics
1. Iteration goals MUST be documented in the iteration description
1. Unfinished work at iteration close MUST be explicitly carried forward or deprioritised — silent rollover is PROHIBITED
1. A retrospective MUST be held at the end of every iteration
   > Review: confirm retrospective format and documentation requirements

### 1.1.7 Milestones

1. Group-level milestones MUST represent major deliverables or release targets that span multiple projects
1. Milestones MUST be linked to the deliverable documents defined in Planning Section 4
1. Milestone due dates MUST align with the roadmap
1. Milestone completion MUST require all linked issues to be closed and all acceptance criteria met

### 1.1.8 Issue Requirements

1. GitLab's requirements feature MUST be used to capture requirements as defined in Planning Section 2
1. Each requirement MUST use the identification format defined in Planning Section 2.3 (`PRJ-TYP-###`)
1. Requirements MUST be linked to the issues that implement them — unlinked requirements MUST be flagged
1. Requirements MUST be linked to the issues that verify them (test cases)
1. Requirement status MUST track the lifecycle: `Draft` → `Approved` → `Implemented` → `Verified`
1. Traceability (source → requirement → implementation → verification) as defined in Planning Section 2.7 MUST be maintained through GitLab's linking features

### 1.1.9 Wiki

1. The group-level wiki MUST serve as the single location for cross-project documentation that does not belong in a repository
1. Wiki pages MUST follow the documentation standards defined in Architectural Principles Section 11
1. Wiki content MUST be reviewed for staleness every iteration — outdated content MUST be updated or archived

#### 1.1.9.1 Plan.md

1. A `Plan.md` MUST exist in the group wiki as the high-level plan for the group's work
1. `Plan.md` MUST contain:
   1. Group mission and objectives
   1. Active epics with current status and owner
   1. Key milestones and target dates
   1. Cross-project dependencies and their resolution status
   1. Risks and mitigations at the group level
1. `Plan.md` MUST be updated at the start of every iteration
1. `Plan.md` is a living document — it reflects current state, not historical record (history is maintained through git)

## 1.2 Project Planning

1. Project-level planning operates within a single GitLab repository
1. Project planning manages the execution of work items derived from group-level epics and requirements

### 1.2.1. Issues

1. Project-level issues MUST represent individual work items (features, bugs, tasks, spikes)
1. Every issue MUST be linked to a parent epic at the group level
1. Issue titles MUST be written in imperative mood (`Add authentication middleware`, not `Authentication middleware` or `Added authentication middleware`)
1. Every issue MUST contain:
   1. Description: clear statement of what needs to be done and why
   1. Acceptance criteria: verifiable conditions for completion (using Given/When/Then where applicable)
   1. Requirement links: references to the requirements this issue implements (using the `PRJ-TYP-###` identifiers)
   1. Labels: type (`feature`, `bug`, `task`, `spike`), priority, project code
   1. Estimate: weight or time estimate
      > Review: confirm estimation method (story points, hours, t-shirt sizes)
1. Issues MUST be sized to be completable within a single iteration — issues exceeding this MUST be decomposed
1. Spike issues MUST have a defined timebox and expected output documented in the description

### 1.2.2. Issue Boards

1. Every project MUST maintain an issue board with the following columns:

| Column | Meaning | Entry Criteria | Exit Criteria |
|--------|---------|----------------|---------------|
| `Backlog` | Triaged and ready for prioritisation | Issue has description, acceptance criteria, and labels | Assigned to an iteration |
| `To Do` | Committed for current iteration | Assigned to iteration and assignee | Work has started |
| `In Progress` | Actively being worked on | Assignee has started implementation | PR opened |
| `In Review` | PR submitted, awaiting review | PR opened and linked to issue | PR approved and merged |
| `Done` | Work complete and merged | PR merged, all acceptance criteria met | Issue closed |

1. Issues MUST move through columns in order — skipping columns is PROHIBITED except for issues closed as `won't fix` or `duplicate`
1. WIP limits MUST be enforced on `In Progress` and `In Review` columns
   > Review: confirm WIP limits per column

### 1.2.3 Milestones

1. Project-level milestones MUST represent release targets for the individual project
1. Milestones MUST align with the project's semantic versioning (Versioning Section 11.1)
1. Every issue MUST be assigned to a milestone
1. Milestone descriptions MUST include:
   1. Target version number
   1. Summary of included changes
   1. Link to the group-level milestone (if applicable)
1. Milestones MUST NOT be closed until all assigned issues are resolved and the release process (Release Process Section 1) is complete

### 1.2.4 Iterations

1. Project-level iterations inherit the group-level iteration cadence (Section 1.1.6)
1. At iteration start, the team MUST:
   1. Select issues from the backlog for the iteration based on priority and capacity
   1. Assign issues to team members
   1. Confirm the iteration goal
1. At iteration end, the team MUST:
   1. Review incomplete work and carry forward or deprioritise
   1. Update issue statuses and board
   1. Participate in the group-level retrospective

### 1.2.5 Wiki

1. The project-level wiki MUST contain project-specific documentation that does not belong in the repository's source code
1. Permitted content:
   1. Technical proposals as defined in Planning Section 3
   1. Architecture diagrams and design notes
   1. Runbooks for project-specific operational procedures
   1. Onboarding guides specific to the project
1. Content that belongs in the repository (API docs, README, CHANGELOG, ADRs) MUST NOT be duplicated in the wiki
1. Wiki pages MUST follow the documentation standards defined in Architectural Principles Section 11

### 1.2.6 Requirements

1. Project-level requirements MUST follow all rules defined in Planning Section 2
1. Requirements MUST be entered into GitLab's requirements feature with:
   1. The requirement ID as defined in Planning Section 2.3
   1. The requirement text following the grammar defined in Planning Section 2.2
   <!-- 1. Classification (functional or non-functional) as defined in Planning Section 2.6 -->
1. Requirements MUST be linked to their implementing issues and verifying test issues
1. Requirement coverage MUST be reviewed at every milestone — requirements without linked implementations MUST be flagged
1. Changes to approved requirements MUST follow the change management process defined in Planning Section 2.8
