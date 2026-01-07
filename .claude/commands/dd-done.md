# /dd-done - Mark Change Complete

> Common definitions in `_base.md`

## Usage

```
/dd-done <CR-id>
```

Process single CR at a time.

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
8. **Update indices**: `changes/_index.yaml`, `specs/_index.yaml`

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
