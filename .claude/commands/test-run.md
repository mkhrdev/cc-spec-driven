# Skill: /test-run

## Purpose

Execute tests from TestSpec asynchronously. This runs tests in the background and returns immediately.

## Context Requirements

| Load | Files |
|------|-------|
| **Always** | `AGENT.md`, `project.yaml` |
| **Required** | TestSpec (approved) |
| **Never** | ProductSpec details (checksum only) |

## Trigger

```
/test-run <test-spec-id> [options]
```

## Options

| Option | Description |
|--------|-------------|
| `--suite <id>` | Run specific test suite |
| `--tags <tags>` | Run by tags: `--tags smoke,critical` |
| `--env <env>` | Target environment |

## Workflow Position

```
TestSpec Approved
       │
       ▼
/test-run (async execution) ←─ THIS SKILL
       │
       ├──▶ Returns immediately
       │     Tests run in background
       │
       ▼
/snapshot-build (triggered after tests pass)
```

## Workflow

### 1. Verify Prerequisites

```
1. Load TestSpec
2. Verify status == "approved"
3. Check environment prerequisites
```

### 2. Launch Async Execution

```
1. Create execution record
2. Start background test runner
3. Return execution ID to user
```

### 3. Output

```
Test Execution Started
═════════════════════

TestSpec: TEST-1.0.0-auth
Execution ID: run-2025-01-15-001
Status: RUNNING (background)

Tests queued: 31
  E2E: 8
  Integration: 6
  Unit: 15
  Performance: 2

─────────────────────────────
Tests running in background.
Results will be available for /snapshot-build.

To check status:
  Check test runner output or CI/CD pipeline
```

## Results Handling

Test results feed into the next phase:
- **All Pass**: Proceed to `/snapshot-build`
- **Failures**: Human reviews results and decides:
  - L1 (lowest): Dev fixes locally, no spec changes
  - L2 (medium): Update TestSpec, re-run
  - L3 (highest): Update ProductSpec, restart workflow

## Integration

This skill:
- Is triggered manually after TestSpec approval
- Runs tests asynchronously
- Results feed into `/snapshot-build`
