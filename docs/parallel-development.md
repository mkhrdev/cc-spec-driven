# Parallel Development Guide

## Overview

When multiple features are developed in parallel, conflicts may arise at the spec or snapshot level. This document defines strategies for handling parallel development.

## Parallel Development Scenarios

### Scenario A: Independent Features

Features that don't touch the same modules can proceed in parallel without coordination.

```
Feature A: auth module (PRD-1.0.0-auth)
Feature B: products module (PRD-1.0.0-products)
                    ↓
Both can proceed independently
                    ↓
Snapshots merge without conflict
```

### Scenario B: Same Module, Different Features

Features modifying the same module require coordination.

```
Feature A: auth/login (PRD-1.0.0-login)
Feature B: auth/logout (PRD-1.0.0-logout)
                    ↓
Must declare dependency or merge order
```

### Scenario C: Dependent Features

Feature B depends on Feature A being complete first.

```
Feature A: user model (PRD-1.0.0-user)
Feature B: auth (PRD-1.0.0-auth) depends on user
                    ↓
Feature B waits for Feature A snapshot merge
```

## Dependency Declaration

In ProductSpec, declare dependencies explicitly:

```yaml
_meta:
  id: PRD-1.0.0-auth
  # ...

dependencies:
  product_specs:
    - id: PRD-1.0.0-user
      status: required  # required | optional
      reason: "Auth depends on User model"
```

## Module Lock Strategy

### Soft Lock (Recommended)

When a feature enters test-run phase, its affected modules are "soft locked":
- Other features can still be drafted
- Warnings shown if overlapping modules
- Human decides merge order

```yaml
# In snapshot/_index.yaml
module_locks:
  - module: auth
    locked_by: PRD-1.0.0-login
    status: test_run  # draft | test_run | snapshot_pending
    locked_at: "2025-01-03T10:00:00Z"
```

### Conflict Resolution

When two features want to modify the same module:

1. **First to snapshot wins**: The first feature to reach `/snapshot-build` claims the module
2. **Second feature rebases**: Must regenerate TestSpec with updated snapshot as base
3. **Human arbitration**: If timing is close, human decides priority

## Merge Order Workflow

```
PRD-1.0.0-login (auth module) ─────────────────┐
                                               ↓
PRD-1.0.0-logout (auth module) ───► Wait ──► Rebase on new snapshot
                                               ↓
                                        Continue workflow
```

### Rebase Process

When a parallel feature completes first:

```bash
# Feature B needs to rebase
/test-build PRD-1.0.0-logout --rebase

# This will:
# 1. Load updated snapshot/_index.yaml
# 2. Regenerate TestSpec with new baseline
# 3. Mark dependency on completed feature
```

## Parallel Development Checklist

Before starting a new feature:

- [ ] Check `snapshot/_index.yaml` for active locks
- [ ] Review `spec/product/` for in-progress features
- [ ] Declare dependencies in ProductSpec if needed
- [ ] Coordinate with team on merge order if same module

## Best Practices

1. **Smaller features**: Break large features into smaller, independent specs
2. **Clear module boundaries**: Design modules with minimal overlap
3. **Communicate early**: Identify conflicts during product-review phase
4. **Sequential for high-risk**: For critical modules, avoid parallel development
5. **Version bumps**: Different features should use same version if related, different if independent

## Example: Two Features on Same Module

```yaml
# PRD-1.0.0-login.yaml
_meta:
  id: PRD-1.0.0-login
  parallel_info:
    affects_modules: [auth]
    priority: 1  # Lower number = higher priority

# PRD-1.0.0-logout.yaml
_meta:
  id: PRD-1.0.0-logout
  parallel_info:
    affects_modules: [auth]
    priority: 2
    waits_for: PRD-1.0.0-login  # Explicit wait
```

## Conflict Detection

During `/product-review`, Claude checks:

```
Conflict Detection
══════════════════

PRD-1.0.0-logout affects module: auth

WARNING: Module 'auth' is currently locked by:
  - PRD-1.0.0-login (status: test_run)

Options:
  1. Proceed (will need rebase after login merges)
  2. Wait for PRD-1.0.0-login to complete
  3. Merge features into single PRD
```
