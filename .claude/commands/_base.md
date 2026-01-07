# DD Framework Common Definitions

This file defines shared strategies and formats for all `/dd-*` commands. Commands should reference this file instead of duplicating definitions.

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
products/{product}/
├── project.yaml
├── glossary.yaml
├── overview.md
├── features/
│   ├── _deps.yaml           # Dependency graph index (auto-maintained)
│   ├── {feature}.md
│   ├── {feature}.rc-{id}.md
│   ├── {feature}.tech.md
│   └── {feature}.tech.rc-{id}.md
├── changes/
│   ├── _index.yaml
│   ├── CR-{id}.md
│   ├── archive/
│   └── dropped/
└── specs/
    ├── _index.yaml
    ├── CR-{id}.dev.md
    ├── CR-{id}.test.md
    ├── archive/
    └── dropped/
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
