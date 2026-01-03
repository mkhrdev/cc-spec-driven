# Skill: /cold-start

## Purpose

Initialize existing project with first version of Business Snapshot.

For existing projects, start from **Snapshot v1** rather than ProductSpec.

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `CLAUDE.md`, `project.yaml` |
| **Required** | `cold_start_context.yaml` (human input) |
| **Analyze** | Target repo codebase |
| **Never** | Other products |

## Trigger

```
/cold-start [options]
```

## Options

| Option | Description |
|--------|-------------|
| `--analyze` | Analyze codebase and create Snapshot v1 |
| `--regression` | Generate baseline regression TestSpec from Snapshot v1 |
| `--repos <list>` | Specific repos to analyze |

## Cold Start Philosophy

For existing projects:
1. **Start from Snapshot**, not ProductSpec
2. Snapshot v1 captures current state
3. Future changes follow normal workflow

```
Existing Codebase
       │
       ▼
/cold-start --analyze
       │
       ▼
Snapshot v1 (current state)
       │
       ▼
New feature? → /product-build → normal workflow
```

## Prerequisites

Create `cold_start_context.yaml`:

```yaml
project_overview: |
  What does this project do?
  Who are the users?

core_flows: |
  Main user journeys

glossary_seeds: |
  Domain-specific terms
```

## Workflow

### 1. Analyze Codebase

```
For each repo:
├── Directory structure
├── Framework detection
├── Route/endpoint mapping
├── Database schema
└── Integration points
```

### 2. Generate Snapshot v1

Create initial snapshot from discovered features:

```yaml
# snapshot/_index.yaml
_meta:
  version: 1
  source: cold_start
  completeness: partial

one_liner: "{Project summary}"

modules:
  - id: auth
    name: "User Authentication"
    summary: "Existing auth system"
    feature_count: 3
```

### 3. Generate Module Files

```yaml
# snapshot/modules/auth.yaml
_meta:
  id: auth
  source: cold_start

features:
  - id: login
    name: "User Login"
    status: active
    details:
      apis:
        - endpoint: "POST /api/auth/login"
      # Captured from codebase analysis
```

### 4. Output

```
Cold Start Complete
═══════════════════

Analyzed: 2 repos
Snapshot Version: 1

Modules Created: 4
  - auth (3 features)
  - products (5 features)
  - orders (4 features)
  - users (2 features)

APIs Discovered: 15
Models Discovered: 8

─────────────────────────────
Snapshot v1 created.

For new features, run:
  /product-build --new <feature>
```

## Cold Start Regression Test

**IMPORTANT**: After Snapshot v1, generate baseline regression tests to validate existing functionality.

### Trigger

```bash
/cold-start --regression
```

### Input Sources

Regression tests are generated from **combined sources**:

```
cold_start_context.yaml (human description)
           +
Snapshot v1 (code analysis result)
           ↓
TEST-0.0.0-baseline.yaml (full regression coverage)
```

### Generated TestSpec

```yaml
# TEST-0.0.0-baseline.yaml
_meta:
  id: TEST-0.0.0-baseline
  title: "Cold Start Baseline Regression Suite"
  status: draft
  source: cold_start  # NOT from ProductSpec

generated_from:
  snapshot:
    version: 1
    checksum: sha256:xxx
  context: cold_start_context.yaml
  product: null  # No ProductSpec for baseline

test_suites:
  - id: TS-REG-auth
    name: "Auth Module Regression"
    type: regression
    covers:
      modules: [auth]
    test_cases:
      - id: TC-REG-001
        title: "Login endpoint responds correctly"
        source: snapshot/modules/auth.yaml#login
```

### Why Full Coverage?

- All existing functionality becomes "regression scope"
- Tests validate **current behavior** as baseline
- Future changes measured against this baseline
- No ProductSpec exists for legacy features

### Execution

```bash
# Run baseline tests
/test-run TEST-0.0.0-baseline

# Expected: ALL PASS (describing current behavior)
# If FAIL: code bug OR test generation issue
```

## Next Steps After Cold Start

1. Review `snapshot/_index.yaml`
2. Adjust modules if needed
3. Run `/cold-start --regression` to generate baseline tests
4. Execute baseline tests to validate
5. For new features: `/product-build --new <feature>`
6. Follow normal workflow

## Discovery Report (Optional)

If detailed analysis needed:

```yaml
# discovery_report.yaml
detected_features:
  - id: DF-001
    name: "User Authentication"
    evidence:
      - path: "src/auth/"
    confidence: high

questions_for_human:
  - "Found admin/ but not mentioned. Include?"
```

## Integration

This skill:
- Creates Snapshot v1 from existing code
- No ProductSpec needed for initial state
- Future changes use normal workflow
