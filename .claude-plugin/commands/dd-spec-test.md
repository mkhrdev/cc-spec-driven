---
description: Generate test spec (requires confirmed status)
allowed-tools: [Read, Write, Glob, Grep]
arguments:
  - name: cr-id
    description: Change ID (e.g. CR-001), or --init for initialization
    required: true
---

# /dd-spec-test - Generate Test Spec

> Common definitions in `_base.md`

## Usage

```
/dd-spec-test <CR-id>
/dd-spec-test --init
```

## Prerequisites

- Status must be `confirmed`
- `--init` is exception, used for project initialization

## Execution Steps

1. **Load CR**: Validate status
2. **Load context**: Involved feature.md (or RC), focus on "Boundaries" section
3. **Generate test cases**: Given-When-Then format
4. **Generate spec**: `specs/CR-{id}.test.md`
5. **Update index**: `specs/_index.yaml`

## Spec Structure

```markdown
# Test Spec: CR-{id}

## Test Scope
## Test Cases
### TC-NNN: {case-name}
**Priority**: P0|P1|P2
**Type**: E2E|API|Unit
**Given**: ...
**When**: ...
**Then**: ...

## Boundary Testing
## Regression Checkpoints
```

## Priority and Type

| Priority | Meaning | | Type | Meaning |
|----------|---------|---|------|---------|
| P0 | Core flow, blocks release | | E2E | End-to-end |
| P1 | Important feature, recommended fix | | API | Interface test |
| P2 | Edge case, low priority | | Unit | Unit test |

## --init Mode

Generate complete test spec for all existing feature.md files as `specs/INIT.test.md`.
