# CLAUDE.md - Spec-Driven Development Framework

## Overview

This repo is a **Specification Hub** for software products. Test-driven workflow: ProductSpec → TestSpec → Development → Snapshot.

**Core Rules**:
1. ProductSpec is single source of truth (includes API design)
2. Test specs created before implementation (test-driven)
3. Human checkpoints after each build-review pair
4. No backtracking after snapshot phase

---

## Workflow

```
/product-build → /product-review → HUMAN CHECK #1
                                          ↓
/test-build → /test-review → HUMAN CHECK #2
                                    ↓
/test-run (async) → HUMAN CHECK #3 (L1/L2/L3) → tests pass
                                                     ↓
/snapshot-build → /snapshot-review → HUMAN CHECK #4 → END
```

### Skills

| Skill | Auto-Chains To | Human After |
|-------|----------------|-------------|
| `/product-build` | `/product-review` | YES |
| `/test-build` | `/test-review` | YES |
| `/test-run` | (none) | YES (L1/L2/L3 attribution) |
| `/snapshot-build` | `/snapshot-review` | YES |
| `/cold-start` | (none) | YES |

---

## Version Management

### Spec Version

```
PRD-{ver}[-feature][-suffix]

Suffixes:
  -draft.N   During build/review
  -rc        Release candidate (after test pass)
  (none)     Final approved
```

**Key**: x.x.x version tracks **product scope**, not iterations.

### Version Flow

```
Draft: PRD-1.0.0-auth-draft.1 → draft.2 → draft.3
After test pass: PRD-1.0.0-auth-rc
After snapshot merge: PRD-1.0.0-auth (final)
```

---

## Repository Structure

```
spec/
├── CLAUDE.md
├── .claude/commands/        # Skills
├── docs/                    # Documentation
├── template/                # Copy for new products
└── products/
    └── {product}/
        ├── project.yaml
        ├── glossary.yaml
        ├── snapshot/
        │   ├── _index.yaml
        │   ├── modules/
        │   └── changelog.yaml
        └── spec/
            ├── product/     # PRD-*.yaml
            └── test/        # TEST-*.yaml
```

---

## Context Loading

**Always Load**: `CLAUDE.md`, `project.yaml`, `snapshot/_index.yaml`

**Never Load**: `template/`, `docs/`, other products

---

## Test Failure Handling

When tests fail, human decides (ordered by impact):

| Level | Issue | Action |
|-------|-------|--------|
| L1 (lowest) | Dev bug | Dev fixes locally, no spec change |
| L2 (medium) | Test wrong | Update TestSpec, re-run |
| L3 (highest) | PRD wrong | Update ProductSpec, restart workflow |

---

## Cold Start (Existing Projects)

1. Create `cold_start_context.yaml`
2. Run `/cold-start --analyze`
3. Creates Snapshot v1 from existing code
4. New features follow normal workflow

---

## ProductSpec Technical Design

ProductSpec includes API design for cross-repo alignment:

```yaml
technical_design:
  apis:
    - endpoint: "POST /api/auth/login"
      request: { email: string, password: string }
      response: { token: string, user: User }

  models:
    - name: User
      fields: [id, email, name]
```

This feeds into Snapshot and development.

---

## Documentation

| Doc | Purpose |
|-----|---------|
| [docs/workflow.md](docs/workflow.md) | Full workflow |
| [docs/revision-handling.md](docs/revision-handling.md) | Backtrack handling |
| [docs/onboarding.md](docs/onboarding.md) | Setup guide |
