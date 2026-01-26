---
description: Check documentation consistency (output only, no modifications)
allowed-tools: [Read, Glob, Grep]
arguments:
  - name: options
    description: --scope=docs|cases|all --type=<check-type>
    required: false
---

# /dd-check - Check Document Consistency

> Common definitions in `_base.md`

All check results **output to console only**, no file generation, non-blocking.

## Usage

```
/dd-check
/dd-check --scope=<scope>
/dd-check --scope=<scope> --type=<check-type>
```

## --scope Parameter

| Value | Description |
|-------|-------------|
| `docs` | Check requirement documents only (default) |
| `cases` | Check test cases only |
| `all` | Check everything |

## Check Types (--type)

### docs Scope

| Type | Check Content |
|------|---------------|
| glossary | Glossary and document usage consistency |
| format | Document format standards (required sections, frontmatter) |
| deps | Dependencies (existence, cycles, bidirectional consistency, _deps.yaml sync) |
| refs | References (CR↔feature, orphan documents, RC↔CR) |
| status | Status consistency (long-term draft, index sync) |

### cases Scope

| Type | Check Content |
|------|---------------|
| runflow | runFlow path validity (whether referenced files exist) |
| index | Index consistency (refs/referenced_by vs actual files) |
| blessed | blessed version check (multiple version notification) |

---

## _deps.yaml Validation

Check `features/_deps.yaml` consistency with each feature.md:

1. **Sync check**: Whether dependencies in _deps.yaml match feature.md frontmatter
2. **Bidirectional check**: A.deps contains B → B.affects should contain A
3. **Existence check**: Whether referenced features exist

If inconsistencies found, output specific differences.

---

## Output Format

```
=== Document Check: {product} ===

[Critical] Must fix:
  ✗ {type}: {issue description}

[Warning] Recommended fix:
  ⚠ {type}: {issue description}

[Info] For reference:
  ℹ {type}: {issue description}

Statistics: {n} features | {m} tech docs | {p} in-progress | {q} completed

Check completed: {x} critical | {y} warning | {z} info
```

---

## Design Principles

- **Advisory only, non-blocking**: Check results for reference only
- **Console output**: No report file generation
- **User discretion**: Whether to fix is decided by the user
