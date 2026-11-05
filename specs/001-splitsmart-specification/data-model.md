# Provisional Data Model

**Status**: Logical design proposal. Fields below support the documented capabilities; they are not a final schema or additional customer policies. Currency, precision, identity, access, and lifecycle decisions remain in [research.md](research.md).

| Entity | Proposed fields | Relationships and constraints |
|---|---|---|
| Account | id, identity reference | One individual identity; identity method OQ-006 |
| Group | id, display name | Participating accounts through membership; name is A1 |
| Membership | id, group id, account id, history reference | Attribution retained independently; roles, removal conditions, access after removal OQ-009 |
| Expense | id, submitter id, payer id, optional group id, amount, monetary unit reference, date, description, split method, review state | Payer is A2; selected participants need not imply group-wide visibility |
| Expense share | expense id, participant id, percentage when applicable, allocated amount | Unique participant per expense; shares sum to amount; rounding OQ-002 |
| Receipt | id, expense id, private storage reference, media metadata | Optional; retrieval authorized through expense policy; formats/limits OQ-006 |
| Review decision | id, expense id, reviewer id, outcome, decision time, optional stage reference | Eligibility and stage semantics OQ-008; retain decisions without treating each as a posting |
| Review policy reference | id, policy descriptor | Logical extension point; establishment, version binding and change behavior OQ-007/OQ-008; no fixed role or stage schema |
| Financial effect | id, source type/id, debtor id, creditor id, monetary unit, amount | Proposed unique source/component identity; posted only for approved expense or accepted payment |
| Payment | id, payer id, recipient id, amount, monetary unit, payment date, recorded-by id | Records an already-made settlement; acceptance/confirmation/correction OQ-003 |
| Notification intent | id, source expense id, intended recipient id, delivery reference/state | Other group members on creation; channels, recipient timing and visibility OQ-004/OQ-009 |
| Balance/report view | authorized participants, monetary unit, source record references, totals | Derived from financial records; not a separate source of truth |

## Review State and Posting

Proposed logical states: submitted/awaiting review, approved, and declined. Declined is a derived representation, not a confirmed rejection/resubmission lifecycle. Within review, one or multiple decisions may be required; neither the stage count nor eligible actors are fixed.

- Creation → awaiting review: retain review information; no financial effect.
- Partial review → awaiting review: no financial effect.
- Approval complete under agreed policy → approved: commit financial effects once.
- Unsuccessful review → no balance effect; subsequent transitions await OQ-007.

No automatic approval, creator exemption, majority voting, or self-approval rule is implied.

## Integrity and Open Schema Choices

Validate required expense fields, valid participant selection, equal/percentage split totals, and receipt association. Positive amounts, unique participants, and single payer are derived assumptions in the spec, subject to confirmation. Avoid binary floating-point as a proposed financial design safeguard; concrete precision and residual assignment require OQ-002.

Keep membership independent of historic participant identities. A membership change must not rewrite who incurred an obligation. Whether a change is allowed or who may then read records remains OQ-009. Record corrections and deletion behavior remain OQ-003/OQ-006; no retention period or immutable-forever policy is established.

Approval completion and posting must share an atomic transaction or equivalent integrity guarantee. A unique posting identity prevents duplicate effects during retries/concurrent review. Payment processing has the same invariant, but no recipient confirmation rule is selected. Balances must not combine distinct monetary units without an agreed conversion policy.
