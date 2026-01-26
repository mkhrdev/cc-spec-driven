# DD Framework Common Definitions

This file defines shared strategies and formats for all `/dd-*` commands. Commands should reference this file instead of duplicating definitions.

---

## Execution Environment Check

**All commands must check the current directory before execution**:

1. Check if `project.yaml` file exists
2. If not found, prompt the user:
   ```
   project.yaml not found in current directory.

   Please confirm:
   - If this is a new project, run /dd-init first
   - If you have an existing product directory, cd to that directory and retry
   ```
3. Exit the current command

**Exception**: `/dd-init` command doesn't require this check (it initializes the product directory)

---

## Context Loading Strategy

```
Level 0: File List
└── glob features/*.md (exclude .rc-*.md, .tech.md)

Level 1: Meta Information
└── Read YAML frontmatter (first 20 lines) of each file

Level 2: Structure Information
└── Read all ## headings

Level 3: Detailed Content
└── Read specific sections on demand
```

**Always load**: `project.yaml`, `glossary.yaml`, `overview.md`, `features/_deps.yaml`
**Always Level 0**: All feature file list
**Level 1**: Only load frontmatter of [target feature + direct dependencies from _deps.yaml]

**Level 2 trigger**: Features in deps/affects, features in CR impact scope
**Level 3 trigger**: Need to modify specific sections, need to understand technical details

**Additional loading for test case generation**:
- `specs/CR-{id}.test.md` (main input)
- `changes/CR-{id}.md` (impact scope)
- `features/{feature}.md` (deps field)
- `cases/_index.yaml` (existing cases for runFlow)
- `cases/config.yaml`
- `cases/ui-mapping.yaml` (if exists)

---

## State Transitions

```
draft → confirmed → done (archived)
  │         │
  └────┬────┘
       ↓
    dropped
```

- `update → confirm`: `/dd-confirm`
- `confirm → update`: `/dd-update CR-{id}` triggers rollback
- `confirm → done`: `/dd-done`
- `* → dropped`: `/dd-drop`

---

## File Format Reference

### feature.md

```yaml
---
id: {uuid}
title: {title}
deps: [{dependencies}]
affects: [{affected}]
updated: {date}
---
```

Sections: `## Overview` `## Related` `## Features` `## Boundaries`
Footer: `_Updated: {date} | CR-{id}_`

### feature.rc-{id}.md

Same as feature.md, with additional field:
```yaml
rc_for: CR-{id}
```
Footer: `_Preview: CR-{id} | Generated: {date}_`

### feature.tech.md

```yaml
---
id: {uuid}
feature: {feature-name}
cr: CR-{id}
updated: {date}
---
```

Sections: `## Repositories` `## Technical Decisions` `## Interface Contracts` `## Caveats`

### CR-{id}.md

```yaml
---
uuid: {uuid}
status: draft | confirmed | done | dropped
type: change | implemented
created: {date}
updated: {date}
---
```

Sections: `## Original Input` `## Clarification Records` `## Change Content` `## Impact Scope` `## Dependency Changes`

### Dependency Change Format

```markdown
## Dependency Changes
- {feature}.deps: +[added] -[removed]
- {feature}.affects: +[added] -[removed]
```

### {feature}.{platform}.yaml (Test Cases)

Maestro YAML format. Contains `appId`, `name`, `tags` (CR-id, feature, priority), `env`.
Organized by TC with Given/When/Then steps. Platform suffixes: `.ios.yaml`, `.android.yaml`, `.web.yaml`

### REPORT.md (Test Cases Report)

Generated report with Summary table (platform/files/scenarios/regression/TODO), Dependencies, and Files list.

---

## Index File Formats

### changes/_index.yaml

```yaml
changes:
  - id: CR-001
    uuid: {uuid}
    title: {title}
    status: done | confirmed | draft | dropped
    created: {date}
    updated: {date}
    completed: {date}    # when done
    dropped: {date}      # when dropped
    features: [{feature}]
```

### specs/_index.yaml

```yaml
specs:
  - cr: CR-001
    dev: true | false
    test: true | false
    status: active | archived
```

### cases/_index.yaml

```yaml
cases:
  - cr: CR-001
    status: active | done | dropped
    feature: {feature}
    platforms: [ios, android, web]
    refs: []
    referenced_by: [CR-002]
    files: [{path, platform, scenarios, has_regression}]
```

### cases/config.yaml

```yaml
appId: com.example.app
platforms: [ios, android, web]
flows: ["CR-*/**/*.yaml", "blessed/*.yaml"]
```

### features/_deps.yaml

Dependency graph index, auto-maintained by `/dd-done`, validated by `/dd-check`.

```yaml
# Auto-generated, do not edit manually
# Updated by: /dd-done
updated: {date}

graph:
  {feature}:
    deps: [{dependency features}]
    affects: [{affected features}]
```

**Purpose**:
- Provides global dependency view, reduces Level 1 loading
- No need to read each file's frontmatter to build dependency graph

---

## Directory Structure

```
{product}/                        # Product directory = working directory
├── project.yaml
├── glossary.yaml
├── overview.md
├── features/
│   ├── _deps.yaml                # Dependency graph index (auto-maintained)
│   ├── {feature}.md
│   ├── {feature}.rc-{id}.md
│   ├── {feature}.tech.md
│   └── {feature}.tech.rc-{id}.md
├── changes/
│   ├── _index.yaml
│   ├── CR-{id}.md
│   ├── archive/
│   └── dropped/
├── specs/
│   ├── _index.yaml
│   ├── CR-{id}.dev.md
│   ├── CR-{id}.test.md
│   ├── archive/
│   └── dropped/
└── cases/                        # Test Cases
    ├── _index.yaml               # Cases index
    ├── config.yaml               # Maestro config
    ├── ui-mapping.yaml           # UI mapping (optional)
    ├── CR-{id}/                  # In-progress CR
    │   ├── REPORT.md             # Generation report
    │   └── {feature}.{platform}.yaml
    ├── blessed/                  # Reusable cases (versioned: {feature}.{cr-id}.{platform}.yaml)
    ├── archive/                  # Completed
    └── dropped/                  # Abandoned
```

---

## Common Behaviors

### Terminology Check
When undefined terms are found, **displayed in console only**, not recorded to CR:
```
⚠️ Undefined term found: "xxx"
Suggestion: Consider updating glossary.yaml
```

### Status Validation Templates

**done status**:
```
Error: CR-{id} is already archived.
```

**dropped status**:
```
Error: CR-{id} is already dropped.
```

**requires confirmed**:
```
Error: CR-{id} status is draft, cannot perform this operation.
Please run first: /dd-confirm CR-{id}
```

---

## Common Output Formats

### Successful Operation
```
{Action} completed: {target}

{details list}

Next steps:
- {suggested command}
```

### Error
```
Error: {reason}
Suggestion: {fix suggestion}
```

### Dependency Scope Extension (confirm only)

When `/dd-confirm` finds dependency changes involving documents outside CR scope, automatically extend CR scope and exit:

```
⚠️ Out-of-scope dependency found, CR-{id} updated:

Impact scope extended:
- {feature} (needs {deps|affects} update)

Dependency changes added:
- {feature}.{deps|affects}: +[{added}]

CR updated, please review and re-run:
/dd-confirm CR-{id}
```

**Flow**:
1. Check all features in CR's `## Dependency Changes` section
2. If a feature is NOT in `## Impact Scope` → extend scope
3. Add corresponding bidirectional dependency changes
4. Update CR file
5. Output notice and exit (do not generate RC)

---

## Impact Analysis

Output impact analysis during `/dd-update` (for reference only, non-blocking):

```
Impact Analysis:
- Direct impact: {feature} (deps {target})
- Indirect impact: {feature} (via {intermediate feature})
  ↳ Consider reviewing for updates
```

**Calculation**:
- Direct impact: All features in _deps.yaml whose deps contain target feature
- Indirect impact: Features that depend on directly impacted features (show one level only)

---

## blessed/ Naming Convention

File naming: `{feature}.{cr-id}.{platform}.yaml` (e.g., `login.CR-001.ios.yaml`)

Search priority: blessed/ by CR-id descending for latest → CR-*/ other in-progress cases

Add source comment when promoting: `# Promoted from: CR-{id} | {date}`
