# Spec-Driven Development Hub

Centralized specification repository for software products. Test-driven workflow using Claude Code.

## Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1: PRODUCT SPECIFICATION                                  │
├─────────────────────────────────────────────────────────────────┤
│  /product-build → /product-review → HUMAN CHECKPOINT #1         │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 2: TEST SPECIFICATION                                     │
├─────────────────────────────────────────────────────────────────┤
│  /test-build → /test-review → HUMAN CHECKPOINT #2               │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 3: TEST EXECUTION                                         │
├─────────────────────────────────────────────────────────────────┤
│  /test-run (async) → HUMAN CHECKPOINT #3 (L1/L2/L3)             │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 4: SNAPSHOT UPDATE                                        │
├─────────────────────────────────────────────────────────────────┤
│  /snapshot-build → /snapshot-review → HUMAN CHECKPOINT #4       │
│                                                                  │
│  ══════════════════ WORKFLOW COMPLETE ══════════════════        │
└─────────────────────────────────────────────────────────────────┘
```

## Human Checkpoints

| Checkpoint | Reviews | Decisions |
|------------|---------|-----------|
| #1 Product | ProductSpec + Quality Report | Approve / Changes |
| #2 Test | TestSpec Coverage | Approve / Changes |
| #3 Test Run | Test Results | L1/L2/L3 Attribution |
| #4 Snapshot | Snapshot Diff | Approve / Reject |

## Test Failure Attribution

| Level | Issue | Impact | Action |
|-------|-------|--------|--------|
| L1 | Dev bug | Lowest | Dev fixes locally |
| L2 | Test wrong | Medium | Update TestSpec |
| L3 | PRD wrong | Highest | Update ProductSpec |

## Getting Started

### New Products

```bash
cp -r template/ products/my-product/
vim products/my-product/project.yaml
/product-build --new my-feature
```

### Existing Projects (Cold Start)

```bash
cp -r template/ products/my-product/
vim products/my-product/cold_start_context.yaml
/cold-start --analyze
```

## Commands

| Command | Purpose | Auto-Chains To |
|---------|---------|----------------|
| `/product-build` | Create ProductSpec | `/product-review` |
| `/product-review` | Review ProductSpec | (checkpoint) |
| `/test-build` | Create TestSpec | `/test-review` |
| `/test-review` | Review TestSpec | (checkpoint) |
| `/test-run` | Execute tests | (async) |
| `/snapshot-build` | Generate diff | `/snapshot-review` |
| `/snapshot-review` | Review diff | (checkpoint) |
| `/cold-start` | Analyze existing | (none) |

## Repository Structure

```
spec/
├── CLAUDE.md                    # Config for Claude Code
├── .claude/commands/           # Skill definitions
├── docs/                       # Documentation
├── template/                   # Template for new products
└── products/                   # Product specs
    └── {product}/
        ├── project.yaml
        ├── glossary.yaml
        ├── snapshot/           # Business Snapshot
        └── spec/
            ├── product/        # PRD-*.yaml
            └── test/           # TEST-*.yaml
```

## Version Management

```
Draft:    PRD-1.0.0-auth-draft.1 → draft.2 → draft.3
RC:       PRD-1.0.0-auth-rc (after tests pass)
Final:    PRD-1.0.0-auth (after snapshot merge)
```

**Note**: x.x.x version tracks product scope, not iterations.

## Documentation

| Document | Description |
|----------|-------------|
| `docs/workflow.md` | Detailed workflow |
| `docs/revision-handling.md` | Backtrack handling |
| `docs/onboarding.md` | Setup guide |

## Key Principles

1. **ProductSpec is source of truth** - Includes API design
2. **Test-driven** - TestSpec before implementation
3. **Human judgment at checkpoints** - AI assists, human decides
4. **No backtracking after snapshot** - Issues require new cycle
