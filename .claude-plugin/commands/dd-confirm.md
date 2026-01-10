---
description: Confirm changes and generate RC preview documents
allowed-tools: [Read, Write, Edit, Glob, Grep]
arguments:
  - name: cr-id
    description: Change ID (e.g. CR-001)
    required: true
  - name: options
    description: Optional parameter --dry-run
    required: false
---

# /dd-confirm - Confirm Change

> Common definitions in `_base.md`

## Usage

```
/dd-confirm <CR-id>
/dd-confirm <CR-id> --dry-run
```

## Parameter Description

### --dry-run

Preview mode, only output operations to be executed without actually creating or modifying files.

**Output content**:
- RC files to be generated
- Dependency changes summary
- Out-of-scope dependency detection results (if any)

## Prerequisites

Status must be `draft`, otherwise reject with current status message.

---

## Execution Steps

1. **Load CR**: Validate status
2. **Check out-of-scope dependencies**: See "Dependency Scope Extension" below
3. **Load context**: Follow strategy in `_base.md`, focus on features in impact scope
4. **Generate RC documents**: Generate preview version for each involved feature
5. **Apply dependency changes**: Write dependency changes recorded in CR to RC files
6. **Update status**: status → `confirmed`

### Generate RC Documents

- **Feature exists**: Load `features/{feature}.md`, apply changes, save as `.rc-{id}.md`
- **New feature**: Extract info from CR, create `.rc-{id}.md`

RC format in `_base.md`.

---

## Dependency Scope Extension

Check all features in CR's `## Dependency Changes` section:

**If a feature is NOT in `## Impact Scope`**:
1. Auto-update CR (extend impact scope, add dependency changes)
2. Output notice (format in `_base.md` common output formats)
3. **Exit, do not generate RC**

User needs to review updated CR, then re-run `/dd-confirm`.

**If all dependencies are in scope**: Proceed to generate RC normally.
