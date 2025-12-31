# Skill: /cold-start

## Purpose

Initialize existing project with first version of Business Snapshot.

For existing projects, start from **Snapshot v1** rather than ProductSpec.

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `AGENT.md`, `project.yaml` |
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
| `--analyze` | Analyze codebase |
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

## Next Steps After Cold Start

1. Review `snapshot/_index.yaml`
2. Adjust modules if needed
3. For new features: `/product-build --new <feature>`
4. Follow normal workflow

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
