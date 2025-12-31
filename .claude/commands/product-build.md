# Skill: /product-build

## Purpose

Create or update ProductSpec from requirements discussion. ProductSpec is the single source of truth.

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `AGENT.md`, `project.yaml` |
| **First** | `spec/product/index.yaml` |
| **On Demand** | Existing ProductSpec (if editing) |
| **Never** | TestSpecs, other products |

## Trigger

```
/product-build [options]
```

## Options

| Option | Description |
|--------|-------------|
| `--list` | List all existing ProductSpecs |
| `--load <id>` | Load existing ProductSpec for editing |
| `--new <feature>` | Create new ProductSpec |
| `--from-discovery` | Create from cold-start discovery |

## Workflow Position

```
/product-build (this skill)
       │
  AUTO-CHAIN
       │
       ▼
/product-review
       │
       ▼
HUMAN CHECKPOINT
```

## Workflow

### 1. Initialize

```
IF --list: Display spec list, EXIT
IF --load: Load existing spec
IF --new: Generate new ID: PRD-{version}-{feature}
```

### 2. Requirements Gathering

**Use `AskUserQuestion` tool** to confirm key decisions:
- Scope boundaries (what's in/out)
- Priority of requirements (P0-P3)
- API design choices
- Technical approach options

Core sections:

```
1. OVERVIEW
   - Description, goals, non_goals
   - Target users

2. FUNCTIONAL REQUIREMENTS (FR)
   - Title, description
   - Priority (P0-P3)
   - Acceptance criteria (testable!)
   - Affected repos

3. NON-FUNCTIONAL REQUIREMENTS (NFR)
   - Performance, security, scalability

4. TECHNICAL DESIGN (for cross-repo alignment)
   - API contracts
   - Data models / types
   - Integration points
```

### 3. Technical Design Section

**Important**: Include detailed technical specs for developer alignment:

```yaml
technical_design:
  # API contracts for cross-repo consistency
  apis:
    - endpoint: "POST /api/auth/login"
      method: POST
      request:
        body:
          email: { type: string, required: true }
          password: { type: string, required: true }
      response:
        success:
          token: { type: string }
          user: { type: User }
        errors: [invalid_credentials, user_not_found]

  # Shared data models
  models:
    - name: User
      fields:
        - { name: id, type: string }
        - { name: email, type: string }
        - { name: name, type: string }

  # Optional: State management, integration points
```

### 4. Validation

Before saving:
```
[ ] All requirements have unique IDs
[ ] All acceptance criteria are testable
[ ] At least one affected repo identified
[ ] API contracts defined for cross-repo features
```

### 5. Save & Auto-Chain

```
1. Write spec/product/PRD-{version}-{feature}.yaml
2. Update spec/product/index.yaml
3. Generate checksum
4. AUTO-CHAIN to /product-review
```

### 6. Output

```
ProductSpec Created
═══════════════════

ID: PRD-1.0.0-auth
Path: spec/product/PRD-1.0.0-auth.yaml
Status: draft

Requirements:
  Functional: 5 (P0: 1, P1: 2, P2: 2)
  Non-Functional: 2

APIs Defined: 3
Models Defined: 2

─────────────────────────────
AUTO-CHAIN: Triggering /product-review...
─────────────────────────────
```

## Integration

This skill:
- Creates the source of truth for all downstream specs
- Auto-chains to `/product-review`
- Technical design feeds into development and testing
