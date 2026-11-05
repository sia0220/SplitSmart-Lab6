# Provisional Application Boundary Contracts

These logical operations serve web, Android, and iOS; no HTTP routes, framework, or serialization format is selected. Every operation receives an authenticated actor and applies an agreed action/resource authorization policy. This does not define which actors are granted access.

| Operation | Input | Successful result and invariant | Unresolved rules |
|---|---|---|---|
| Create/access account | Identity credentials/reference | Personal account and authenticated context | Identity, recovery, closure OQ-006 |
| Create/manage group | Group details, proposed membership change | Persisted group/change when permitted | Management roles/removal OQ-009 |
| Submit expense | Amount, date, description, participants, split, payer, optional group/receipt | Reviewable expense; no balance posting; notification intent for other group members | Monetary rules, individual review OQ-002/OQ-008 |
| Read expense/receipt | Record reference | Authorized details or image | Current/former-member visibility OQ-009 |
| Record review | Expense reference, actor decision, concurrency/retry reference | Decision recorded; post once only if approval complete | Eligibility, self-approval, levels OQ-008 |
| Read balances | Authorized participant/group context | Pairwise outstanding amounts from approved expenses and accepted payments | Currency/netting A3/OQ-002; visibility OQ-009 |
| Record payment | Payer, recipient, amount, payment date, retry reference | Persisted settlement; apply balance effect once when accepted | Permissions, confirmation, overpayment OQ-003 |
| Read reports | Authorized record context | Expense/balance/payment summaries reconcile with source records | Custom ranges/exports OQ-001 |
| Deliver notice | Expense creation event and recipient reference | Notice of new expense, without implying approval | Channel, deadlines, failures, detail visibility OQ-004/OQ-009 |

## Shared Failure Contract

Invalid input identifies corrections and produces no financial effect. Unauthorized reads disclose no protected record details; unauthorized changes do not mutate records. Storage/receipt failures must not be presented as successful submissions. Repeated financial commands preserve their original single effect. Conflicting edits require an agreed resolution policy under OQ-003; no last-write-wins behavior is prescribed.

## Policy Boundary Shapes

- Authorization: evaluate actor, action, resource, and relevant context; return a decision under an explicitly agreed policy. Role names and grants remain absent.
- Approval: evaluate eligibility and review completion from expense context, decisions, and applicable policy. A single decision need not complete review; self-approval is not presumed allowed or prohibited.

Absent an agreed policy, policy-dependent operations are not production-ready. Test policies must be labeled fixtures rather than promoted to customer defaults. These contracts remain conditional until permission, currency, and lifecycle decisions are finalized.
