# Skill: /snapshot-build

## Purpose

Generate Business Snapshot diff after tests pass. This skill:
1. Promotes specs from suffix versions to release-candidate
2. Generates snapshot diff for review
3. Does NOT backtrack to ProductSpec/TestSpec

**Triggered after test-run completes successfully.**

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `CLAUDE.md`, `project.yaml` |
| **Required** | ProductSpec, TestSpec, `snapshot/_index.yaml` |
| **On Demand** | `snapshot/modules/{id}.yaml` |
| **Never** | Other products |

## Trigger

```
/snapshot-build <product-spec-id> [options]
```

## Options

| Option | Description |
|--------|-------------|
| `--preview` | Preview diff without saving |

## Workflow Position

```
/test-run (async, all pass)
       │
       ▼
/snapshot-build (this skill)
       │
  AUTO-CHAIN
       │
       ▼
/snapshot-review
       │
       ▼
HUMAN CHECKPOINT → Merge → VERSIONS FINALIZED
```

## Version Management

### On snapshot-build Trigger

When human triggers `/snapshot-build`:
1. ProductSpec and TestSpec **suffix versions become release-candidate**
2. Example: `PRD-1.0.0-auth-draft.3` → `PRD-1.0.0-auth-rc`

### On snapshot-review Approval

After snapshot merge:
1. All specs become **final versions** (no suffix)
2. Example: `PRD-1.0.0-auth-rc` → `PRD-1.0.0-auth`

### Version Flow

```
Draft Phase:
  PRD-1.0.0-auth-draft.1 → draft.2 → draft.3
  TEST-1.0.0-auth-draft.1 → draft.2

On /snapshot-build:
  PRD-1.0.0-auth-rc
  TEST-1.0.0-auth-rc

On /snapshot-review --approve:
  PRD-1.0.0-auth (final)
  TEST-1.0.0-auth (final)
```

**Note**: The x.x.x version number (1.0.0) tracks product scope and does NOT change during a single workflow cycle.

## Workflow

### 1. Verify Prerequisites

```
1. Load ProductSpec (approved)
2. Verify tests passed
3. Promote specs to -rc suffix
```

### 2. Build Requirement-to-Module Mapping

Map ProductSpec requirements (FR/NFR) to Snapshot structure (module/feature):

```yaml
# Generated mapping table
requirement_mapping:
  FR-001:
    module: auth
    feature: login
  FR-002:
    module: auth
    feature: logout
  NFR-001:
    modules: [auth, users]  # Cross-cutting
```

### 3. Generate Snapshot Diff

```yaml
snapshot_diff:
  spec_id: PRD-1.0.0-auth
  current_version: 3
  proposed_version: 4

  # Traceability mapping
  requirement_mapping:
    FR-001: { module: auth, feature: login }
    FR-002: { module: auth, feature: logout }

  changes:
    modules:
      added:
        - id: auth
          name: "User Authentication"
          feature_count: 3
          source_requirements: [FR-001, FR-002]

    features:
      added:
        - module: auth
          id: login
          name: "User Login"
          source_requirements: [FR-001]
          details:
            apis:
              - endpoint: "POST /api/auth/login"
            forms:
              - name: login_form
```

### 4. Include Technical Details

Snapshot captures technical design from ProductSpec:

```yaml
features:
  - id: login
    details:
      apis:
        - endpoint: "POST /api/auth/login"
          method: POST
          request: { email: string, password: string }
          response: { token: string, user: User }

      models:
        - name: User
          fields: [id, email, name]

      rules:
        - "Failed login locked after 5 tries"
```

### 5. Save & Auto-Chain

```
1. Save diff to snapshot/pending/
2. AUTO-CHAIN to /snapshot-review
```

### 6. Output

```
Snapshot Diff Generated
═══════════════════════

Source: PRD-1.0.0-auth
Tests: PASSED

Version Promotion:
  ProductSpec: draft.3 → rc
  TestSpec: draft.2 → rc

Changes:
  +1 new module (auth)
  +3 new features
  +3 API endpoints
  +2 data models

─────────────────────────────
AUTO-CHAIN: Triggering /snapshot-review...
─────────────────────────────
```

## No Backtracking

At this phase:
- ProductSpec and TestSpec are **frozen**
- Issues found require **new workflow cycle**
- Snapshot review only accepts or rejects the diff

## Integration

This skill:
- Requires test-run to pass
- Promotes specs to release-candidate
- Auto-chains to `/snapshot-review`
- Final versions set after snapshot merge
