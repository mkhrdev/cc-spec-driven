# Skill: /test-review

## Purpose

Review TestSpec quality and coverage before execution.

**HUMAN CHECKPOINT: Human reviews TestSpec quality report.**

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `CLAUDE.md`, `project.yaml` |
| **Required** | TestSpec (draft), Source ProductSpec |
| **Required** | `snapshot/_index.yaml` (regression validation) |
| **On Demand** | `snapshot/modules/{id}.yaml` (affected modules only) |
| **Never** | Other products |

## Trigger

```
/test-review <test-spec-id> [options]
```

**Note**: Usually auto-triggered by `/test-build`.

## Options

| Option | Description |
|--------|-------------|
| `--quick` | Quick review (coverage only) |
| `--full` | Full review (all gates) |
| `--approve` | Approve after review |

## Workflow Position

```
/test-build
       │
  AUTO-CHAIN
       │
       ▼
/test-review (this skill)
       │
       ▼
┌────────────────────────────┐
│ HUMAN CHECKPOINT           │
│ Review: TestSpec Quality   │
│ Approve / Request Changes  │
└────────────────────────────┘
       │
       ▼ (approved)
/test-run (async)
```

## Workflow

### 1. Load Specs

```
1. Load TestSpec
2. Load ProductSpec
3. Verify ProductSpec checksum matches
4. Load snapshot/_index.yaml
5. Load affected snapshot/modules/{id}.yaml
```

### 2. Coverage Analysis

```yaml
coverage_report:
  product_spec: PRD-1.0.0-auth
  test_spec: TEST-1.0.0-auth

  # Feature coverage
  requirements:
    - id: FR-001
      acceptance_criteria:
        - criterion: "User can login"
          covered_by: [TC-001, TC-002]
          status: covered

  # Regression coverage (from snapshot)
  regression:
    - module: users
      reason: "auth depends on user data"
      test_cases: [TC-R01, TC-R02]
      status: covered
```

### 3. Quality Gate Checks

```
Feature Tests:
[ ] Every FR has test cases
[ ] Every acceptance criterion covered
[ ] Test cases have clear steps

Regression Tests:
[ ] Impact analysis modules have regression tests
[ ] Regression scope matches snapshot dependencies
```

### 4. Generate Review Report

```yaml
review_report:
  test_spec_id: TEST-1.0.0-auth

  feature_coverage: 100%
  regression_coverage: 100%

  gaps: []
  status: approved
```

### 5. Approve or Request Changes

```
IF all issues resolved:
    Update TestSpec.status = "approved"
    OUTPUT: "TestSpec APPROVED"
            "Ready for /test-run"

ELSE:
    Keep TestSpec.status = "review"
    OUTPUT: "TestSpec needs changes"
```

## Output

```
TestSpec Review Complete
════════════════════════

ID: TEST-1.0.0-auth
Source: PRD-1.0.0-auth
Status: APPROVED

Coverage:
  Feature: 100% (25 cases)
  Regression: 100% (6 cases)

─────────────────────────────
HUMAN CHECKPOINT COMPLETE

Next step:
  /test-run TEST-1.0.0-auth
```

## Integration

This skill:
- References snapshot for regression validation
- Verifies both feature and regression coverage
- Leads to `/test-run` after approval
