# SplitSmart UML class diagram — CIS 285 Lab 7

This editable Mermaid `classDiagram` follows [final architecture](../architecture/final_architecture.md) and the [detailed schema/contracts](detailed_design.md). Requirements and policy status are in the [shared register](../requirements/updated_requirements.md#assumption-and-decision-register). It is a conceptual model, not generated code or a selected ORM.

## Domain entities

```mermaid
classDiagram
    class User {
        +ID id
        +String identityRef
        +String credentialHash_optional
        +Timestamp createdAt
    }
    class Group {
        +ID id
        +String name
        +Integer version
        +Timestamp createdAt
    }
    class Membership {
        +ID id
        +Timestamp joinedAt
        +Timestamp leftAt_optional
    }
    class Expense {
        +ID id
        +MoneyUnits amountUnits
        +MonetaryUnit monetaryUnit
        +Date expenseDate
        +String description
        +SplitMethod splitMethod
        +ReviewState reviewState
        +Integer version
        +Timestamp createdAt
        +String policyRef_provisional
    }
    class ExpenseShare {
        +MoneyUnits amountUnits
        +ExactPercent percentage_optional
    }
    class ReviewDecision {
        +ID id
        +ReviewOutcome outcome
        +Timestamp decidedAt
        +String stageRef_provisional
    }
    class Payment {
        +ID id
        +MoneyUnits amountUnits
        +MonetaryUnit monetaryUnit
        +Date paymentDate
        +Timestamp createdAt
    }
    class Receipt {
        +ID id
        +String storageKey
        +String mediaType
        +Integer byteSize
        +String contentDigest
        +Timestamp createdAt
    }
    class Notification {
        +ID id
        +Timestamp createdAt
    }
    class ReviewState {
        <<enumeration>>
        pending
        approved
        declined
    }
    class SplitMethod {
        <<enumeration>>
        equal
        percentage
    }
    class ReviewOutcome {
        <<enumeration>>
        approve
        decline
    }
    User "1" -- "0..*" Membership : account history
    Group "1" -- "0..*" Membership : group history
    Group "0..1" -- "0..*" Expense : optional context
    User "1" -- "0..*" Expense : payer
    User "1" -- "0..*" Expense : submitter
    Expense "1" -- "1..*" ExpenseShare : allocated shares
    User "1" -- "0..*" ExpenseShare : participant
    Expense "1" -- "0..*" ReviewDecision : decision history
    User "1" -- "0..*" ReviewDecision : reviewer
    User "1" -- "0..*" Payment : payment payer
    User "1" -- "0..*" Payment : payment recipient
    User "1" -- "0..*" Payment : recorded by
    Expense "1" -- "0..1" Receipt : optional image
    Expense "1" -- "0..*" Notification : new expense source
    User "1" -- "0..*" Notification : intended recipient
    Expense ..> ReviewState : state type
    Expense ..> SplitMethod : method type
    ReviewDecision ..> ReviewOutcome : outcome type
```

Legend: `--` is an association, `..>` a type/dependency reference, quoted endpoints are explicit UML multiplicities (`1`, `0..1`, `0..*`, `1..*`). `+` denotes a conceptual public attribute, not permission to mutate it through the API. No composition diamond is used: deleting a group, membership or user must not imply deletion of financial history.

Each Expense has exactly one payer and submitter (possibly the same person), zero/one group, one or more participant shares and zero/one receipt. Each membership episode belongs to one group and one user, retaining joined/left history. Payer is allowed to participate under A2 but need not have a self-share. A Payment has distinct payer and recipient, one authenticated recorder and no group link under B; group settlement attribution remains OQ-010. A recipient may have many notices; one notice identifies one expense.

`MoneyUnits` is an exact integer-unit type; `ExactPercent` is exact decimal/rational, never binary float. Their concrete precision and residual rules await OQ-002. Sum(shares)=expense amount; percentage totals equal 100 for percentage splits. ExpenseShare identity is the composite expense/participant key. Pending and partially reviewed expenses stay `pending`; only `approved` shares enter balance queries; `declined` is provisional (A6). Review outcome/stage interpretation and policyRef binding await OQ-007/008. No automatic individual approval or fixed reviewer role is implied.

## Application services, interfaces and derived views

```mermaid
classDiagram
    class ApplicationUseCases {
        +SubmitExpense(actor, command)
        +RecordReview(actor, command)
        +RecordPayment(actor, command)
        +ReadBalances(actor, scope)
        +ReadReports(actor, scope)
        +ReadNotices(actor, cursor)
    }
    class AuthorizationPolicy {
        <<interface>>
        +authorize(actor, action, context)
        +scopeFor(actor, kind, scope)
    }
    class AllocationPolicy {
        <<interface>>
        +allocate(amount, unit, participants, method, percentages)
    }
    class ApprovalEligibilityPolicy {
        <<interface>>
        +check(actor, expense, decisions, context)
    }
    class ApprovalCompletionPolicy {
        <<interface>>
        +evaluate(expense, decisions, context)
    }
    class Repositories {
        <<interface>>
        +loadScoped(context)
        +saveWithinTransaction(records)
        +lockExpense(id)
    }
    class UnitOfWork {
        <<interface>>
        +begin()
        +commit()
        +rollback()
    }
    class ReceiptAdapter {
        <<interface>>
        +stage(actor, image)
        +readAuthorized(reference)
        +cleanupUnreferenced()
    }
    class FinancialQueryService {
        +ReadBalances(actor, scope)
        +ReadReports(actor, scope)
    }
    class BalanceView {
        <<derived>>
        +ID userA
        +ID userB
        +MoneyUnits signedAmount
        +MonetaryUnit monetaryUnit
        +String scopeAndCompleteness
    }
    class ReportView {
        <<derived>>
        +String category
        +String scopeAndSnapshot
        +String sourceReferences
        +String reconciledTotals
    }
    ApplicationUseCases ..> AuthorizationPolicy : authorizes
    ApplicationUseCases ..> AllocationPolicy : validates allocation
    ApplicationUseCases ..> ApprovalEligibilityPolicy : checks reviewer
    ApplicationUseCases ..> ApprovalCompletionPolicy : determines state
    ApplicationUseCases ..> Repositories : persists through
    ApplicationUseCases ..> UnitOfWork : owns transaction
    ApplicationUseCases ..> ReceiptAdapter : stages and retrieves
    ApplicationUseCases ..> FinancialQueryService : shared reads
    FinancialQueryService ..> AuthorizationPolicy : scopes before totals
    FinancialQueryService ..> Repositories : reads sources
    FinancialQueryService ..> BalanceView : derives
    FinancialQueryService ..> ReportView : derives
```

This is a dependency diagram rather than an instance association diagram, so multiplicities do not apply to its dashed dependency arrows. ApplicationUseCases abbreviates the separate focused operations and business owners detailed in [component contracts](detailed_design.md#component-contracts); Repositories abbreviates per-responsibility interfaces. Accounts, groups and notices retain their own focused use cases even though only representative operations appear here. Controllers call these operations and never repositories directly.

Entity/table mappings are User→users, Group→groups, Membership→memberships, Expense→expenses, ExpenseShare→expense_shares, ReviewDecision→review_decisions, Payment→payments, Receipt→receipts, Notification→notifications. Foreign-key fields are expressed by labeled associations to reduce duplication; their names/nullability appear in the data dictionary. Command-key retry metadata, upload staging details and identity-provider/session records are supporting storage concerns omitted from the domain drawing. BalanceView and ReportView are query results, **not persisted tables**. Policy interfaces define seams, not settled customer policy. Approved shares/payment history cannot be silently edited or deleted; correction operations remain OQ-003.

## Rendering

Open this Markdown in a Mermaid-capable preview, or copy each fenced block into a Mermaid editor supporting class diagrams. The source remains editable; no image replaces it. Use [activity diagrams](activity_diagram.md) for workflow semantics and [deployment design](deployment_design.md) for runtime nodes. Diagram rendering is a documentation check, not application verification.
