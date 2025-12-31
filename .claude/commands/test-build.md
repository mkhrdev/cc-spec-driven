# Skill: /test-build

## Purpose

Generate TestSpec from approved ProductSpec. Test-driven: can be created before implementation.

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `AGENT.md`, `project.yaml` |
| **Required** | ProductSpec (approved), `spec/test/index.yaml` |
| **Required** | `snapshot/_index.yaml` (impact analysis) |
| **On Demand** | `snapshot/modules/{id}.yaml` (affected modules only) |
| **On Demand** | Existing TestSpec (if regenerating) |
| **Never** | Other products |

## Trigger

```
/test-build <product-spec-id> [options]
```

## Options

| Option | Description |
|--------|-------------|
| `--list` | List all TestSpecs |
| `--load <id>` | Load existing TestSpec |
| `--regenerate` | Regenerate from updated ProductSpec |

## Workflow Position

```
ProductSpec Approved
       │
       ▼
/test-build (this skill)
       │
  AUTO-CHAIN
       │
       ▼
/test-review
       │
       ▼
HUMAN CHECKPOINT
```

## Workflow

### 1. Load Context

```
1. Load ProductSpec, verify approved
2. Load snapshot/_index.yaml
3. Identify affected modules from ProductSpec
4. Load only affected snapshot/modules/{id}.yaml
```

### 2. Impact Analysis

Reference snapshot to determine scope:

```yaml
impact_analysis:
  new_module: auth          # Not in snapshot
  extends_modules: []       # Existing modules being extended
  depends_on: [users]       # Modules this feature uses

  regression_scope:
    - module: users         # May be affected
      reason: "auth depends on user data"
```

### 3. Build Coverage Matrix

```yaml
coverage_matrix:
  # New feature tests
  FR-001:
    title: "User Login"
    acceptance_criteria:
      - criterion: "User can login"
        generated_cases: [TC-001, TC-002]

  # Regression tests (from impact analysis)
  regression:
    - module: users
      test_cases: [TC-R01, TC-R02]
      reason: "Verify user data access still works"
```

### 4. Generate Test Suites

```yaml
test_suites:
  # Feature tests
  - id: TS-001
    name: "Authentication E2E"
    type: e2e
    test_cases: [...]

  # Regression suite
  - id: TS-REG
    name: "Regression Tests"
    type: regression
    scope: [users]
    test_cases:
      - id: TC-R01
        title: "User profile still accessible"
        tags: [regression, users]
```

### 5. Save & Auto-Chain

```
1. Write spec/test/TEST-{version}-{feature}.yaml
2. Update spec/test/index.yaml
3. Generate checksum
4. AUTO-CHAIN to /test-review
```

### 6. Output

```
TestSpec Generated
══════════════════

ID: TEST-1.0.0-auth
Source: PRD-1.0.0-auth (sha256:a1b2...)

Impact Analysis:
  New module: auth
  Regression scope: users

Coverage: 100%
  Feature Tests: 25
  Regression Tests: 6

─────────────────────────────
AUTO-CHAIN: Triggering /test-review...
─────────────────────────────
```

## Integration

This skill:
- References snapshot for impact analysis
- Generates regression tests for affected modules
- Auto-chains to `/test-review`
