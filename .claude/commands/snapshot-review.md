# Skill: /snapshot-review

## Purpose

Review and approve Business Snapshot diff. Final step of workflow cycle.

**HUMAN CHECKPOINT: Review snapshot diff and finalize versions.**

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `AGENT.md`, `project.yaml` |
| **Required** | Pending snapshot diff, `snapshot/_index.yaml` |
| **On Demand** | `snapshot/modules/{id}.yaml` (affected modules only) |
| **Never** | Other products, unaffected modules |

## Trigger

```
/snapshot-review [options]
```

**Note**: Usually auto-triggered by `/snapshot-build`.

## Options

| Option | Description |
|--------|-------------|
| `--approve` | Approve and apply diff |
| `--reject` | Reject diff (requires new cycle) |

## Workflow Position

```
/snapshot-build
       │
  AUTO-CHAIN
       │
       ▼
/snapshot-review (this skill)
       │
       ▼
┌────────────────────────────┐
│ HUMAN CHECKPOINT #4        │
│ Review: Snapshot Diff      │
│ Approve → Finalize         │
│ Reject → New Cycle         │
└────────────────────────────┘
       │
       ▼
═══════════════════════════
WORKFLOW COMPLETE
═══════════════════════════
```

## Workflow

### 1. Load Context

```
1. Load pending diff
2. Load snapshot/_index.yaml
3. Load ONLY affected modules from diff:
   - New modules: none to load
   - Modified modules: load current state
4. Do NOT load unaffected modules
```

### 2. Present Diff

```
SNAPSHOT DIFF REVIEW
════════════════════

Source: PRD-1.0.0-auth
Current Version: 3
Proposed Version: 4

Affected Scope:
  New: auth
  Modified: (none)
  Unchanged: users, products, orders (not loaded)

MODULES
  [+] auth (3 features)

FEATURES
  [+] auth/login
      APIs: POST /api/auth/login
      Forms: login_form

─────────────────────────────
Options:
  [A] Approve - Finalize versions
  [R] Reject - Requires new workflow cycle
```

### 3. On Approve

```
1. Apply snapshot diff (only affected modules)
2. Finalize spec versions:
   PRD-1.0.0-auth-rc → PRD-1.0.0-auth
   TEST-1.0.0-auth-rc → TEST-1.0.0-auth
3. Update changelog
```

### 4. On Reject

```
Rejection requires new workflow cycle:
  - ProductSpec issue → /product-build
  - TestSpec issue → /test-build
  - Both preserved for reference
```

## Output (Approved)

```
SNAPSHOT UPDATED
════════════════

Source: PRD-1.0.0-auth
Version: 3 → 4

Versions Finalized:
  ✓ PRD-1.0.0-auth (was -rc)
  ✓ TEST-1.0.0-auth (was -rc)

Changes Applied:
  +1 module (auth)
  +3 features
  +3 API endpoints

═══════════════════════════════════════
WORKFLOW COMPLETE: PRD-1.0.0-auth
═══════════════════════════════════════
```

## Output (Rejected)

```
SNAPSHOT REJECTED
═════════════════

Reason: {human provided reason}

Snapshot NOT updated.
Specs remain at -rc suffix.

To fix:
  1. Identify issue source
  2. Start new workflow cycle
```

## No Backtracking

At this phase:
- Cannot modify ProductSpec or TestSpec
- Reject means **start new cycle**
- This is the final gate

## Integration

This skill:
- Only loads affected modules (not full snapshot)
- Final human checkpoint
- Finalizes all spec versions on approval
