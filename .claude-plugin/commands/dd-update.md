---
description: Create or modify change records
allowed-tools: [Read, Write, Edit, Glob, Grep]
arguments:
  - name: target
    description: Change description (create new CR) or CR-id (modify existing)
    required: true
  - name: extra
    description: Additional description or flags (--implemented, --bootstrap)
    required: false
---

# /dd-update - Update Documents

> Common definitions in `_base.md`

## Usage

```
/dd-update "<description>"                    # Create new change
/dd-update "<description>" --implemented      # Already implemented feature
/dd-update "<description>" --bootstrap        # Bootstrap mode
/dd-update <CR-id>                            # Modify existing change
/dd-update <CR-id> "<additional description>" # Append content
```

---

## Create New Change

1. **Generate ID**: Read `next_cr_id` from `project.yaml`, increment and save, generate UUID
2. **Load context**: Follow Context Loading in `_base.md` (including `_deps.yaml`)
3. **Analyze impact scope**: Identify involved features, determine if new features needed
4. **Analyze dependency changes**: Read `_deps.yaml`, derive bidirectional relationships, output impact analysis
5. **Terminology check**: Displays in console only
6. **Multi-round clarification**: Prompt for clarification when requirements are unclear, record to CR
7. **Create CR**: Format in `_base.md`
8. **Create tech.md**: If new feature involved

---

## Modify Existing Change

| Status | Handling |
|--------|----------|
| draft | Append directly |
| confirmed | Prompt warning, then rollback (delete RC/spec, status→draft) |
| done/dropped | Reject |

**Confirmed rollback**:
```
Warning: CR-{id} is confirmed, modification will trigger:
- Status rollback to draft
- Delete RC files and spec files
Continue? (y/n)
```

---

## --bootstrap Mode

Directly create/update feature.md, skip CR/spec generation.

1. Load context (including `_deps.yaml`)
2. Analyze impact scope
3. Directly create/update `features/{feature}.md`
4. Update dependency relationships in document
5. Update `_deps.yaml`
6. Terminology check (displays in console only)

After bootstrap, suggest running `/dd-check`.

---

## --implemented

Run CR workflow, but mark type as `implemented`, skip spec-dev generation.
