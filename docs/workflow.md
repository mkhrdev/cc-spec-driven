# Spec-Driven Development Workflow

## Overview

Test-driven workflow: ProductSpec → TestSpec → Development → Snapshot.

## Core Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 1: PRODUCT SPECIFICATION                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  /product-build ──▶ [ProductSpec Draft]                             │
│                           │                                          │
│                      AUTO-CHAIN                                      │
│                           ▼                                          │
│  /product-review ──▶ [Quality Report]                               │
│                           │                                          │
│                           ▼                                          │
│  ┌────────────────────────────────────────┐                         │
│  │ HUMAN CHECKPOINT #1                    │                         │
│  │ Review: ProductSpec + Quality Report   │                         │
│  │ Approve / Request Changes              │                         │
│  └────────────────────────────────────────┘                         │
│                           │                                          │
└───────────────────────────┼──────────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 2: TEST SPECIFICATION                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  /test-build ──▶ [TestSpec Draft]                                   │
│                       │                                              │
│                  AUTO-CHAIN                                          │
│                       ▼                                              │
│  /test-review ──▶ [Coverage Report]                                 │
│                       │                                              │
│                       ▼                                              │
│  ┌────────────────────────────────────────┐                         │
│  │ HUMAN CHECKPOINT #2                    │                         │
│  │ Review: TestSpec Coverage              │                         │
│  │ Approve / Request Changes              │                         │
│  └────────────────────────────────────────┘                         │
│                       │                                              │
└───────────────────────┼──────────────────────────────────────────────┘
                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 3: TEST EXECUTION                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  /test-run (async execution)                                        │
│                       │                                              │
│                       ▼                                              │
│  ┌────────────────────────────────────────┐                         │
│  │ HUMAN CHECKPOINT #3                    │                         │
│  │ Review: Test Results                   │                         │
│  │ L1/L2/L3 Attribution                   │                         │
│  └────────────────────────────────────────┘                         │
│       │                                                              │
│       ├── ALL PASS ──▶ Continue to Phase 4                          │
│       │                                                              │
│       └── FAILURES ──▶ Attribution decides next step                │
│               ├── L1 (lowest): Dev fixes locally                    │
│               ├── L2 (medium): Update TestSpec                      │
│               └── L3 (highest): Update ProductSpec                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 4: SNAPSHOT UPDATE                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  /snapshot-build ──▶ [Snapshot Diff]                                │
│                           │                                          │
│                      AUTO-CHAIN                                      │
│                           ▼                                          │
│  /snapshot-review                                                   │
│                           │                                          │
│                           ▼                                          │
│  ┌────────────────────────────────────────┐                         │
│  │ HUMAN CHECKPOINT #4                    │                         │
│  │ Review: Snapshot Diff                  │                         │
│  │ Approve → Finalize / Reject → New Cycle│                         │
│  └────────────────────────────────────────┘                         │
│                           │                                          │
│                           ▼                                          │
│  ═══════════════════════════════════════════                        │
│  WORKFLOW COMPLETE - Versions Finalized                             │
│  ═══════════════════════════════════════════                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Human Checkpoints

| Checkpoint | Reviews | Actions |
|------------|---------|---------|
| #1 Product | ProductSpec + Quality Report | Approve / Request Changes |
| #2 Test | TestSpec Coverage | Approve / Request Changes |
| #3 Test Run | Test Results | L1/L2/L3 Attribution |
| #4 Snapshot | Snapshot Diff | Approve / Reject (no backtrack) |

## Version Flow

```
Draft Phase:
  PRD-1.0.0-auth-draft.1 → draft.2 → draft.3
  TEST-1.0.0-auth-draft.1 → draft.2

On /snapshot-build (after tests pass):
  PRD-1.0.0-auth-rc
  TEST-1.0.0-auth-rc

On /snapshot-review --approve:
  PRD-1.0.0-auth (final)
  TEST-1.0.0-auth (final)
```

## Test Failure Attribution

| Level | Issue Type | Impact | Action |
|-------|------------|--------|--------|
| L1 | Dev bug | Lowest | Dev fixes locally, no spec change |
| L2 | Test wrong | Medium | Update TestSpec, re-run |
| L3 | PRD wrong | Highest | Update ProductSpec, restart workflow |

## Backtracking Rules

### Before Snapshot Phase

- ProductSpec issues: Update and re-review
- TestSpec issues: Update and re-review
- Both allow iteration within phase

### During/After Snapshot Phase

- **No backtracking** to ProductSpec or TestSpec
- Reject = start new workflow cycle
- Specs are frozen at -rc suffix

## Quick Reference

| Phase | Command | Auto-Chain | Human |
|-------|---------|------------|-------|
| Product | `/product-build` | `/product-review` | YES |
| Test | `/test-build` | `/test-review` | YES |
| Execution | `/test-run` | (none) | YES (L1/L2/L3) |
| Snapshot | `/snapshot-build` | `/snapshot-review` | YES |
