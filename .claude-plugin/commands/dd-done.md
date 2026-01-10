---
description: Mark changes as done, merge RC to official documents
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash]
arguments:
  - name: cr-id
    description: Change ID (e.g. CR-001)
    required: true
  - name: options
    description: Optional parameter --dry-run
    required: false
---

# /dd-done - Mark Change Complete

> Common definitions in `_base.md`

## Usage

```
/dd-done <CR-id>
/dd-done <CR-id> --dry-run
```

Process single CR at a time.

## Parameter Description

### --dry-run

Preview mode, only output operations to be executed without actually modifying files.

**Output content**:
- RC files to be merged
- Files to be archived
- Cases to be promoted to blessed/ (if has referenced_by)
- Index files to be updated

## Prerequisites

Status must be `confirmed`, otherwise prompt to run `/dd-confirm` first.

---

## Execution Steps

1. **Load CR**: Validate status
2. **Merge RC to formal documents**:
   - Find all `features/*.rc-{id}.md` and `*.tech.rc-{id}.md`
   - Replace formal documents with RC content (or rename to formal document)
   - Remove `rc_for` field
   - Update footer to `_Updated: {date} | CR-{id}_`
3. **Delete RC files**
4. **Update `_deps.yaml`**: Rebuild dependency graph from merged documents
5. **Update CR status**: status → `done`, add `completed: {date}`
6. **Archive**: Move CR to `changes/archive/`
7. **Archive specs**: Move `specs/CR-{id}.*.md` to `specs/archive/` (if exist)
8. **Archive test cases**: Move `cases/CR-{id}/` to `cases/archive/CR-{id}/` (if exist)
9. **Promote reusable cases**: If CR has `referenced_by` in `cases/_index.yaml`:
   - Copy cases to `cases/blessed/` with versioned naming:
   - Format: `{feature}.{cr-id}.{platform}.yaml`
   - e.g. `archive/CR-001/login.ios.yaml` → `blessed/login.CR-001.ios.yaml`
   - Add source comment in file header: `# Promoted from: CR-{id} | {date}`
10. **Update indices**: `changes/_index.yaml`, `specs/_index.yaml`, `cases/_index.yaml`

---

## _deps.yaml Maintenance

Rebuild `features/_deps.yaml` from all formal documents' frontmatter:

```yaml
updated: {date}

graph:
  {feature}:
    deps: [from feature.md deps field]
    affects: [from feature.md affects field]
```

Ensure bidirectional consistency (A.deps contains B → B.affects should contain A).
