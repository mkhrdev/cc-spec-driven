# Revision & Backtrack Handling

## Version Concepts

### Spec Version Format

```
PRD-{ver}[-feature][-suffix]

Suffixes:
  -draft.N   During build/review (before approval)
  -rc        Release candidate (after tests pass)
  (none)     Final approved version
```

**Key Rule**: x.x.x version tracks **product scope**, not iterations.

### Semantic Versioning Reference

**Note**: Version numbers are **decided by humans**. This workflow does not determine or change versions - it only aligns with human decisions.

| Change Type | Version Bump | Examples |
|-------------|--------------|----------|
| **MAJOR** (x.0.0) | Breaking changes | Remove feature, API incompatible change, data model restructure |
| **MINOR** (0.x.0) | New functionality | Add new module, new API endpoint, new feature |
| **PATCH** (0.0.x) | Improvements | Bug fix, performance optimization, clarification |

### Version Guidance (for human reference)

```
Breaking change? → Consider MAJOR bump
New capability?  → Consider MINOR bump
Fix/improve?     → Consider PATCH bump
```

### Multi-Feature Version Patterns

- **Related features**: Same MAJOR.MINOR, different feature suffix
  - `PRD-1.0.0-login`, `PRD-1.0.0-logout` (both part of auth 1.0)
- **Independent features**: Can have different versions
  - `PRD-1.0.0-auth`, `PRD-2.0.0-payments` (different product areas)
- **Dependent features**: Child inherits parent's MAJOR version
  - `PRD-1.0.0-user` → `PRD-1.0.0-auth` (auth depends on user)

## Test Failure Attribution

When tests fail, human attributes to one of three levels (ordered by impact):

| Level | Issue Type | Impact | Action |
|-------|------------|--------|--------|
| **L1** | Dev bug | Lowest | Dev fixes locally, no spec change |
| **L2** | Test wrong | Medium | Update TestSpec, re-run tests |
| **L3** | PRD wrong | Highest | Update ProductSpec, restart workflow |

### L1: Development Issue (Lowest Impact)

```
Test fails because implementation has bug.
Specs are correct.

Action:
  - Developer fixes code locally
  - No changes to this repo
  - Re-run tests
```

### L2: Test Issue (Medium Impact)

```
Test fails because test case is wrong.
ProductSpec is correct.

Action:
  - Update TestSpec
  - Re-run /test-review
  - Re-run tests
```

### L3: Requirement Issue (Highest Impact)

```
Test fails because ProductSpec is unclear/wrong.

Action:
  - Update ProductSpec
  - Restart workflow from /product-build
  - Regenerate TestSpec
```

## Backtracking Rules

### Before Snapshot Phase

Backtracking is allowed within phases:

```
ProductSpec Phase:
  PRD-draft.1 → review → changes requested
  PRD-draft.2 → review → approved

TestSpec Phase:
  TEST-draft.1 → review → changes requested
  TEST-draft.2 → review → approved
```

### During/After Snapshot Phase

**No backtracking allowed**:

```
/snapshot-build triggered
       ↓
PRD-1.0.0-auth-rc (frozen)
TEST-1.0.0-auth-rc (frozen)
       ↓
/snapshot-review
       ↓
   ┌───────────────────────────────┐
   │ Approve: Finalize versions    │
   │ Reject: Start NEW cycle       │
   └───────────────────────────────┘
```

## Revision Scenarios

### A: ProductSpec Review Issues

```
PRD-1.0.0-auth-draft.1
       ↓ review finds issues
PRD-1.0.0-auth-draft.2
       ↓ review passes
PRD-1.0.0-auth (approved, awaiting tests)
```

Version unchanged. Only suffix changes.

### B: TestSpec Review Issues

```
TEST-1.0.0-auth-draft.1
       ↓ review finds coverage gap
TEST-1.0.0-auth-draft.2
       ↓ review passes
TEST-1.0.0-auth (approved)
```

### C: Test Execution Failures

```
Tests run → TC-003 fails

Human attribution:
  L1 → Dev fixes, re-run
  L2 → Update TEST, re-run
  L3 → Update PRD, restart workflow
```

### D: Snapshot Rejection

```
/snapshot-review
       ↓
Human: "Reject - PRD needs clarification"
       ↓
Start new workflow cycle:
  /product-build --load PRD-1.0.0-auth
  (continues as new draft)
```

## Changelog Tracking

Every change tracked in spec changelog:

```yaml
changelog:
  - version: "1.0.0"
    iteration: 3
    status: draft
    changes:
      - type: revised
        description: "Clarified acceptance criteria"
        trigger:
          type: human_feedback  # or review_finding, test_failure
          content: "Original text of feedback"
        sections_modified:
          - "requirements.functional[FR-002]"
```

## Quick Reference

| Phase | Can Backtrack? | Action on Issue |
|-------|----------------|-----------------|
| Product Build/Review | YES | Update, re-review |
| Test Build/Review | YES | Update, re-review |
| Test Execution | DEPENDS | L1-L3 attribution |
| Snapshot Build/Review | NO | Reject = new cycle |
