---
description: Generate development spec (requires confirmed status)
allowed-tools: [Read, Write, Glob, Grep, WebFetch]
arguments:
  - name: cr-id
    description: Change ID (e.g. CR-001)
    required: true
---

# /dd-spec-dev - Generate Development Spec

> Common definitions in `_base.md`

## Usage

```
/dd-spec-dev <CR-id>
```

## Prerequisites

- Status must be `confirmed`
- Skip when type is `implemented`

## Execution Steps

1. **Load CR**: Validate status and type
2. **Load context**:
   - Involved feature.md (or RC), tech.md (or RC)
   - repos config from project.yaml
3. **Read external repo info**: README.md, config files (mark as "pending confirmation" if repo unreachable)
4. **Process tech.md open decisions**: Interactive confirmation for option selection
5. **Generate spec**: `specs/CR-{id}.dev.md`
6. **Update index**: `specs/_index.yaml`

## Spec Structure

```markdown
# Development Spec: CR-{id}

## Overview
## Prerequisites
## Technical Constraints
## Technical Decisions
## Implementation Tasks
### Task N: {task-name}
**Goal**: {user story}
**Context**: Related files, interfaces
**Acceptance criteria**: [ ] criteria
**Implementation hints**: Reuse xxx

## Implementation Order
## References
```

## Design Principles

- **Self-contained**: Can develop just by reading spec
- **Executable**: Acceptance criteria can be converted to tests
- **Ordered**: Task dependencies are clear
