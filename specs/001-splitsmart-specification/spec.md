# Feature Specification: SplitSmart Expense Sharing

**Feature Branch**: Existing branch `lab7-architecture-design`; no branch hook configured

**Created**: 2026-11-05

**Status**: Draft — source clarifications recorded; architecture planning may proceed with unresolved policies documented

**Input**: Create a software specification from the existing User Requirements document, preserving intended features, identifying assumptions and unresolved questions, and producing no application code.

**Primary source**: [SplitSmart Customer Requirements](../../UserRequirement.md). This is the only completed Lab 6 artifact. No completed technical specification, architecture, or detailed design is assumed. This document describes required behavior, not an implementation design.

## Clarifications

### Session 2026-11-05

- Q: What approval workflow and approver permissions are required (Q1)? → A: Expenses must be reviewed and approved before affecting balances. The workflow may use a single review process or multiple approval levels depending on group needs. Approver roles and self-approval permissions are unspecified and remain open questions.
- Q: What group management and access policies are required (Q2)? → A: Users must be able to create and manage groups and access the expense and balance information they are authorized to view. Membership management roles, removal restrictions, and former-member access are unspecified and remain open questions.

These answers preserve the source's flexibility; they do not select a single workflow, mandate configurable approval levels, establish roles, or settle access rules. Architecture planning proceeds where independent of those choices; affected implementation and final acceptance remain conditional.

## User Scenarios & Testing *(mandatory)*

Priorities express relative user value, not permission to omit lower-priority requirements. Each journey can be tested with prepared accounts, groups, and financial records without exercising every other journey.

### User Story 1 - Access Personal Information (Priority: P1)

As a person sharing expenses, I want my own account so I can access my expenses and balances.

**Why this priority**: Personal identification is necessary to attribute expenses and obligations correctly.

**Independent Test**: Create two accounts with separate prepared records and verify authenticated access and isolation.

**Acceptance Scenarios**:

1. **Given** a person without an account, **When** they complete valid account registration and log in, **Then** they can access their own expense and balance information.
2. **Given** invalid login information or no authenticated account, **When** personal information is requested, **Then** access is denied without exposing that information.
3. **Given** two users with unrelated expenses, **When** either views their personal records, **Then** the other's unrelated records are not disclosed.

### User Story 2 - Organize Shared Groups (Priority: P1)

As a user, I want to create and manage a group for roommates, a trip, or another shared activity so expenses and balances are associated with the right people.

**Why this priority**: Groups provide the documented context for sharing and reviewing expenses.

**Independent Test**: Use prepared accounts to create a group and exercise membership management under the agreed permission policy.

**Acceptance Scenarios**:

1. **Given** an authenticated user, **When** they create a named group with selected users, **Then** the group and its membership can be viewed by authorized users.
2. **Given** a group and a user authorized by the agreed policy, **When** membership is changed, **Then** subsequent participant selection reflects the change and existing financial records remain attributable to their original participants.
3. **Given** a user without the required permission, **When** they request a group management action, **Then** the action is denied and the group remains unchanged.

### User Story 3 - Submit and Review Shared Expenses (Priority: P1)

As a user, I want to describe an expense, specify its participants and split, and have it reviewed before it changes balances.

**Why this priority**: Correct expense allocation and approval are the central financial workflow.

**Independent Test**: Use a prepared group and its agreed approval policy to submit, review, approve, and decline sample expenses.

**Acceptance Scenarios**:

1. **Given** three participants and an expense of 90 paid by one of them, **When** an equal split is submitted, **Then** each share is 30, the expense awaits approval, and balances remain unchanged.
2. **Given** an expense of 100, **When** shares of 50%, 30%, and 20% are submitted, **Then** the recorded shares are 50, 30, and 20 respectively.
3. **Given** a new group expense, **When** submission succeeds, **Then** other group members receive a notification identifying the new expense without implying approval.
4. **Given** an expense awaiting review, **When** the agreed approval policy is satisfied, **Then** it becomes approved and its balance effects occur exactly once; if declined, it has no balance effect.
5. **Given** otherwise valid expense details, **When** submitted with or without a receipt image, **Then** submission succeeds and an attached image can be viewed by authorized viewers.
6. **Given** missing amount, date, description, participants, or split, or percentages totaling other than 100%, **When** submission is attempted, **Then** the user is told what to correct and balances do not change.

### User Story 4 - View Balances and Record Settlements (Priority: P1)

As a user, I want to see what I owe each other user and record payments already made so balances reflect outstanding obligations.

**Why this priority**: Users need accurate amounts to settle shared costs.

**Independent Test**: Start with approved expenses between prepared users, record a payment, and compare before-and-after balances.

**Acceptance Scenarios**:

1. **Given** Alice paid 90 shared equally with Bob and Cara, **When** the expense is approved, **Then** Bob owes Alice 30, Cara owes Alice 30, and Alice owes neither for this expense.
2. **Given** Bob owes Alice 30, **When** a payment of 10 from Bob to Alice is recorded, **Then** the outstanding balance is 20 and the payment appears in their payment information.
3. **Given** an outstanding balance of 20, **When** a payment of 20 is recorded, **Then** the outstanding balance becomes zero.
4. **Given** a recorded transaction, **When** its processing is retried, **Then** it does not affect balances a second time.

### User Story 5 - Review Summaries and Reports (Priority: P2)

As a user, I want reports and summaries of expenses, balances, and payments so I can understand shared spending and settlement history.

**Why this priority**: Reporting makes accumulated financial activity understandable.

**Independent Test**: Load known approved expenses, pending expenses, and payments; compare authorized reports with those records.

**Acceptance Scenarios**:

1. **Given** known financial records, **When** a user requests their reports and summaries, **Then** expenses, balances, and payments agree with the records they are authorized to view; unapproved expenses are distinguishable and excluded from balances.
2. **Given** no relevant activity, **When** a report is viewed, **Then** it explicitly indicates no activity and shows no invented financial entries.

### Edge Cases

- Zero or negative expense/payment amounts, missing required fields, duplicate participants, or invalid percentages: reject the entry with an actionable explanation and preserve balances.
- An equal or percentage split produces fractional currency units: allocated amounts must still sum exactly to the expense; the allocation rule remains to be agreed (OQ-002).
- Multiple reviewers or repeated approval attempts: balances must reflect one approved expense, irrespective of repeated actions.
- A member leaves a group with outstanding obligations: preserve historical attribution; removal permissions and treatment of unsettled membership remain OQ-009.
- Receipt cannot be accepted or a save fails: explain the failure and do not report a successful submission or apply partial balance changes.
- A new user, empty group, or user with no debts: show the empty state or zero balance explicitly.
- Opposing obligations between the same pair: show the net outstanding direction and amount, without counting the same obligation twice (assumption A3).
- Overpayments, conflicting changes, and correction of approved records: no behavior is prescribed beyond integrity requirements until the policies in OQ-003 are decided.

## Requirements *(mandatory)*

The source references below name the original document's bullet headings. Derived safeguards are explicitly identified; they are proposed interpretations supporting the documented features rather than additional customer statements.

### Functional Requirements

- **FR-001 — Accounts**: Each person MUST be able to obtain an individual account, log in, and access their own expense and balance information. Source: User accounts. Acceptance: Story 1.1.
- **FR-002 — Access protection**: Personal financial information MUST require authentication; unrelated users' records MUST not be exposed. Derived from User accounts. Acceptance: Story 1.2–1.3 and NFR-001.
- **FR-003 — Group creation**: Users MUST be able to create named groups and associate other registered users with them. Source: Group management; registration and naming are assumptions A1/A2. Acceptance: Story 2.1.
- **FR-004 — Group management and visibility**: Users MUST be able to create and manage groups and access the expense and balance information they are authorized to view. Membership management roles, removal restrictions, and former-member access remain unspecified; no such policy is established here. Source: Group management and User accounts; Q2 clarification. Acceptance: Story 2.2–2.3, with the permission matrix conditional on OQ-009.
- **FR-005 — Expense details**: Users MUST be able to submit an expense with an amount, date, description, split method, selected involved users, and an optional receipt image. Source: Expense creation. Acceptance: Story 3.1–3.2, 3.5–3.6.
- **FR-006 — Attribution**: Each expense MUST identify who paid and the share assigned to each involved user, including expenses between individuals and expenses within groups. Derived from Expense creation and Balance tracking; single payer is assumption A2. Acceptance: Stories 3.1 and 4.1.
- **FR-007 — Splits**: The system MUST support equal splits and participant percentage splits; percentages MUST total 100%, and allocated shares MUST sum to the expense amount. Source: Expense creation; exact-total validation is derived. Acceptance: Story 3.1–3.2, 3.6 and fractional-unit edge case.
- **FR-008 — Input validation**: Invalid financial entries MUST be rejected with an explanation identifying what needs correction, without adjusting balances. Derived safeguard. Acceptance: Story 3.6 and invalid-entry edge case.
- **FR-009 — Receipts**: Receipt images MUST be optional and associated with their expense when provided; access MUST follow the expense visibility policy. Source: Expense creation. Acceptance: Story 3.5 and FR-004 access tests.
- **FR-010 — Notifications**: When a user successfully creates a group expense, the system MUST notify the other current group members and identify the expense. Source: Notification system. Acceptance: Story 3.3. Delivery channel and timing remain OQ-004.
- **FR-011 — Approval gate**: Expenses MUST be reviewable before becoming approved financial records; pending or declined expenses MUST NOT affect balances. Submission must retain the information needed for review. Source: Approval workflow. Acceptance: Story 3.1, 3.4.
- **FR-012 — Approval policy**: Expenses MUST be reviewed and approved before affecting balances. The workflow may use a single review process or multiple approval levels depending on group needs. Approver roles and self-approval permissions remain unspecified; neither workflow is selected or mandated as the sole supported mode here. Source: Approval workflow; Q1 clarification. Acceptance: Story 3.4 and repeated-review tests; reviewer eligibility and workflow-specific tests remain conditional on OQ-008.
- **FR-013 — Balances**: Users MUST be able to see the outstanding amount owed to or by each other user with whom they have recorded obligations, accounting for all approved expenses and recorded payments. Source: Balance tracking and Payment tracking. Acceptance: Story 4.1–4.3.
- **FR-014 — Payment recording**: Users MUST be able to record an already-made payment with payer, recipient, positive amount, and payment date, and see the resulting balance adjustment. Source: Payment tracking; required fields are derived. Acceptance: Story 4.2–4.3. Recording permissions and exceptional payments remain OQ-003.
- **FR-015 — Reports**: Users MUST be able to view reports and summaries of authorized expenses, balances, and payments; financial totals MUST reconcile with the underlying approved expenses and recorded payments. Source: Reporting. Acceptance: Story 5.1–5.2.
- **FR-016 — Transaction integrity**: Approval and payment recording MUST apply each financial effect exactly once; failed actions MUST NOT leave partial balance adjustments. Derived safeguard. Acceptance: Story 4.4 and repeated-review/save-failure edge cases.
- **FR-017 — Record continuity**: Saved expenses, approval decisions, payments, and balances MUST remain available after logout and subsequent login, retaining their participant attribution after membership changes. Derived from tracking/reporting. Acceptance: save known records, log out/in, change membership under the agreed policy, and verify unchanged historical attribution and totals.

### Nonfunctional Requirements

- **NFR-001 — Confidentiality**: For each personal or group view, receipt, report, and modifying action, a user outside the agreed access policy MUST be denied access without disclosure or modification. Verify permitted and denied actions using the eventual policy, including current and former members; former-member access is not presumed permitted or denied. Unauthenticated access to personal financial information is denied. Derived from personal accounts; group boundaries remain OQ-009.
- **NFR-002 — Financial correctness**: All accepted allocation and settlement test cases MUST reconcile exactly at the selected currency precision; approval, retry, and failed-action tests MUST produce no missing or duplicate balance effects. Derived from accurate balance tracking. Currency and rounding decisions remain OQ-002.
- **NFR-003 — Platform support**: SplitSmart MUST provide a primary web-based experience and applications runnable on Android and iOS. The documented account, group, expense, review, notification, balance, payment, and reporting journeys MUST pass on each supported platform. Source: OS supported. Specific browser and OS versions remain OQ-005.
- **NFR-004 — Understandable feedback**: Validation and failed-save scenarios MUST identify the affected action and correction or retry needed; pending, approved, and declined expense states and zero-activity states MUST be distinguishable in text. Derived usability criterion, not a customer-specified usability benchmark.
- **NFR-005 — Consistency and persistence**: A successfully saved financial change MUST remain reflected after reopening an account and MUST produce matching financial totals when the same account and records are viewed on supported platforms after retrieval of the latest saved records. Derived from tracking and platform support. Offline operation is not assumed.

No customer-specified latency, capacity, availability, retention, or accessibility thresholds exist in the source. These remain explicit questions below rather than fabricated commitments.

### Key Entities *(include if feature involves data)*

- **User account**: An individual identity associated with personal expenses, balances, and payments.
- **Group and membership**: A named shared activity and its participating accounts; management and visibility permissions remain OQ-009.
- **Expense**: Amount, date, description, payer, participants, split method, participant shares, optional group and receipt, and review state.
- **Receipt**: An optional image associated with an expense and governed by its visibility rules.
- **Approval decision and policy**: The review outcome and applicable reviewer/level rules; workflow selection, reviewer roles, and self-approval remain OQ-008.
- **Balance**: An outstanding obligation between two users, derived from approved expenses and recorded payments.
- **Payment record**: An already-made settlement identifying payer, recipient, amount, and date.
- **Notification**: Notice to another group member of a newly submitted group expense.
- **Report or summary**: A presentation of authorized expenses, balances, and payments; not a separate source of financial truth.

## Success Criteria *(mandatory)*

### Measurable Outcomes

These are specification acceptance targets derived from the documented capabilities, not measured results or customer-provided performance promises.

- **SC-001**: All nine source feature headings are traced to requirements: accounts FR-001–002; groups FR-003–004; expenses FR-005–009; notifications FR-010; approval FR-011–012; balances FR-013; payments FR-014/016; reporting FR-015; platforms NFR-003.
- **SC-002**: In the 90 equal-split and 100 percentage-split scenarios, 100% of participant shares and resulting balances match the stated amounts, and pending/declined expenses contribute zero to balances.
- **SC-003**: Recording a payment of 10 against a debt of 30 yields 20 outstanding; recording the remaining 20 yields zero; replaying either saved transaction yields no additional adjustment.
- **SC-004**: Every successful group expense submission in acceptance tests creates a notification for each other current group member. Delivery timing is assessed only after OQ-004 is resolved.
- **SC-005**: For each supported platform, every primary journey passes its acceptance scenarios, and the same saved financial records produce identical totals across all three platforms.
- **SC-006**: All negative access tests deny disclosure/modification, and all tested expense and payment report totals reconcile with approved source records.
- **SC-007**: In a walkthrough on each platform, a representative user completes each primary journey and correctly identifies their outstanding balance and an expense's approval state without facilitator correction. This is a proposed qualitative acceptance method, not a statistically validated usability target.

## Assumptions

- **A1 — Accounts and groups**: People selected for expenses and groups have registered accounts. Groups have names so users can distinguish them. These are working interpretations, not explicit source details.
- **A2 — Expense attribution**: Each expense initially records one payer, who may also be a participant. Individuals may share an expense without creating a permanent group. Multiple payers on one expense are not committed by the source; confirm if required.
- **A3 — Settlement scope**: Payments record settlements made outside SplitSmart; moving money is not specified. Pairwise balances show net obligations. No cross-user debt simplification or currency conversion is presumed.
- **A4 — Optional reporting scope**: Basic reports and summaries are mandatory. The source says custom date ranges and other parameters are “possibly” included, so custom reports remain a candidate enhancement pending OQ-001, rather than being silently removed or promoted to mandatory scope.
- **A5 — Connectivity**: Acceptance journeys assume connectivity and retrieval of current saved records. Offline editing or synchronization guarantees are not stated.
- **A6 — Derived safeguards**: A declined state is a proposed representation of unsuccessful review, not a customer-defined rejection or resubmission workflow. Positive monetary amounts, validation, confidentiality, exact totals, persistence, and duplicate-effect prevention operationalize the documented account and financial tracking needs. They are derived criteria, not claims that the customer supplied detailed policies.
- **A7 — Governance and dependencies**: The repository constitution contains only unfilled template placeholders and supplies no adopted project principles. No existing authentication, payment, notification, or reporting service is assumed. Architecture planning may proceed without settling the policies below; affected implementation and acceptance tests depend on their resolution.

### Unresolved Questions and Scope Decisions

Q1 and Q2 are answered in the clarification record. They confirm source behavior while intentionally leaving policy decisions open. The questions below are not accepted defaults or permission to implement arbitrary policies.

- **OQ-001 — Custom reports**: Are specified date ranges mandatory? Which other parameters or export formats, if any, are required?
- **OQ-002 — Monetary policy**: Which currency or currencies and precisions are supported? How are residual fractional units allocated while preserving exact totals? Currency conversion is not presently specified.
- **OQ-003 — Financial changes**: Who may record payments, and must the recipient confirm them? Are overpayments supported? How are approved expenses or payments corrected, deleted, disputed, or reversed? How should concurrent conflicting changes be resolved?
- **OQ-004 — Notifications**: Which delivery channel, delivery deadline, and failed-delivery behavior are expected? No email, push, or other channel is imposed here.
- **OQ-005 — Supported environments and quality targets**: Which browsers and Android/iOS versions are supported? What measurable response-time, load, availability, accessibility, backup/recovery, and retention targets are needed? These are planning inputs, not accepted numeric commitments.
- **OQ-006 — Account lifecycle and attachments**: What registration identity, recovery, closure, and data deletion policies apply? What receipt image formats and size limits are acceptable?
- **OQ-007 — Review lifecycle**: Beyond approval/decline, are resubmission, reviewer delegation, reminders, or notifications of decisions required? How is a policy established or changed for existing expenses?

- **OQ-008 — Approval policy (Q1 follow-up)**: Which group needs call for single review or multiple levels? Who may approve group and individual expenses? Is self-approval permitted? What completes each level? These remain open; no administrator, owner, participant-voting, or automatic approval policy is assumed.
- **OQ-009 — Group authorization (Q2 follow-up)**: Who may manage membership? Are there restrictions on removal, including outstanding balances? Which expense, receipt, and balance information may current and former members access? Historical attribution does not imply continued viewing rights.

Custom reporting, payment execution, multiple expense payers, currency conversion, offline operation, and advanced review features above are unresolved or unspecified capabilities, not implementation commitments. Web, Android, and iOS support remain within the documented scope.
