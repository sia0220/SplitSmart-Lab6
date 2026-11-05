# Architecture Decision Record and Open Decisions

**Date**: 2026-11-05
**Evidence**: [original requirements](../../UserRequirement.md), [specification](spec.md), and the user's Q1/Q2 answers. This is local source analysis; no external product or technology research was needed for these conceptual boundaries.

## Decisions Proposed for Planning

| Decision | Rationale | Alternatives considered |
|---|---|---|
| Shared backend with logical modules | Common rules across web, Android, and iOS | Rules duplicated per client; independently deployed services |
| Separate review records from financial effects | Source requires approval before balance impact | Posting on submission; clients filtering already-posted entries |
| Approval policy interface | Leaves single review and multiple levels possible | Hard-coded single reviewer; mandatory multi-level configuration |
| Authorization policy interface | Central access enforcement without inventing roles | Creator-as-admin; membership grants universal visibility |
| Transactional financial records with source uniqueness | Proposed means of preventing duplicate/partial effects | Client-only retry suppression; independent balance writes |
| Private receipt storage with authorized retrieval | Optional image association without public disclosure | Public receipt URLs; embedding images in financial records |
| Persist notification intent on successful submission | Separate creation notice from later approval and delivery | Coupling expense creation to a particular delivery provider |

These proposals are implementation options, not additions to the original requirements. No dependency, language, provider, deployment, or version has been selected.

## Unresolved Decisions and Impact

| Questions | Decisions remaining | Work that can proceed | Work conditional on answers |
|---|---|---|---|
| OQ-008 | Single/multiple review needs, reviewers, self-approval, completion | Review record and policy boundary design | Eligible reviewers, stage transitions, final review tests |
| OQ-009 | Membership management, removal restrictions, former-member visibility | Membership/history separation and access boundary | Role matrix, removal workflow, former-member tests |
| OQ-002 | Currency, precision, rounding | Exact allocation invariants | Monetary schema precision and residual allocation |
| OQ-003 | Payment permissions/confirmation, overpayments, corrections | Payment record boundary and unique-effect invariant | Settlement transitions and corrections |
| OQ-004 | Notification channel/timing/failure handling | Submission notice intent | Delivery implementation and timing tests |
| OQ-005 | Supported versions and quality targets | Shared journey validation design | Environment matrix, capacity and recovery acceptance |
| OQ-006 | Identity/lifecycle and receipt constraints | Account and receipt boundaries | Provider, deletion/retention, upload validation |
| OQ-007 | Review lifecycle and policy changes | Recording decisions separately | Policy versioning, resubmission, delegation |
| OQ-001 | Custom report scope | Basic summaries | Filters and exports beyond basic reporting |
| Technical selection | Stack, tooling, hosting, storage vendors | Logical architecture | Runnable setup and build commands |

No research can substitute for customer answers to policy questions. These decisions are deliberately unresolved. Phase 0 and Phase 1 are provisional; the default planning requirement that all unknowns be resolved is not satisfied. Continuing independent planning follows the user's explicit direction and carries possible rework when those decisions are settled.
