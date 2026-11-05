# Architecture Validation Guide

This repository contains requirements and design documents, not an implemented application. No install, launch, migration, or automated application-test command exists yet. Stack selection must supply those commands before implementation validation can be run; no command or passing result is invented here.

## Prerequisites

Read [spec.md](spec.md), [plan.md](plan.md), [data-model.md](data-model.md), and [contracts](contracts/application-boundaries.md). Prepare three account identities, a sample group, and explicit test approval/access policies. These policies are fixtures only; production rules require OQ-008/OQ-009 answers. Monetary test cases use whole units and do not settle currency or rounding.

## Design Walkthrough Now; Executable Acceptance Later

1. Map all nine original feature headings to SC-001. Check that no original feature was dropped and optional custom reporting was not made mandatory.
2. Submit a 90-unit expense paid by Alice and split equally among Alice, Bob, and Cara. Review record exists, each share is 30, balances remain unchanged, and other group members have new-expense notice intents.
3. Exercise a single-review fixture and a multi-level fixture separately. After partial review, balances remain unchanged. Once the fixture policy is satisfied, Bob owes Alice 30 and Cara owes Alice 30. Neither fixture establishes production workflow requirements.
4. Repeat/concurrently complete approval. Inspect proposed transaction and source-uniqueness boundaries: each share must post once. Unsuccessful or pending review produces no balance effect.
5. Submit 100 split 50/30/20 percent; expect matching shares. Invalid totals and missing required fields do not post. Test fractional amounts only after OQ-002 is settled.
6. Record an accepted 10-unit payment from Bob to Alice, then 20; expect 20 then zero outstanding. Retry each recorded payment; totals stay unchanged. Payment authorization/confirmation requires OQ-003.
7. Walk permitted and denied group actions and reads, including receipts/reports and former members, against an explicit policy matrix. Do not assume former members lose or retain access. Preserve original financial attribution after any permitted membership change.
8. Inspect optional receipt handling, submission failures, empty reports, and notification delivery failures. Reports reconcile with approved expenses and accepted payments; pending expenses remain distinguishable.
9. After implementation, repeat primary journeys on web, Android, and iOS with the agreed environment matrix, then reopen accounts and compare persisted totals. Install/run/test commands and performance checks must be added after stack and OQ-005 decisions.

## Readiness Result

Current validation is a document walkthrough, not runtime proof. Q1/Q2 clarifications are integrated; OQ-001–009 and technical selections remain visible. Final reviewer tests, membership tests, monetary cases, and platform execution remain conditional. Return to the specification/checklist when policies are settled before treating this plan as implementation-ready.
