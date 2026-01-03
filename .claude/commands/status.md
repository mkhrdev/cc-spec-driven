# Skill: /status

## Purpose

Display current workflow status for a product. Shows where each spec is in the workflow pipeline.

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `CLAUDE.md`, `project.yaml` |
| **Required** | `snapshot/_index.yaml` |
| **Scan** | `spec/product/*.yaml`, `spec/test/*.yaml` |
| **Never** | Other products |

## Trigger

```
/status [product-name]
```

## Options

| Option | Description |
|--------|-------------|
| `--all` | Show all specs (including completed) |
| `--active` | Show only in-progress specs (default) |

## Output

```
Workflow Status: {product-name}
════════════════════════════════

Snapshot Version: 3
Last Updated: 2025-01-03

ACTIVE SPECS
────────────────────────────────
PRD-1.0.0-auth
  Status: [■■■■□] test-run
  Phase:  Product ✓ → Test ✓ → Execution ● → Snapshot □
  TestSpec: TEST-1.0.0-auth (approved)

PRD-1.0.0-payments
  Status: [■■□□□] product-review
  Phase:  Product ● → Test □ → Execution □ → Snapshot □
  Awaiting: Human approval

PARALLEL DEVELOPMENT
────────────────────────────────
Module Locks:
  - auth: locked by PRD-1.0.0-auth (test_run)

Conflicts: None

RECENT COMPLETIONS
────────────────────────────────
PRD-0.9.0-users → Snapshot v3 (2025-01-01)
```

## Status Indicators

| Symbol | Meaning |
|--------|---------|
| ✓ | Phase completed |
| ● | Currently in this phase |
| □ | Not yet reached |
| ⛔ | Blocked (needs human intervention) |

## Workflow Phases

```
[■□□□□] product-build/review
[■■□□□] test-build/review
[■■■□□] test-run (external execution)
[■■■■□] snapshot-build/review
[■■■■■] completed
```

## Integration

This skill:
- Read-only, no modifications
- Quick overview of all active work
- Helps identify bottlenecks and conflicts
