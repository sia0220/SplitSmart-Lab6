# SplitSmart Architecture Review

**Date:** November 5, 2026  
**Recommendation:** Select Architecture B as the implementation baseline, subject to the decisions and refinements below.

## Evaluation basis

The primary authority is the original [User Requirements](../../UserRequirement.md). The alternatives reviewed are [Architecture A: Spec Kit](architecture_a_speckit.md) and [Architecture B: Layered Application with Derived Balances](architecture_b_codex.md). The [generated specification](../../specs/001-splitsmart-specification/spec.md) supplies clarification context and identifies assumptions; it does not turn those assumptions into original customer requirements.

This is a document review, not an implementation audit. Coverage means that a design addresses a capability, not that working software or passing tests demonstrate it. Neither architecture has measured performance results. More detail improves implementation readiness but does not by itself establish better architecture.

The original document requires accounts, group management, expense creation, notifications, approval, pairwise balances, payment recording, reporting, and web/Android/iOS support. Equal and percentage splits and optional receipt images are explicit requirements. Custom reports with date ranges and other parameters are possible extensions, rather than unequivocal mandatory scope. The document establishes no technology stack, workload, delivery deadline, team size, or operational service levels.

## Requirements coverage

| Original requirement | Architecture A | Architecture B | Assessment / unresolved detail |
|---|---|---|---|
| Individual accounts, login, personal expenses and balances | Accounts module and shared identity boundary | Authentication controllers, account use cases, users table | Both cover the capability. Identity, recovery, and account closure remain undecided. |
| Create and manage groups | Dedicated groups/authorization module | Group use cases, groups and membership records | Both cover the structure, but neither defines complete management operations or permissions. |
| Expense amount, date, description, participants | Traced through FR-005–009 and expense module | Explicit submission flow and schema | B is more self-contained. A relies more on the referenced specification for field-level detail. |
| Equal and percentage splits | Allocation validation proposed | Exact shares, percentage validation, transaction checks | Both address the requirement; currency precision and residual allocation remain unresolved. |
| Optional receipt image | Private storage and authorized access | Private staged uploads, metadata, cleanup, access checks | Both cover receipts; B specifies more failure handling. Formats and size limits remain open. |
| Notify users of other group members' new expenses | Transactional notification intent and eventual delivery | Per-recipient in-app records committed with submission | Both provide a mechanism. B's retrieval-based channel requires acceptance; A leaves delivery implementation open. |
| Review and approve before balance adjustment; potentially multiple levels | Dedicated review module and policy boundary | Transactional review procedure and policy functions | Both enforce the gate conceptually. Neither establishes approvers or a complete workflow policy. |
| Balance owed by each user to every other user | Financial effects and pairwise balance responsibility | Explicit pairwise formula over approved shares and payments | B explains the calculation more clearly. A leaves posting representation and balance calculation unspecified. |
| Record settlement payments and update balances | Payment effects with duplicate-effect protection | Payment records, request identifiers, derived totals | Both cover recording. Permission, confirmation, overpayment, and correction policies remain open. |
| Reports and summaries of expenses, balances, payments | Reporting module tied to financial records | Shared authorized queries and consistent snapshots | Both cover basic reporting; report shapes remain undefined. Custom filters/exports need a scope decision. |
| Primary web plus Android and iOS applications | Separate web/mobile boundaries; mobile approach deferred | Shared mobile codebase packaged for both platforms | Both retain actual apps in scope. Framework, supported versions, packaging, and platform acceptance remain unfinished. |

No original feature heading is entirely absent from either design. Coverage is nevertheless conditional where unresolved policies determine observable behavior.

## Individual review: Architecture A

### Requirements coverage

A preserves the original scope and explicitly keeps unapproved expenses out of balances. Its review policy interface accommodates single or multilevel approval without inventing roles. It also correctly avoids treating a responsive website as sufficient mobile delivery.

Its coverage is primarily at responsibility and traceability level. The standalone document does not explain individual expense handling, expense fields, financial arithmetic, or report outputs as concretely as B. Referencing the specification supports coverage, but additional design work is needed before implementation.

### Modularity

The accounts, groups, expenses, review, financial, notification, and reporting boundaries align well with business responsibilities. One shared backend keeps client behavior consistent. These are logical modules, so A does not impose microservice infrastructure.

The expense/review/financial boundaries need explicit ownership and transaction contracts. Approval completion spans those responsibilities; separate modules must still participate in the proposed atomic database transaction. Module names alone do not establish dependency direction or prevent cycles.

### Maintainability

Feature-oriented organization can localize future changes and give reviewers clear places to inspect business rules. Policy interfaces isolate decisions likely to change. Preserving historical attribution independently of membership is a sound foundation.

Maintenance risk comes from unspecified contracts and financial representation. A unique source reference prevents duplicate posting only when backed by appropriate constraints and transactional behavior. If postings duplicate information in expense shares, reconciliation and correction rules become additional responsibilities. A does not explicitly require a stored balance table, so that cost should not be attributed to it automatically.

### Scalability

A shared backend and relational storage are reasonable starting points. Notification intents separate reliable persistence from delivery retries. Logical modules could support later extraction if a demonstrated need arises.

There is no query strategy, indexing plan, worker design, or capacity evidence. A posting subsystem could support efficient reads if designed accordingly, but posting alone does not guarantee faster balance queries. No scalability superiority over B can be established from this document.

### Security

A puts authorization on the backend, covers receipts and reports, and distinguishes historical attribution from access rights. It recognizes that notification contents can disclose information and must follow visibility policy.

Authentication/session controls, credential protection, transport security, upload validation, secrets handling, and logging safeguards are not specified. These are design gaps, not evidence of an insecure implementation. Production authorization also remains blocked on the permission policy.

### Testability

A proposes domain unit, transactional integration, authorization contract, and platform journey tests. Module boundaries offer useful seams for allocation and approval tests. Atomic completion and duplicate-effect behavior provide concrete integration-test targets.

The plan does not define detailed fixtures, transaction failure cases, or expected posting reconciliation. Final authorization and workflow tests cannot be completed until the relevant policies are agreed.

### Implementation complexity and scope suitability

Deployment can remain small because separate services are unnecessary. Nevertheless, defining seven boundaries and coordinating approval with financial posting adds design and testing work. Deferred stack and persistence details also increase the work remaining before coding.

A is a credible foundation when feature separation and future policy changes are priorities. For the currently stated scope, its financial posting mechanism needs a clearer benefit to justify its extra coordination. It is a sound provisional architecture, but less ready to implement than B.

## Individual review: Architecture B

### Requirements coverage

B translates the same features into concrete use cases, records, and transactions. It explicitly covers individual expenses without a group, optional receipts, notifications to other current group members, approval gating, pairwise balances, and all three platforms.

Its formula shows how expenses and payments produce obligations and avoids cross-user debt simplification. In-app notices are a proposed interpretation of “notify,” not proof that customer delivery expectations are met. Group management, report presentation, and workflow behavior still need specification.

### Modularity

Controllers, application procedures, rule functions, and persistence have distinct responsibilities. Pure monetary and policy functions can remain independent of HTTP and database code. A shared financial query definition supports consistent balances and reports.

Horizontal layers make a feature change touch several directories. Procedures can grow too large, and generic repositories can expose records without proper access checks. B acknowledges this risk; implementation should enforce application entry points and keep use cases focused.

### Maintainability

Approved expense shares and recorded payments form a small, understandable financial source of truth. Derived balances remove the need to synchronize an additional posting representation. Central query definitions and migrations support consistent changes.

Correctness depends heavily on query semantics, authorization scope, and stable historical records. Directly editing an approved share would immediately change balances. Corrections therefore need explicit authorized workflows and history preservation; derived balances alone do not provide an audit trail. Idempotency identifiers also need defined caller/operation scope and payload-conflict handling across submission and review, as B already describes for payments.

### Scalability

B proposes indexes and separates aggregation of expenses and payments to prevent join multiplication. One server and database are proportionate to an initial deployment with no specified high-load target.

Balance computation grows with relevant financial history, and synchronous creation of one notice per group member increases submission work with group size. Actual limits require representative queries and measurements. Optimize queries and pagination first; introduce cached or materialized totals only if measurements justify the resulting consistency work. A shared API does not eliminate database or receipt-storage bottlenecks.

### Security

B gives stronger implementation guidance: password hashing, HTTPS, protected sessions/tokens, parameterized queries, conditional CSRF protection, protected mobile credential storage, private validated uploads, and restricted logs. It explicitly guards against leaking inaccessible records through aggregate totals.

These measures still depend on a completed authorization policy and implementation verification. Record-level query scoping must be applied consistently, and the meaning of an authorized total must be settled so privacy filtering does not silently misrepresent a user's debt. Account abuse controls and lifecycle behavior also need decisions before deployment.

### Testability

Pure rules and shared financial queries support focused tests. B identifies concurrency, retries, rollbacks, access denial, historical continuity, and cross-platform journeys. Derived balances allow direct checks against known source records without separately reconciling postings.

Database integration tests remain essential: pure function tests cannot establish locking, uniqueness, snapshot behavior, or aggregation correctness. Include opposing obligations, payer self-shares, repeated review/payment requests, and receipt/database failure boundaries. Fractional-unit expectations depend on the monetary policy.

### Implementation complexity and scope suitability

B reduces financial synchronization work and specifies enough persistence and deployment detail to guide implementation. Receipt staging and cleanup, mobile packaging, and multilevel review still require substantial work; a small server topology does not make the entire product trivial.

It is the better fit for a modest initial project with the documented capabilities. Its “student project” rationale is contextual rather than a constraint stated in the original requirements. The recommendation rests on proportional complexity and clearer execution details, not an assumed team size or deadline.

## Comparison

The judgments below are qualitative assessments of the documents, not benchmark scores.

| Criterion | Architecture A | Architecture B | Comparative judgment |
|---|---|---|---|
| Requirements coverage | All feature areas traced; behavior often deferred | All feature areas mapped to concrete flows | Similar breadth; B provides clearer realization |
| Modularity | Business-oriented modules | Technical layers and focused use cases | A offers clearer feature boundaries; B requires fewer financial coordination points |
| Maintainability | Good change isolation; posting contracts unfinished | Fewer financial representations; central queries | B initially simpler; A useful as feature complexity grows |
| Scalability | Growth seams, but read/storage strategy incomplete | Index plan and direct queries; growing read cost | Neither proven; no stated requirement demands a more distributed design |
| Security | Correct trust boundaries; fewer concrete controls | More explicit controls, including aggregate privacy | B is more actionable; both await access policy |
| Testability | Strong module-level testing outline | Concrete query, concurrency, and failure targets | Both testable; B has clearer immediate test cases |
| Implementation complexity | Moderate, with more unresolved design and posting coordination | Lower financial coordination; concrete single-database design | B has the smaller initial implementation burden |
| Scope suitability | Reasonable provisional design with stronger feature separation | Proportionate baseline for stated features | B preferred for initial delivery |
| Operational readiness | Deployment and recovery mostly deferred | Deployment, durable images, backups, restoration addressed | B more complete; operational targets remain open |
| Financial model | Approved effects posted atomically with review completion | Balances calculated from approved shares and payments | B minimizes synchronization; A could support richer posting history if explicitly designed |

## Missing decisions and unsupported assumptions

Neither document is ready for unconditional implementation acceptance. The following distinguish customer decisions from engineering details:

| Issue | Gap or assumption | Required resolution |
|---|---|---|
| Approval workflow | Policy interfaces do not define reviewers, self-approval, levels, individual-expense approval, or policy changes | Agree eligible actors and completion/state-transition rules before implementing real approval behavior. |
| Group management and visibility | Operations, management rights, removal with debt, and former-member access remain open | Define an action/resource permission matrix, including receipts, notices, reports, and aggregate balances. |
| Monetary rules | Exact totals are proposed safeguards; currencies, precision, and residual allocation are unspecified | Choose supported currencies and deterministic allocation rules; prevent invalid cross-currency aggregation if multiple currencies are supported. |
| Payment and correction policy | Neither design settles who may record settlements, recipient confirmation, overpayments, disputes, or reversals | Define authorized operations and their effect on history and balances. Payment execution is outside the documented requirement. |
| Notifications | A leaves channel/delivery open; B assumes retrieval-based in-app notices and captures recipients at submission | Confirm acceptable channel and timing, retry behavior where applicable, and access after membership changes. Persisting a notice is distinct from user delivery. |
| Reporting | Basic summaries are covered without concrete report definitions; custom filters remain optional | Define required views and totals; decide whether date ranges, exports, and other parameters enter scope. |
| Accounts and receipts | Registration identity, recovery/deletion, attachment formats and limits are unspecified | Define lifecycle and attachment policies, then implement the associated security and retention behavior. |
| Platforms and quality | No supported browser/OS matrix, performance workload, or recovery targets | Agree acceptance environments and proportionate quality targets. Shared code still requires Android and iOS verification. |
| Expense attribution | Registered participants, one payer, and optional groupless expenses are working interpretations | Confirm these interpretations; multiple payers would require a changed model. A describes groupless expenses less explicitly than B. |
| Connectivity and settlement semantics | Online retrieval, pairwise netting, and externally made payments are assumptions | Confirm them. Offline synchronization, money transfer, conversion, and cross-user debt simplification are not original commitments. |
| Technology and deployment | Relational storage, private image storage, B's PostgreSQL/shared mobile proposal, and A's posting design are engineering proposals | Select a stack based on available skills and deployment constraints; do not present these choices as customer requirements. |

Additional implementation details need engineering design rather than new product scope: API contracts and errors, request-key scoping, pagination, enforced module/layer dependencies, transaction isolation, and receipt cleanup across storage/database failures. A needs a concrete posting and reconciliation design if retained. B needs reviewed aggregation queries and a historical correction approach. Neither design demonstrates a complete operational runbook or a tested recovery process.

## Recommendation and tradeoffs

Select **Architecture B** as the baseline. It covers the original capabilities with one application server, one transactional database, and a clear calculation from approved expenses and payments. Its transaction flows, persistence outline, security controls, and deployment considerations reduce the amount of design still needed before implementation. With no stated high-load or independent-service requirement, an additional financial posting representation has no demonstrated benefit sufficient to outweigh its coordination cost.

The tradeoff is greater dependence on relational query correctness and potentially higher balance-read cost as history grows. Layered code also needs discipline to prevent oversized procedures and scattered feature logic. These are manageable initial risks, but should be verified through database tests and representative measurements rather than assumed away.

Carry A's business boundaries into B as clear ownership and dependency rules for accounts, groups, expenses, review, notifications, and reporting. This does not require separate deployments or adding financial postings. Keep B's derived balances initially. If future requirements establish complex financial adjustments, stronger audit needs, or measured read bottlenecks, evaluate postings or projections with an explicit reconciliation and migration plan; A's current document does not already solve those issues.

Before implementing affected behavior, resolve approval, authorization, monetary, payment, and notification decisions. Finalize required report views, platform support, and the stack as delivery planning proceeds. Acceptance should demonstrate both split methods, zero balance impact before approval, correct pairwise settlements, retry/concurrency safety, authorized receipts and reports, and the required journeys on web, Android, and iOS.
