# SplitSmart activity diagrams — CIS 285 Lab 7

These editable Mermaid flowcharts express UML-style activities with partitions, action nodes, guarded decisions, loops and start/end nodes. Mermaid uses `flowchart` rather than a native UML `activityDiagram` keyword. This satisfies the explicit Mermaid request; it replaces the consolidated guidance's suggested PlantUML notation without changing behavior. Paste a fenced block into a Mermaid-capable Markdown preview/editor to render it.

Legend: rounded start/end nodes mark activity boundaries; rectangles are actions; diamonds are decisions; labeled arrows are guards/control flow; subgraphs are responsibility partitions. A transaction's BEGIN/COMMIT/ROLLBACK actions delimit database atomicity; receipt storage is outside that boundary. No parallel fork or distributed atomic commit is implied. The workflow can resume in later requests; a successful pending submission does not wait for review to return to the client.

Operations, fields and states match [detailed design](detailed_design.md), [class model](class_diagram.md) and [requirements](../requirements/updated_requirements.md). Policy IDs refer to the [single register](../requirements/updated_requirements.md#assumption-and-decision-register). Guards concerning eligibility/completion are conditional on agreed policies, not fixed reviewer roles, approval counts or permissions.

## SubmitExpense: review record and durable notices

```mermaid
flowchart TD
    subgraph Client["User / client"]
        S([Start]) --> Input["SubmitExpense: fields, optional image, stable request key"]
        Success["Receive pending expense; zero balance contribution"]
        Retry["Retry same operation, key and payload after uncertain outcome"]
        Fail([Failure or denial shown])
        Done([Submission request complete])
    end
    subgraph App["Application / policies"]
        Auth{"Authenticated and authorized?"}
        Preflight{"Committed caller/SubmitExpense/key already exists?"}
        PreflightSame{"Committed payload equivalent?"}
        Valid{"Input and exact allocation valid?"}
        ReceiptQ{"Optional image supplied?"}
        VerifyToken{"Stage token owner and content reference valid?"}
        ReturnOld["Reauthorize and return original committed response"]
        Conflict["Reject conflicting payload or stale context"]
        Response["Return success only after commit"]
        Unknown{"Response received?"}
        Notice["ReadNotices after commit; reauthorize recipient and content"]
        Delivery["Retrieve proposed in-app notice; failure retries retrieval only"]
    end
    subgraph Images["Private receipt storage"]
        Stage["Stage validated image durably under private stable key"]
        StageOK{"Staging succeeded?"}
        Clean["Coordinate cleanup of abandoned object; check no DB reference or active submission"]
    end
    subgraph DB["Transactional database"]
        Begin["BEGIN; reserve caller/SubmitExpense/key"]
        Key{"Existing committed key?"}
        Same{"Equivalent payload?"}
        Context["Lock group context if present; recheck membership and authorization"]
        Save["Insert pending Expense and exact ExpenseShares; optional Receipt metadata"]
        Notices["Insert unique notices for other current group members; save command result"]
        Commit{"All writes and COMMIT succeed?"}
        Rollback["ROLLBACK; no partial expense, shares, receipt metadata or notices"]
    end
    Input --> Auth
    Auth -->|No| Fail
    Auth -->|Yes| Preflight
    Preflight -->|Yes| PreflightSame
    PreflightSame -->|Yes| ReturnOld
    PreflightSame -->|No| Conflict
    Preflight -->|No; authoritative recheck in transaction| Valid
    Valid -->|No| Fail
    Valid -->|Yes| ReceiptQ
    ReceiptQ -->|Yes| Stage
    Stage --> StageOK
    StageOK -->|No| Fail
    StageOK -->|Yes| VerifyToken
    VerifyToken -->|Yes| Begin
    VerifyToken -->|No| Fail
    ReceiptQ -->|No| Begin
    Begin --> Key
    Key -->|Yes| Same
    Same -->|Yes| ReturnOld
    Same -->|No| Conflict
    Conflict --> Fail
    ReturnOld --> Done
    Key -->|No; concurrent keys serialize| Context
    Context -->|Valid| Save
    Context -->|Denied or invalid| Rollback
    Save --> Notices
    Notices --> Commit
    Commit -->|No| Rollback
    Rollback --> Clean
    Clean --> Fail
    Commit -->|Yes| Response
    Response --> Unknown
    Unknown -->|Yes| Success
    Success --> Done
    Unknown -->|No| Retry
    Retry --> Auth
    Response --> Notice
    Notice --> Delivery
```

Submission and notice retrieval are separate operations even though shown together. Notice records commit atomically with submission; actual delivery/retrieval is post-commit and provisional A8/OQ-004. A subsequent retrieval does not create another expense. If image bytes cannot be staged, do not save an attached submission or claim success. Staged objects are referenced without an indispensable post-commit file move. Cleanup also runs independently for abandoned uploads, using the same reference/active-submission checks; failure before database work can likewise leave an orphan. StageReceipt retries reuse their token when available. A mutation retry that lost its response resolves its command key before attempting a fresh submission or requiring a consumed stage token; the optional staging path above is for a new submission. DB/key/storage exceptions use the rollback or safe failure exits as applicable; no distributed transaction is implied.

## RecordReview: pending to approved or declined

```mermaid
flowchart TD
    subgraph Reviewer["Authorized user / client"]
        Start([Review request begins]) --> Request["RecordReview: expense, outcome, expectedVersion and key"]
        Fail([Denial, conflict or retryable failure])
        Later["Later eligible review request with its own command key"]
        Finish([Review request complete])
    end
    subgraph App["Application / approval policies"]
        Auth{"Authenticate and authorize review access?"}
        Replay["Reauthorize and return original recorded outcome"]
        Eligible{"Current state/version and reviewer/stage eligibility valid?"}
        Evaluate["ApprovalCompletionPolicy evaluates decisions"]
        State{"Policy result?"}
        Return["Return committed decision and state; no posting"]
        More{"Pending and further review required by policy?"}
        Read["ReadBalances with authorized scope"]
    end
    subgraph DB["Transactional database"]
        Begin["BEGIN; reserve caller/RecordReview/key; lock expense"]
        Key{"Key outcome?"}
        Append["Append ReviewDecision"]
        Pending["Keep reviewState pending; zero financial contribution"]
        Declined["Set reviewState declined; zero financial contribution"]
        Approved["Set reviewState approved; shares become query eligible"]
        Persist["Save version and command response atomically with decision/status"]
        Commit{"COMMIT succeeds?"}
        Rollback["ROLLBACK; decision and status unchanged"]
        Query["Shared query derives balance from approved shares and payments"]
    end
    Request --> Auth
    Auth -->|No| Fail
    Auth -->|Yes| Begin
    Begin --> Key
    Key -->|Same payload committed| Replay
    Key -->|Different payload| Rollback
    Key -->|New command| Eligible
    Replay --> Finish
    Eligible -->|No| Rollback
    Eligible -->|Yes| Append
    Append --> Evaluate
    Evaluate --> State
    State -->|Incomplete| Pending
    State -->|Unsuccessful review under agreed policy| Declined
    State -->|Approval complete| Approved
    Pending --> Persist
    Declined --> Persist
    Approved --> Persist
    Persist --> Commit
    Commit -->|No| Rollback
    Rollback --> Fail
    Commit -->|Yes| Return
    Return --> More
    More -->|Yes; conditional multilevel loop| Later
    Later --> Request
    More -->|No| Finish
    Return --> Read
    Read --> Query
```

Policy evaluation exceptions roll back the transaction. Review row locking serializes concurrent decisions; eligibility and completion are evaluated against the locked current record. An identical retry returns its original outcome; changed content under the same key conflicts. After lost response or transient failure, retry with the same key/payload; after a genuine stale-version conflict, refresh and deliberately choose a new command only if policy permits. New-key repeated decisions are evaluated as new decisions and may be rejected. Partial multilevel progress remains `pending`, with no invented stage count. Decline semantics, later resubmission, self-approval and individual-expense reviewers remain OQ-007/008. Approval commit changes query eligibility only; no balance table or financial posting is written.

## RecordPayment and subsequent balance retrieval

```mermaid
flowchart TD
    subgraph Client["User / client"]
        Start([Start]) --> Input["RecordPayment: parties, amountUnits, unit, date and stable key"]
        Fail([Show validation, denial or conflict])
        Retry["After transient or uncertain outcome retry identical command"]
        Saved["Receive original or newly saved Payment"]
        Done([Complete])
    end
    subgraph App["Application / policies"]
        Auth{"Authenticated and permitted recorder?"}
        Valid{"Valid distinct parties, positive exact amount and date?"}
        Equiv{"Existing payload equivalent?"}
        Replay["Reauthorize and return original payment result"]
        Policy{"Agreed acceptance and concurrency checks pass?"}
        Read["ReadBalances in authorized personal scope"]
    end
    subgraph DB["Transactional database"]
        Begin["BEGIN; reserve caller/RecordPayment/key"]
        Key{"Committed duplicate key?"}
        Insert["Insert Payment and save command response; no money transfer"]
        Commit{"COMMIT succeeds?"}
        Rollback["ROLLBACK; no new payment"]
        Query["Derive net from approved shares and payments; never update balance row"]
    end
    Input --> Auth
    Auth -->|No| Fail
    Auth -->|Yes| Valid
    Valid -->|No| Fail
    Valid -->|Yes| Begin
    Begin --> Key
    Key -->|Yes| Equiv
    Equiv -->|Yes| Replay
    Equiv -->|No; conflict| Rollback
    Replay --> Saved
    Key -->|No; concurrent duplicate waits| Policy
    Policy -->|No| Rollback
    Policy -->|Yes| Insert
    Insert --> Commit
    Commit -->|Yes| Saved
    Commit -->|No| Rollback
    Rollback -->|Validation or conflict| Fail
    Rollback -->|Transient error| Retry
    Retry --> Input
    Saved --> Read
    Read --> Query
    Query --> Done
```

Lost responses also go to Retry with the same key. Same-payload duplicates do not insert again; different-key commands are distinct payments, subject to OQ-003. Permission, recipient confirmation, overpayment and correction rules are unresolved: the acceptance-check node cannot be implemented with arbitrary defaults. If confirmation is required, payment states/query inclusion need refinement first. OQ-010 prevents treating personal recorded payments as belonging to every group.

## ReadReports: authorized consistent summaries

```mermaid
flowchart TD
    S([Start]) --> R["ReadReports: category and requested scope"]
    R --> Auth{"Authenticated and authorized scope available?"}
    Auth -->|No| Denied([Deny without disclosure])
    Auth -->|Yes| Scope["Resolve permitted sources, unit and completeness descriptor"]
    Scope --> Snap["Begin consistent database read snapshot"]
    Snap --> Reads["Read labeled expense states, separately aggregate approved shares and payments"]
    Reads --> OK{"All report reads succeed?"}
    OK -->|No| Error([Return retryable failure, not fabricated zero totals])
    OK -->|Yes| Reconcile["Reconcile same scope/unit/snapshot through shared financial definitions"]
    Reconcile --> Group{"Group-settled balance requested without agreed attribution?"}
    Group -->|Yes| Gap["Explain OQ-010; distinguish group expense obligations from personal settlements"]
    Group -->|No| Result["Return authorized summaries, scope and explicit empty states if applicable"]
    Gap --> Result
    Result --> End([Complete])
```

This read uses application authorization/policies and DB query responsibilities as labeled actions; no receipt-store mutation is involved. The report shape is proposed; basic categories are mandatory while custom ranges/exports await OQ-001. Financial totals exclude pending/declined records even if those records appear in an authorized list. Filtered totals must not be represented as complete debts without resolving OQ-009. These are proposed workflows and future verification inputs, not runtime test results.
