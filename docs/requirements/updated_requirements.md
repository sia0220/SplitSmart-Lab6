# SplitSmart updated requirements — CIS 285 Lab 7

Date: November 5, 2026. Status: final documentation baseline; policy-dependent implementation and acceptance remain conditional.

## Authority and document map

The [original User Requirements](../../UserRequirement.md), specifically its nine feature bullets, are the sole completed Lab 6 artifact and primary authority for customer scope. This document refines that scope without replacing the original. The [supporting Lab 7 specification](../../specs/001-splitsmart-specification/spec.md#requirements-mandatory) supplies stable FR/NFR identifiers and clarifications, not additional customer commitments. The [architecture decision](../architecture/architecture_decision.md#selected-architecture) selects B; the [consolidated instructions](../architecture/consolidated_instructions.md#scope-and-architectural-guardrails) govern technical refinements.

These six requested files are the authoritative final Lab 7 set: this requirements document; [final architecture](../architecture/final_architecture.md); [detailed design, including database and data flows](../design/detailed_design.md); [class diagram](../design/class_diagram.md); [activity diagrams](../design/activity_diagram.md); and [deployment design](../design/deployment_design.md). Suggested alternate filenames in the consolidated instructions are mapped to these files rather than creating duplicate artifacts. Earlier specification/plan/data-model documents remain source records. In particular, their proposed financial-effect/posting model is superseded by B's derived balances.

Classification used throughout: **S** = source requirement; **G** = derived safeguard; **E** = selected engineering decision; **A** = provisional assumption; **OQ** = unresolved customer decision. A row may mix source behavior and a clearly identified safeguard. Engineering proposals are recorded separately from accepted decisions.

## Purpose, boundary, actors, and terms

SplitSmart helps individuals and groups record shared expenses, review them, determine pairwise obligations, record payments already made, and inspect summaries. It provides a primary web experience and actual Android and iOS applications. External settlement occurs outside SplitSmart; it does not execute transfers (A3).

An account holder can act as submitter, payer, participant, reviewer, payment recorder, or report reader only when authorized for the action/resource. These are contextual roles, not permission grants or an administrator hierarchy. An unauthenticated visitor can use the chosen account-entry flow but cannot read personal financial information. Identity and lifecycle details remain OQ-006.

A **pending** expense is a stored review submission, not an approved financial entry. Partial review leaves it pending. An **approved** expense is eligible for financial queries. **Declined** is a provisional representation of unsuccessful review, with zero financial contribution. A **share** is an involved person's exact allocation; the payer may also have a self-share. A **balance** is a derived pairwise net obligation, not a stored entity. A **notice** is a persisted new-expense notification record; persistence does not prove delivery.

## Functional requirements and acceptance scenarios

The heading references below refer to [the original feature bullets](../../UserRequirement.md). For all authenticated operations, preconditions include a valid session and action/resource authorization under the eventual policy. Failure must return an understandable error without partial changes or disclosure. Acceptance scenarios below are proposed future evidence, not executed tests.

| ID / source / classification | Required behavior and rationale | Preconditions and observable acceptance scenario | Error behavior / dependency |
|---|---|---|---|
| FR-001 — User accounts — S | Each person can obtain an individual account, log in, and access their own expense/balance information; identity permits personal attribution. | AC-001: valid account-entry flow yields the person's own saved records. | Invalid credentials deny access; identity flow OQ-006. |
| FR-002 — User accounts — G | Require authentication and protect unrelated personal records. | AC-002: anonymous and unrelated-account requests disclose no protected data. | Generic denial without confirming protected record existence. |
| FR-003 — Group management — S/A1 | Create groups and associate people; group names and registered selections are provisional identifying conventions. | AC-003: permitted creation returns a named group and recorded memberships. | Invalid references fail without partial group creation; OQ-009. |
| FR-004 — Group management — S/G | Manage groups and view authorized group records while preserving history. | AC-004: permitted membership change affects future selection, preserves original attribution; forbidden change does nothing. | Management/removal/current/former-member rules OQ-009. |
| FR-005 — Expense creation — S | Submit amount, date, description, involved users, equal/percentage split, and optional receipt for review. | AC-005: complete input produces a pending submission and shares, with zero balance impact. | Missing fields fail with field-level feedback. |
| FR-006 — Expense creation / Balance tracking — G/A2 | Identify one payer and each participant's share; support optional group as an individual-sharing interpretation. | AC-006: group and groupless records retain payer, submitter, and participants distinctly. | Unknown/ineligible users rejected; one payer/groupless interpretation needs confirmation. |
| FR-007 — Expense creation — S/G | Support both equal and percentage splits; exact shares sum to amount; percentages total exactly 100. | AC-007: 90 equally split three ways gives 30 each; 100 at 50/30/20 gives 50/30/20. | Invalid percentages rejected; fractional expectations await OQ-002. |
| FR-008 — Expense creation — G | Reject invalid amounts, duplicate participants, and inconsistent allocations so balances stay accurate. | AC-008: zero/negative amount or invalid split produces useful correction feedback and no saved submission. | No notices or financial contribution on failure; exact units OQ-002. |
| FR-009 — Expense creation — S/G | Receipt image is optional; any attached receipt follows expense visibility. | AC-009: valid submissions with/without image succeed; forbidden image read is denied. | Image failure must not be reported as successful submission; limits OQ-006, visibility OQ-009. |
| FR-010 — Notification system — S/A | Notify other group members of a new expense. Proposed baseline captures current recipients at submission and uses in-app retrieval. | AC-010: commit creates one notice per other current member; subsequent authorized retrieval identifies new expense without implying approval. | Rollback creates none; delivery/channel/timing OQ-004, disclosure OQ-009. |
| FR-011 — Approval workflow — S/G | Retain review records before financial inclusion; pending, partially reviewed, and declined expenses contribute zero. | AC-011: all three conditions leave balances unchanged. | No automatic approval, including individual expenses. |
| FR-012 — Approval workflow — S | Review completion follows agreed single-review or multilevel policy as group needs require. | AC-012: incomplete fixture stays pending; completed fixture becomes approved once; decline fixture remains nonfinancial. | Reviewer eligibility, self-approval, individual review and level rules OQ-008; lifecycle OQ-007. |
| FR-013 — Balance tracking — S/A3 | Show amount owed between distinct users from approved expenses and recorded settlements, using pairwise netting. | AC-013: Alice pays 90 equally with Bob/Cara; approved result is Bob→Alice 30 and Cara→Alice 30. Opposing 12 obligation yields Bob→Alice 18. | No payer self-debt; authorized scope/complete-debt meaning OQ-009. |
| FR-014 — Payment tracking — S/G/A3 | Record an already-made settlement with payer, recipient, positive amount and date; reflect it in balances. | AC-014: Bob pays Alice 10 against 30; remaining 20; another 20 leaves zero. | No money transfer; authority, confirmation and overpayment OQ-003. |
| FR-015 — Reporting — S/G | Provide expense, balance, and payment summaries with reconciled authorized financial totals. | AC-015: known approved records/payments match balance query; pending/declined lists are labeled and excluded from financial totals; empty scope shows no activity/zero. | Custom filters/exports OQ-001; group settlement gap and privacy OQ-009/OQ-010. |
| FR-016 — Tracking/approval — G | Transactions, retries, and concurrent operations preserve one effect per accepted command and no partial save. | AC-016: lost-response replay returns original result; changed payload with same key conflicts; concurrent final reviews include each share once. | A new review command is not automatically a retry; policy eligibility still required. |
| FR-017 — Tracking/reporting — G | Durable expense, review and payment history retains original attribution across login and permitted membership changes. | AC-017: reopen records and compare totals/attribution after logout/login and membership change. | Historical identity does not grant former-member access; corrections/deletion OQ-003/OQ-006/OQ-009. |

Required report categories are expense summaries (records and state, approved totals separately), pairwise balance summaries (direction, amount, monetary unit and scope), and payment summaries (parties, dates, amounts). These column layouts are design proposals. Personal views are required; authorized group views support group tracking. Group expense totals are possible, but group-settled balances remain conditional because payments have no group attribution. No global all-users report privilege is granted. Reconciliation compares identical scope, unit, and snapshot. Custom date ranges and other parameters remain optional OQ-001.

## Quality requirements

| ID | Criterion and proposed evidence | Classification / dependency |
|---|---|---|
| NFR-001 | Deny unauthorized mutations and disclosure across individual/group records, receipts, notices, reports and aggregates; exercise allowed/denied permission matrix cases. | G; OQ-009. |
| NFR-002 | Exact financial arithmetic and sum invariants; no duplicate/partial effects under retries and failures. Verify AC-007/013/014/016 plus fractional cases after policy selection. | G; OQ-002. |
| NFR-003 | Primary web plus runnable Android and iOS apps support every primary journey. AC-018: repeat AC-001–017 separately on all three platforms. | S, OS supported; supported versions OQ-005. |
| NFR-004 | Explain invalid inputs, failed saves, pending/approved/declined states and empty results in text. | G; assess walkthrough feedback. |
| NFR-005 | Persist successful saves and return matching totals after current-data retrieval on each platform. AC-019: reopen and compare same scope/unit/source records. | G/A5. |

Retry safety includes submissions, review decisions and payments. No numeric response time, load, capacity, availability, accessibility, retention, or recovery target is supplied by the customer; OQ-005 must establish them. Offline synchronization is not committed.

## Provisional permission matrix

Every protected action requires authentication and a positive server policy result. Unknown grants remain unavailable for production until resolved; this is a safe boundary, not a selected product permission policy. No creator-as-administrator, membership-as-universal-access, self-approval, or former-member entitlement is inferred.

| Action/resource | Known constraint | Unresolved permission / acceptance dependency |
|---|---|---|
| Own personal expense/balance view | Account holders access their own information; unrelated records protected. | Cross-group/history aggregation completeness OQ-009; AC-001/002/013. |
| Create/manage group, add/remove membership | Capability required; preserve historical identities. | Who may perform each mutation and debt-related removal restrictions OQ-009; AC-003/004. |
| Submit group or individual expense | Authenticated submitter; validate parties and context. | Allowed groups/participants/payers OQ-009; individual interpretation A2; AC-005/006. |
| Review group or individual expense | Eligibility and completion policy required. | Reviewers, self-approval, stages OQ-008; AC-011/012. |
| Record payment | Authenticate recorder; validate distinct parties. | Recorder authority, confirmation and exceptional payments OQ-003; AC-014. |
| Read expense or receipt | Receipt access follows expense policy. | Current/former-member and individual visibility OQ-009; AC-009. |
| Read notice | Only intended recipient can be considered, then recheck current visibility. | Content and access after membership changes OQ-004/009; AC-010. |
| Read report or aggregate balance | Scope authorized before aggregation; no unrestricted global access. | Privacy versus complete debt, group settlements OQ-009/010; AC-013/015. |

## Assumption and decision register

This is the single final register referenced by all other deliverables. A1–A7 and OQ-001–009 retain their meanings from [specification assumptions and questions](../../specs/001-splitsmart-specification/spec.md#assumptions). All assumptions below are provisional, not customer approvals.

| ID / status | Interpretation and rationale/source | Affected artifacts and consequences |
|---|---|---|
| A1 — provisional | Selected people have registered accounts; groups have identifying names. Source spec A1. | Requirements; detailed/class design; AC-003/006 need confirmation. |
| A2 — provisional | One payer, optionally participating; groupless sharing supported. Source spec A2/B. | All design/flows; AC-006/013 and schema change if multiple payers required. |
| A3 — provisional | Payments record outside settlements; obligations net pairwise; no conversion or cross-user simplification. Source spec A3/B. | Detailed/class/activity; AC-013/014. |
| A4 — provisional scope interpretation | Basic reporting mandatory; custom filters only possible scope. Source Reporting/spec A4. | Requirements/detailed; AC-015 extension depends OQ-001. |
| A5 — provisional | Connected journeys retrieve current persisted state. Source spec A5. | Architecture/activity/deployment; AC-018/019 omit offline guarantees. |
| A6 — provisional safeguards | Declined is a representation; positive amounts, exactness, confidentiality, durability and deduplication operationalize tracking. Source spec A6. | All artifacts; AC-008/011/016/017; no resubmission entitlement. |
| A7 — planning context | Constitution has unfilled placeholders; no existing external service is assumed; planning can proceed with open policy seams. Source spec A7. | Architecture/deployment; no adopted template gates or service assumptions. |
| A8 — provisional B proposal | Retrieval-based in-app notices; recipient set captured during submission. B §3. | Architecture/detailed/activity; AC-010 delivery/visibility pending OQ-004/009. |

| ID / status | Unresolved decision and safe design boundary | Affected documents / blocked acceptance |
|---|---|---|
| OQ-001 — open | Are custom date ranges/parameters or exports required? Deliver basic categories only; extensions conditional. | Requirements/detailed/activity; custom AC-015 cases. |
| OQ-002 — open | Currency, precision, exact percentages, residual allocation. Use exact types/interfaces; no assumed rounding. | All financial models/flows; fractional AC-007, production monetary schema. |
| OQ-003 — open | Payment authority, confirmation, overpayment, disputes, corrections/deletion/reversal and conflicting changes. Protect saved history; no silent edits/deletes or invented correction workflow. | Detailed/class/activity; production AC-014/016/017 and any future correction cases. |
| OQ-004 — open | Notification channel, timing, failures and acceptance of A8. Persist notices separately from retrieval/delivery; no delivery guarantee. | Architecture/detailed/activity/deployment; delivery AC-010. |
| OQ-005 — open | Browser/OS matrix, workload, latency, availability, accessibility, capacity, retention, backup/recovery targets. No invented benchmarks. | Requirements/deployment; AC-018 and operational acceptance. |
| OQ-006 — open | Account identity, registration, recovery/closure/deletion; receipt types/sizes. Keep identity/upload policies abstract. | Requirements/detailed/class/deployment; AC-001/009/017 and retention checks. |
| OQ-007 — open | Review resubmission and other lifecycle features; policy establishment/version changes for in-flight expenses. Leave later transitions conditional. | Detailed/class/activity; lifecycle AC-012/016. No reminders/delegation commitment. |
| OQ-008 — open | Group and individual reviewers, self-approval, single/multiple-level needs and completion. Eligibility/completion interfaces; no automatic approvals or fixed roles/counts. | All approval design; production AC-011/012. |
| OQ-009 — open | Group management/removal; current/former-member visibility for records and aggregates; privacy versus complete debt. Retain history but evaluate each access; label filtered totals. | All artifacts; permission cases AC-002/004/009/010/013/015. |
| OQ-010 — open, newly exposed schema gap | How are settlements attributed to groups? B payments lack a group field. Show group expense obligations separately; never duplicate personal payments across groups. | Requirements/detailed/class/activity; group net-settled AC-015 cannot be finalized. |

Engineering decisions E1–E6 and proposals EP1–EP2 are recorded in [final architecture](../architecture/final_architecture.md#decisions-and-tradeoffs). Stack/provider/version/hosting/budget and mobile distribution choices are engineering selections still open, separate from customer OQs.

## Source-to-design traceability and final consistency review

| Original feature heading | Requirements / scenarios | Architecture responsibility | Detailed entities/flow and linked artifacts |
|---|---|---|---|
| User accounts | FR-001/002; AC-001/002/019 | Accounts | User / account access; detailed §Component contracts; class; deployment |
| Group management | FR-003/004/017; AC-003/004/017 | Groups/authorization | Group/Membership; detailed group mutation; class |
| Expense creation | FR-005–009; AC-005–009 | Expenses/receipts | Expense/ExpenseShare/Receipt; detailed submission; activity submission |
| Notification system | FR-010; AC-010 | Notices | Notification; detailed notice transaction; activity post-commit retrieval |
| Approval workflow | FR-011/012/016; AC-011/012/016 | Review | ReviewDecision and status; detailed review; activity review |
| Balance tracking | FR-013; AC-013/019 | Financial queries | Derived BalanceView; detailed formula/examples; class read model; activity reads |
| Payment tracking | FR-014/016; AC-014/016 | Payments | Payment; detailed idempotency; activity payment |
| Reporting | FR-015; AC-015 | Reporting/shared queries | Derived ReportView; detailed snapshot/scope/DFD; activity reporting |
| OS supported | NFR-003/005; AC-018/019 | Web/Android/iOS shared API | Every primary flow on each client; deployment nodes/build verification |

Cross-document review checklist (document inspection, not executable application validation):

- [x] All nine source headings retained; custom reporting remains optional.
- [x] Six requested destinations cover all seven subjects in consolidated guidance; database/data flows are within detailed design.
- [x] Consistent `pending`, `approved`, `declined` states; partial reviews stay pending; only approved shares enter balances.
- [x] Shared signed pairwise definition; no stored balance, financial posting, or client-owned financial rules.
- [x] Submission/notices, review/status, and payments have distinct atomic database transactions; image storage is a separate failure boundary.
- [x] IDs, field names, cardinalities, permission boundaries, historical attribution and cross-links align across final artifacts.
- [x] Web, Android and iOS remain required; diagram sources are editable Mermaid.
- [x] Assumptions, open policies and future verification are labeled; no performance or executed-application-test claim.

Remaining source-record conflicts are explicit: Architecture A and earlier design artifacts propose financial postings; the selected B model supersedes that proposal. Consolidated guidance suggests PlantUML activity notation, while the direct user request requires Mermaid; the activity document uses Mermaid flowcharts with UML-style activity semantics and partitions. No unresolved contradiction is silently treated as an approved product policy. Policy and engineering dependencies above prevent unconditional implementation readiness.
