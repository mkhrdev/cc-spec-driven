---
description: Generate Maestro test cases from test spec
allowed-tools: [Read, Write, Glob, Grep]
arguments:
  - name: cr-id
    description: Change ID (e.g. CR-001)
    required: true
  - name: options
    description: --platform=ios|android|web|all --dry-run --force
    required: false
---

# /dd-test-case - Generate Test Cases

> Common definitions in `_base.md`

## Usage

```
/dd-test-case <CR-id>
/dd-test-case <CR-id> --platform=<ios|android|web|all>
/dd-test-case <CR-id> --dry-run
/dd-test-case <CR-id> --force
```

## Prerequisites

- Status must be `confirmed`
- `specs/CR-{id}.test.md` must exist
- If not exists, prompt to run `/dd-spec-test CR-{id}` first

---

## Overwrite Detection

Check if `cases/CR-{id}/` has existing files before generation. If exists, ask user: Overwrite / Skip / Cancel.

`--force` skips prompt and overwrites directly.

---

## Reference Change Detection

When overwriting to regenerate, compare old and new `refs`. If changed, prompt user whether to update reference relationships in `cases/_index.yaml`.

---

## Execution Steps

1. **Load CR**: Validate status, extract involved features
2. **Load test spec**: `specs/CR-{id}.test.md`
3. **Load context**:
   - `changes/CR-{id}.md` → impact scope
   - `features/{feature}.md` → deps field (dependencies)
4. **Load config**: `cases/config.yaml`
5. **Load existing cases index**: `cases/_index.yaml` (for runFlow reference)
6. **Load UI Mapping**: `cases/ui-mapping.yaml` (if exists)
7. **Parse dependencies**:
   - Get dependency features from feature.deps
   - Find existing test cases for dependencies (prefer blessed/, then CR-*/)
8. **Parse Gherkin**: Extract TC, Given/When/Then
9. **Regression test processing**:
   - Parse `## Regression Checkpoints`
   - Associate with feature, find existing cases
   - Generate runFlow calls + regression tag
10. **Generate Maestro YAML**:
    - Dependency has test cases → runFlow reference
    - Dependency has no cases → generate inline prerequisite steps
    - CR-specific steps → generate specific operations
    - Unknown elements → mark `# TODO: UI element`
11. **Write files**: `cases/CR-{id}/{feature}.{platform}.yaml`
12. **Generate report**: `cases/CR-{id}/REPORT.md`
13. **Update index**: `cases/_index.yaml`
14. **Output summary**: file count, runFlow references, regression cases, TODOs

---

## Parameter Description

### --platform

| Value | Behavior |
|-------|----------|
| ios | Generate iOS cases only |
| android | Generate Android cases only |
| web | Generate Web cases only |
| all | Generate all configured platforms (default, from config.yaml) |

### --dry-run

Only output file list and structure preview, do not create files.

---

## Dependency Reuse Mechanism

### Search Priority

1. `blessed/{feature}.*.{platform}.yaml` - Promoted reusable cases, latest version by CR-id descending
2. `CR-*/{feature}.{platform}.yaml` - Other in-progress CR's cases

Automatically selects latest version in blessed (highest CR-id) during generation, and updates `refs` / `referenced_by` in `cases/_index.yaml`.

---

## Regression Test Processing

Extract from test spec's `## Regression Checkpoints`, automatically find existing cases and generate runFlow calls with `regression` tag.

---

## Output

Output file list, statistics (runFlow references, regression cases, TODOs), and next step suggestions.

---

## UI Mapping (Optional)

`cases/ui-mapping.yaml` provides mapping from business semantics to UI locators, serving as bridge between Gherkin (business behavior) and Maestro (UI operations).
