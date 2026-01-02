# Skill: /product-review

## Purpose

Review ProductSpec quality. Human checkpoint for approval.

**HUMAN CHECKPOINT: Human reviews ProductSpec + Quality Report.**

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `CLAUDE.md`, `project.yaml`, `glossary.yaml` |
| **Required** | Target ProductSpec (full) |
| **Required** | `snapshot/_index.yaml` (for context) |
| **Never** | TestSpecs, other products |

## Trigger

```
/product-review <spec-id> [options]
```

**Note**: Usually auto-triggered by `/product-build`.

## Options

| Option | Description |
|--------|-------------|
| `--quick` | Quick review (critical issues only) |
| `--full` | Full review (all quality gates) |
| `--approve` | Approve after review |

## Workflow Position

```
/product-build
       │
  AUTO-CHAIN
       │
       ▼
/product-review (this skill)
       │
       ▼
┌────────────────────────────┐
│ HUMAN CHECKPOINT #1        │
│ Review: ProductSpec        │
│ Approve / Request Changes  │
└────────────────────────────┘
       │
       ▼ (approved)
/test-build
```

## Workflow

### 1. Load Spec and Context

```
1. Load ProductSpec
2. Load snapshot/_index.yaml for context
3. Display summary
```

### 2. Quality Gate Checks

```
Completeness:
[ ] Has unique ID (PRD-{ver}-{feature})
[ ] Has description and goals
[ ] Has target_users

Requirements Quality:
[ ] All FR have unique IDs
[ ] All FR have testable acceptance criteria
[ ] All FR have affected repos
[ ] NFRs have measurable metrics

Technical Design:
[ ] APIs defined for cross-repo features
[ ] Data models documented
```

### 3. Generate Review Report

```yaml
review_report:
  spec_id: PRD-1.0.0-auth
  summary:
    passed: 22
    warnings: 2
    failures: 0
  status: approved  # or changes_requested
```

### 4. Approve or Request Changes

```
IF all critical issues resolved:
    Update spec.status = "approved"
    OUTPUT: "ProductSpec APPROVED"
            "Next: /test-build PRD-1.0.0-auth"

ELSE:
    Keep spec.status = "review"
    OUTPUT: "ProductSpec needs changes"
            "Run /product-build --load to address"
```

## Output

```
ProductSpec Review Complete
═══════════════════════════

ID: PRD-1.0.0-auth
Status: APPROVED

Quality Score: 95/100
  - Completeness: 100%
  - Requirements Quality: 90%
  - Technical Design: 95%

Checksum: sha256:a1b2c3d4...

─────────────────────────────
HUMAN CHECKPOINT COMPLETE

Next step:
  /test-build PRD-1.0.0-auth
```

## Integration

This skill:
- Is auto-triggered by `/product-build`
- Human checkpoint for approval
- Leads to `/test-build` after approval
