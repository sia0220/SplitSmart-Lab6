# Consolidated Codex Instructions: SplitSmart Requirements and Design

## Task and permitted scope

Generate or update the seven documentation artifacts listed below for SplitSmart. Architecture B has been selected. Use its layered application and derived-balance model as the foundation, incorporating the improvements from Architecture A specified here. Preserve the original customer scope.

This is a documentation-only task. Do not generate application code, scaffolding, executable migrations, automated tests, implementation task lists, infrastructure resources, or deployments. API examples, schema specifications, pseudocode, diagrams, and proposed verification scenarios belong in the design documents only. Do not invoke an implementation workflow.

Inspect repository instructions and existing relevant documentation first. Update corresponding artifacts in place if they already exist; otherwise use the destinations below. Maintain one authoritative artifact for each subject and cross-link existing specification/design files rather than creating conflicting copies. Preserve the original requirements, architecture alternatives, and review as source records. End with a short list of changed documents and unresolved decisions.

## Sources and authority

Read these files before drafting:

1. [Original User Requirements](../../UserRequirement.md): primary authority for customer scope.
2. [Selected Architecture B](architecture_b_codex.md): foundation for technical design.
3. [Architecture A](architecture_a_speckit.md): source of the selected refinements below.
4. [Architecture Review](architecture_review.md): rationale, gaps, tradeoffs, and unresolved decisions.
5. [Existing specification](../../specs/001-splitsmart-specification/spec.md): supporting clarification context, requirement identifiers, assumptions A1–A7, and open questions OQ-001–OQ-009. Consult related existing design artifacts where useful, but do not treat generated details as new customer commitments.

Customer scope takes precedence over either architecture. Distinguish **source requirement**, **derived safeguard**, **selected engineering decision**, **provisional assumption**, and **unresolved customer decision**. Preserve existing requirement and question identifiers where applicable. Cite source headings and links for claims; do not attribute technical choices to the customer.

Complete the documentation with explicit gaps where policy is unknown. Do not invent answers or stop all design work because some policies remain open. Identify which sections and future acceptance criteria depend on each decision. A final architecture baseline can coexist with unresolved product policies; do not label it unconditionally ready for implementation.

## Scope and architectural guardrails

Retain all original capabilities: individual accounts/login and personal information; group creation and management; expense amount, date, description, involved users, equal or percentage split, and optional receipt image; notification of other group members' new expenses; review and approval before financial inclusion; pairwise balances; recording settlement payments; expense, balance, and payment reports; primary web experience plus actual Android and iOS applications.

Treat stored pending submissions as review records, not approved financial entries. Pending, partially reviewed, and declined expenses contribute zero to balances. Describe this distinction consistently when interpreting the source's approval-before-addition wording.

Custom date-range reports and other report parameters remain optional scope pending OQ-001. Do not add exports, payment execution, bank integrations, currency conversion, cross-user debt simplification, offline synchronization, multiple payers, workflow configuration tools, reminders, or delegation as mandatory features. Keep unspecified lifecycle capabilities as open decisions rather than new commitments.

Adopt these engineering decisions from B:

- Three tiers: web/mobile presentation, one application server, and one transactional relational database, with private receipt storage.
- Server layers: controllers/authentication → focused application use cases → shared rules/policies and persistence adapters. Clients share one API and authoritative business rules.
- Derive balances from approved expense shares and recorded payments. Do not add a stored balance table, financial-posting subsystem, asynchronous financial projection, broker, or microservices.
- Centralize authorized financial query definitions for balances and reports. Monetary calculations use exact representations, with precision and residual allocation governed by the unresolved monetary policy.
- Use database transactions, constraints, scoped idempotency, and concurrency control for submission, review, and payment recording.
- Treat PostgreSQL and a shared mobile codebase as B's technical proposals. Record their selection status and rationale; no language, framework, cloud provider, or supported version is established by the original requirements.

Incorporate these refinements from A within B's layers:

- Give accounts, groups/authorization, expenses/receipts, review, financial queries/payments, notifications, and reporting explicit business ownership and dependency contracts. These are logical responsibility boundaries within one server.
- Isolate allocation, approval eligibility/completion, and authorization policies from transport and persistence. Interfaces must expose unresolved choices without silently selecting roles or workflow rules.
- Preserve historical attribution independently of current membership. History retention grants no implied viewing permission to former members.
- Define atomic state transitions and failure boundaries across responsibilities. Expense submission and durable notice records commit together; subsequent delivery/retrieval does not repeat submission or affect balances.
- Describe focused domain, database integration, authorization, and platform verification scenarios. Do not claim tests or performance measurements have been executed.

Do not import A's financial posting step. In B, final approval atomically changes eligibility for the shared balance query; it does not create a second financial representation.

## Deliverables

### 1. Software requirements

Suggested destination: `docs/requirements/software_requirements.md`.

- Define purpose, system boundary, actors without invented privileges, terminology, and all nine original feature headings.
- State functional requirements with stable IDs, source references, rationale, preconditions, observable outcomes, error behavior, and acceptance scenarios. Reuse FR-001–FR-017 where appropriate and distinguish derived safeguards from direct customer statements.
- Cover both split methods, optional receipts, individual/group expense interpretations, approval gating, new-expense notices, pairwise debts, payment recording, required reports, and all three platforms.
- Specify required report categories and reconciliation behavior; mark proposed layouts as design choices. Distinguish expense lists containing pending records from approved financial totals.
- Include confidentiality, exact arithmetic, persistence, retry safety, consistent platform results, and understandable validation. Do not invent numeric performance, capacity, availability, retention, or recovery targets; list them as unresolved inputs.
- Include a provisional action/resource permission matrix covering group mutations, expense submission/review, payment recording, individual records, receipts, notices, reports, and aggregate balances. Mark unknown permissions explicitly instead of granting them by default.
- Provide a traceability table from every original feature heading to requirements and acceptance scenarios, then to the relevant design artifacts.

### 2. Final system architecture

Suggested destination: `docs/architecture/final_system_architecture.md`.

- Describe the selected topology, trust boundaries, component responsibilities, allowed layer dependencies, shared API, private storage, and cross-platform behavior. Include context and container/component diagrams with legends.
- Explain business ownership inside the layered server and prohibit controllers from bypassing application authorization to access repositories directly. Prevent duplicated rules across clients or reports.
- Record architecture decisions and tradeoffs: derived balances, relational transactions, private receipts, durable notices, one server, and shared mobile proposal. Explain which improvements came from A and how they fit B.
- Define transaction boundaries for submission, review completion, and payments; read consistency for reports; and receipt/database failure coordination. Database transactions cannot atomically commit an independent image store.
- Cover authentication, server-side resource authorization, aggregate privacy, secure sessions/tokens, HTTPS, password hashing where applicable, parameterized queries, conditional CSRF controls, mobile credential protection, upload validation, and safe logs/secrets.
- Explain growing balance-read cost and per-recipient notice writes without unsupported scalability claims. Propose measurement and query/index optimization before considering additional financial representations.

### 3. Detailed component design

Suggested destination: `docs/design/component_design.md`.

For each business responsibility and supporting layer, document inputs/outputs, public operations, dependencies, owned data, validation, authorization, transaction ownership, errors, retries, and failure behavior. Specify contracts sufficiently to guide later implementation without writing it.

Cover account access; group management; expense submission/allocation; receipt staging and retrieval; review; payment recording; financial queries; notices; and reports. Include conceptual API operations, request/response fields, authentication context, pagination, and consistent error categories. Identify conditional operations whose policies remain unresolved.

Define policy interfaces for reviewer eligibility, completion/levels, visibility, and monetary allocation. Explain handling of policy changes as an open lifecycle decision. Do not assign an administrator role, permit self-approval, or automatically approve individual expenses.

Define idempotency scope by authenticated caller and operation, payload equivalence/conflict handling, uniqueness, and the outcome of concurrent retries for submission, review, and payment. Distinguish a retried request from a genuinely separate review decision. Describe locking or equivalent concurrency controls and atomic review-decision/status persistence.

Define historical protection: approved shares and recorded payments cannot be silently edited or deleted. Describe correction-policy requirements and design constraints while leaving permitted correction/reversal operations conditional on OQ-003. Do not invent a correction product workflow or financial ledger.

### 4. UML class diagram

Suggested destination: `docs/design/uml_class_diagram.md`.

- Use editable PlantUML or Mermaid class-diagram source, with rendering instructions and a notation legend. Use UML class notation with explicit multiplicities and relationship semantics.
- Model User, Group, Membership, Expense, ExpenseShare, ReviewDecision, Payment, Receipt, and Notification. Include identifiers, principal attributes, exact monetary types, review state, and key invariants.
- Show payer, submitter, participant, reviewer, payment payer/recipient, membership history, optional group, and optional receipt relationships consistently with the chosen schema. Distinguish roles on multiple relationships to User.
- Show relevant application services, policy interfaces, repositories, and a derived balance/report read model in a separate diagram if needed for readability. A balance view is not a persisted entity.
- Explain entity-to-table mappings and any diagram simplifications. Use composition only where ownership/deletion semantics justify it; avoid suggesting that deleting membership or a group deletes financial history. Mark policy-dependent states and fields as provisional.

### 5. UML activity diagram

Suggested destination: `docs/design/uml_activity_diagram.md`.

Use editable PlantUML activity notation with partitions for user/client, application/policies, database, and receipt storage as needed. Supply a main submission-to-review-to-balance activity and supporting payment/report activities if a single diagram becomes crowded.

Show authentication/authorization, input validation, optional image staging, transactional pending submission/shares/receipt metadata/notices, commit or rollback, and post-commit notice retrieval/delivery as separate concepts. Receipt failures and abandoned-upload cleanup must be visible.

Show authorized review, concurrency control, recorded decision, policy evaluation, incomplete approval remaining nonfinancial, decline, final approval commit, and subsequent derived balance retrieval. Include guards, retry paths, denial/failure exits, and transaction boundaries. Show a conditional multilevel loop without inventing level counts or reviewer roles.

For payments, show validation/authorization, duplicate-key lookup, identical retry, conflicting payload rejection, transactional insert, and recalculation. Never show money transfer or a balance-table update. Diagrams and prose must use identical operation and state names.

### 6. Database and data flow design

Suggested destination: `docs/design/database_and_data_flow_design.md`.

- Provide an ER diagram and data dictionary for users, groups, memberships, expenses, expense_shares, review_decisions, payments, receipts, and notifications. Define keys, relationships, nullability, uniqueness, checks, indexes, deletion/history rules, and provisional fields. Specify schema design, not executable migrations.
- Explain optional group membership of an expense, receipt cardinality, one-payer assumption, unique participant shares, historical attribution, monetary units, exact percentages, and validation of cross-row share totals inside a transaction.
- Document exact-sum allocation requirements and leave currency/precision/residual rules unresolved under OQ-002. Illustrative arithmetic must state its assumed units. Prevent combining different currencies if multiple currencies are later selected.
- Define the signed pairwise formula for distinct users A and B, positive when B owes A:

  ```text
  sum(B's shares of approved expenses paid by A)
  - sum(A's shares of approved expenses paid by B)
  - sum(payments from B to A)
  + sum(payments from A to B)
  ```

- Exclude payer self-shares, aggregate expense and payment sources separately before combining, and avoid join multiplication. Specify authorization scope before aggregation and consistent snapshots for multi-query reports. Flag the privacy-versus-complete-debt policy question; do not silently present filtered totals as a complete debt.
- Specify whether a balance/report is personal, group-scoped, or global. Group-specific settlement attribution is unresolved because B's payment records have no group field; document this gap rather than duplicating one payment across groups or inventing an allocation rule.
- Include conceptual query/pseudocode examples and worked examples: approved 90 equally shared by Alice, Bob, and Cara produces Bob→Alice 30 and Cara→Alice 30; Bob's payment of 10 leaves 20; replay leaves 20; remaining payment of 20 leaves zero. Also cover 100 split 50/30/20, opposing debts, self-shares, pending/declined exclusion, and empty results. Fractional allocation outcomes remain conditional on monetary policy.
- Supply a context data-flow diagram and a balanced level-one decomposition. Label external actors, processes, data stores, and named data flows. Keep data-flow arrows distinct from control flow. Cover receipts, notices, reviews, payments, authorized reads, and reports; preserve the same external flows across levels.
- Explain database/image storage reconciliation, staged-file cleanup, notice uniqueness, rollback, and historical protection. Design verification scenarios for concurrency, failure, visibility, and query correctness without claiming execution.

### 7. Deployment design

Suggested destination: `docs/design/deployment_design.md`.

- Provide an editable deployment diagram showing browser, Android and iOS applications, HTTPS endpoint/web hosting, one application-server instance, one relational database, and durable private receipt storage. Distinguish logical components from deployment nodes and identify trust/network boundaries.
- Describe proposed environment separation, configuration/secrets, database access restrictions, web hosting options, authentication configuration, mobile builds/distribution, and API compatibility. State stack/provider/version choices as proposals unless already explicitly selected.
- Document controlled release and schema migration procedures conceptually, backup of database and receipts, restoration consistency, recovery verification, and rollback compatibility limits. Do not supply executable deployment configuration or provision resources.
- Cover safe operational logging, storage monitoring, abandoned receipt cleanup, notification failure handling, and a concise recovery/runbook outline. Retention and recovery objectives remain unresolved; do not assume deleting backups is permitted.
- Identify failure effects of the server, database, and storage dependencies. Explain that a single-instance baseline has availability limits and is not evidence of high availability.
- Keep browser/OS versions, workload, response targets, capacity, recovery objectives, hosting budget, and distribution constraints as explicit decisions. Plan separate Android and iOS acceptance verification even with shared source code.

## Assumptions and open decisions across all artifacts

Maintain one linked register with stable IDs, status, rationale/source, affected artifacts, and consequences. Reuse A1–A7 and OQ-001–OQ-009 from the existing specification; append new identifiers only for genuinely new gaps. Separate accepted architecture decisions from assumptions requiring customer confirmation.

Explicitly carry forward these provisional interpretations:

- Participants have registered accounts; groups have identifying names.
- An expense has one payer, who may also participate; sharing without a permanent group is supported as a working interpretation.
- Payments record external settlements; balances are pairwise net obligations.
- Journeys assume connectivity and retrieval of current persisted data.
- In-app retrieval-based notices and a recipient set captured at submission are B's proposed baseline, pending delivery and visibility agreement. Persistence alone is not proof of user delivery.
- A declined state is a design representation, not an agreed resubmission lifecycle. Retaining historical attribution establishes no former-member access entitlement.

Carry forward unresolved decisions about custom reports; currency/precision/rounding; payment authority, confirmation, overpayment and corrections; notification channel/timing/failures; platform and quality targets; account lifecycle and receipt limits; review lifecycle/policy changes; reviewer eligibility, self-approval and level completion; and group management/current/former-member visibility. Include individual-expense approval explicitly. Record stack and hosting choices separately as engineering decisions.

For every unresolved policy, identify the safe design boundary and the future behavior/acceptance criteria it blocks. Do not fabricate defaults merely to fill a diagram or schema column. Do not convert account recovery/deletion, advanced reviews, or exports into mandatory scope by listing them as open questions.

## Cross-document review and completion criteria

Before finishing, review all seven artifacts together and fix inconsistencies:

1. Every original feature heading maps to requirements, component responsibility, relevant entities/flows, platform coverage, and proposed acceptance evidence.
2. Requirement IDs, terms, states, operations, policy references, field names, cardinalities, and transaction boundaries agree across prose and diagrams. Links resolve and diagram source uses the declared syntax.
3. Approval is the only route for an expense to enter financial calculations. No activity, class, schema, or deployment view introduces stored balances or postings.
4. Equal/percentage calculations, opposing obligations, self-shares, payments, retries, failures, and report totals follow one financial definition. Unknown fractional-unit policy remains explicit.
5. Authorization covers writes, reads, receipts, notices, and aggregates. Membership history, current visibility, and financial attribution remain distinct.
6. Database transactions and image-storage cleanup are accurately separated. Persisted notices, delivery, and receipt access are not conflated.
7. Web, Android, and iOS remain required; custom reporting and other undecided capabilities are not silently added or omitted as settled decisions.
8. Assumptions and unresolved decisions are linked consistently, with affected acceptance conditions identified. No invented benchmark, policy, stack commitment, or claim of executed testing appears.

Include a final traceability/consistency checklist in the documentation and report any remaining contradictions or decision dependencies. Stop after documentation updates; do not begin application implementation.
