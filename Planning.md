# 1. Scope Writing

## 1.1. Required Sections

1. Every scope document MUST contain the following sections:
   1. Problem statement: a clear, unambiguous description of the problem to be solved
   1. Success criteria: measurable outcomes that define when the problem is solved
   1. In-scope boundaries: explicit list of what is included
   1. Out-of-scope boundaries: explicit list of what is excluded
   1. Assumptions: all assumptions MUST be documented — implicit assumptions are PROHIBITED
   1. Constraints: technical, resource, time, and regulatory constraints
   1. Stakeholder identification: all stakeholders MUST be named with their role and interest
   1. Impact assessment: systems, teams, and processes affected by the work
   1. Dependency identification: all dependencies (technical, team, external) MUST be listed
   1. Risk identification: all known risks MUST be documented with likelihood and impact

## 1.2. Estimation
> Review change to complexity based estimation
1. Timeline estimation MUST use a three-point estimate: optimistic, expected, pessimistic
1. Resource estimation MUST specify required skills, not just headcount

## 1.3. Approval

1. Scope documents MUST be approved before work begins
1. Changes to an approved scope MUST go through the same approval process as the original
1. Scope changes after approval MUST be documented with rationale

# 2. Requirement Writing

## 2.1. Rules

1. One requirement per statement — conjunctions (`and`, `or`) are PROHIBITED
1. Active voice with explicit subject (`The Login_Module shall...`)
1. Measurable criteria (`within 2 seconds`, not `quickly`)
1. The word `shall` for mandatory requirements
1. The word `should` for recommended requirements
1. The word `may` for optional requirements
1. Ambiguous terms are PROHIBITED: `fast`, `efficient`, `user-friendly`, `robust`, `flexible`, `scalable`, `etc.`, `and so on`, `appropriate`

## 2.2. Requirement Grammar

### 2.2.1. Baseline Grammar

1. `The <subject> shall <action>.`

### 2.2.2. EARS Modifiers

1. State-Dependent:
   1. `While <state>, the <subject> shall <action>.`
1. Event-Triggered:
   1. `When <event>, the <subject> shall <action>.`
1. Optional Feature:
   1. `Where <feature>, the <subject> shall <action>.`
1. Error Handling:
   1. `If <condition>, then the <subject> shall <action>.`
1. Complex (State + Event):
   1. `While <state>, when <event>, the <subject> shall <action>.`

## 2.3. Requirement Identification

1. Format: `PRJ-TYP-###`
   1. `PRJ`: project or system code (e.g., `AUTH`, `PIPE`, `PLAT`)
   1. `TYP`: requirement type code e.g:
      1. `SEC` — security
      1. `INT` — interface
      1. `CON` — constraint
   1. `###`: sequential number within the project-type combination
1. Example: `AUTH-SEC-003` = Auth project, security requirement #3
1. Requirement IDs MUST be unique and MUST NOT be reused after deletion

## 2.4. Requirement Attributes
> Review since no attributes in gitlab
<!-- 
1. Every requirement MUST have the following attributes:
   1. ID: as defined in 3.2.3
   1. Type: `FUN`, `NFR`, `SEC`, `INT`, or `CON`
   1. Priority: MoSCoW classification
      1. `Must` — non-negotiable, delivery fails without it
      1. `Should` — important, expected in delivery unless justification provided
      1. `Could` — desirable, included if time and resources permit
      1. `Won't` — explicitly excluded from this delivery (documented for future reference)
   1. Complexity: score 1–5
      1. 1 — trivial, no unknowns
      1. 2 — straightforward, minor unknowns
      1. 3 — moderate, some design decisions required
      1. 4 — complex, significant design and coordination required
      1. 5 — highly complex, research or prototyping required
   1. Risk: `High`, `Medium`, `Low`
   1. Dependencies: list of requirement IDs this requirement depends on
   1. Project: project code
   1. Source: origin of the requirement (stakeholder name, regulation reference, technical constraint)
   1. Status: `Draft`, `Approved`, `Implemented`, `Verified`, `Deferred`, `Deleted` -->

## 2.5. Acceptance Criteria
> Review
<!-- 
1. Every requirement MUST have acceptance criteria
1. Acceptance criteria MUST use Given/When/Then format:
   1. `Given <precondition>`
   1. `When <action>`
   1. `Then <expected result>`
1. Each acceptance criterion MUST be independently testable
1. Acceptance criteria MUST NOT introduce new requirements — they verify existing ones -->

## 2.6. Classification
> Review

<!-- 1. Every requirement MUST be classified as functional or non-functional
1. Non-functional requirements MUST specify which quality attribute they address (performance, security, availability, maintainability, etc.) -->

## 2.7. Traceability

1. Every requirement MUST be traceable to:
   1. Its source (scope document, stakeholder, regulation)
   1. Its implementation (code, PR, commit)
   1. Its verification (test case, test result)
1. Traceability MUST be maintained throughout the requirement lifecycle
1. Orphaned requirements (no source) and orphaned implementations (no requirement) MUST be flagged

## 2.8. Review and Change Management

1. Requirements MUST be reviewed before approval
1. The review MUST check for:
   1. Ambiguity (no ambiguous terms as defined in 3.2.1)
   1. Completeness (all attributes populated)
   1. Consistency (no contradictions with other requirements)
   1. Testability (acceptance criteria are independently verifiable) 
   > Clear up
   1. Feasibility (requirement is technically achievable within constraints)
1. Changes to approved requirements MUST go through a formal change process:
   1. Change request with rationale
   1. Impact assessment (which other requirements, designs, and implementations are affected)
   1. Re-approval by the original approver or delegate
1. All requirement changes MUST be versioned — previous versions MUST be retained

# 3. Technical Proposals

## 3.1. Required Sections

1. Every technical proposal MUST contain the following sections:
   1. Problem definition: clear statement of the problem, linked to scope and/or requirements
   1. Proposed solution: detailed technical approach
   1. Alternative solutions: minimum 2 alternatives MUST be presented
   1. Trade-off analysis: explicit comparison of proposed solution against alternatives across defined criteria (performance, cost, complexity, correctness, maintainability)
   1. Cost estimation: development effort, infrastructure cost, ongoing maintenance cost
   1. Timeline estimation: using three-point estimates (optimistic, expected, pessimistic)
   1. Risk assessment: risks of the proposed solution with likelihood and impact
   1. Security review: security implications of the proposed solution
   1. Performance impact assessment: expected performance characteristics
   1. Rollback strategy: how to reverse the change if it fails

## 3.2. Dependency Evaluation

1. If the proposed solution introduces an external dependency, the proposal MUST include the dependency approval criteria from Section 1.8.1.2
1. The in-house → open source → proprietary hierarchy MUST be addressed in the alternatives section

## 3.3. Proof of Concept

1. Proposals for new technologies, unfamiliar patterns, or complexity score ≥ 4 MUST include a proof of concept
1. The proof of concept MUST demonstrate the critical technical risk, not the full solution

## 3.4. Review and Approval

1. Technical proposals MUST be reviewed by at least one person who is not the author
1. Proposals MUST be approved before implementation begins
1. Feedback MUST be incorporated within 5 business days of receipt or the proposal is considered stale

## 3.5. Expiration

1. Unactioned proposals expire after 90 days
1. Expired proposals MUST be re-submitted with updated context if they are to be pursued
1. Expiration resets the approval process

# 4. Deliverable Writing

## 4.1. Required Sections

1. Every deliverable document MUST contain:
   1. Acceptance criteria: what constitutes a complete deliverable, linked to requirements
   1. Definition of done linkage: explicit reference to the project's definition of done (see Section 10)
   1. Handoff documentation: what the recipient needs to know to use, maintain, or extend the deliverable
   1. Demo requirements: what must be demonstrated and to whom before the deliverable is accepted
   1. Documentation requirements: what documentation must accompany the deliverable
   1. Support transition: who is responsible for the deliverable after handoff and how support requests are handled

## 4.2. Sign-Off

1. Deliverables MUST be signed off by the stakeholder identified in the scope document
1. Sign-off MUST confirm that acceptance criteria are met
1. Sign-off MUST be recorded (not verbal)

## 4.3. Training

1. If the deliverable introduces new tools, processes, or systems, a training plan MUST be included
1. Training MUST be completed before the deliverable is considered fully handed off.
