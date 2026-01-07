@import CLAUDE.local.md

# CLAUDE.md - Spec-Driven Document-First Development Framework

## Overview

A spec-driven, document-first documentation management framework for managing multi-product requirement documents.

**Core Principles**:
- Documents are Single Source of Truth
- Progressive enhancement: start rough, refine gradually
- AI-assisted maintenance, human confirmation

**Configuration**:
- Change ID prefix: `CR`
- Default language: `en`

---

## Language Policy

- Framework documentation: English only
- Command prompts and outputs: English only
- Products (`products/*/`): Per-product language setting in project.yaml
- User communication: Follow user's language preference

---

## Directory Structure

```
spec/
├── CLAUDE.md                     # This file
│
└── products/
    └── {product}/
        ├── project.yaml          # Product config (includes next_cr_id)
        ├── glossary.yaml         # Glossary (human-maintained)
        ├── overview.md           # Product overview
        ├── features/             # Feature documents
        │   ├── _deps.yaml        # Dependency graph index (auto-maintained)
        │   ├── {feature}.md      # Business requirements (formal)
        │   ├── {feature}.rc-{id}.md    # Business requirements (CR preview)
        │   ├── {feature}.tech.md       # Technical consensus (formal)
        │   └── {feature}.tech.rc-{id}.md # Technical consensus (CR preview)
        ├── changes/              # Change records
        │   ├── _index.yaml       # Change index
        │   ├── CR-{id}.md        # In-progress changes
        │   ├── archive/          # Completed changes
        │   └── dropped/          # Dropped changes
        ├── specs/                # Spec files
        │   ├── _index.yaml       # Spec index
        │   ├── CR-{id}.dev.md    # Development spec
        │   ├── CR-{id}.test.md   # Test spec
        │   ├── archive/          # Completed specs
        │   └── dropped/          # Dropped specs
        └── cases/                # Test cases
            ├── _index.yaml       # Cases index
            ├── config.yaml       # Maestro config
            ├── CR-{id}/          # In-progress cases
            ├── blessed/          # Reusable cases
            ├── archive/          # Completed cases
            └── dropped/          # Dropped cases
```

---

## Skills

### Core Skills

| Skill | Purpose |
|-------|---------|
| `/dd-init` | Initialize product |
| `/dd-status` | View status |
| `/dd-update` | Create/modify change |
| `/dd-confirm` | Confirm change (generate RC preview) |
| `/dd-done` | Mark complete (merge RC to formal) |
| `/dd-drop` | Abandon change |

### Auxiliary Skills

| Skill | Purpose |
|-------|---------|
| `/dd-check` | Consistency check (console output) |
| `/dd-rebase` | Handle branch merge conflicts |
| `/dd-spec-dev` | Generate dev spec (requires confirmed) |
| `/dd-spec-test` | Generate test spec (requires confirmed) |
| `/dd-test-case` | Generate test cases (Maestro YAML) |

### Command Architecture

All `/dd-*` commands share common definitions in `.claude/commands/_base.md`.
Each command file contains only usage and execution steps, format definitions reference `_base.md`.

---

## Workflow

### State Transitions

```
draft → confirmed → done (archived)
  │         │
  └────┬────┘
       ↓
    dropped
```

### Standard Flow

```
1. /dd-update "description"  → Create CR (draft)
2. Human review              → Multiple rounds possible
3. /dd-confirm CR-{id}       → Generate RC preview (confirmed)
4. /dd-spec-dev|test         → Generate specs (optional)
5. /dd-done CR-{id}          → Merge to formal documents
```

### Dependency Change Flow

```
/dd-update   →  Detect dependency changes, record to CR
/dd-confirm  →  Check out-of-scope deps, extend CR if needed then exit
/dd-done     →  Merge RC to formal docs, update _deps.yaml
```

### Bootstrap Mode

```
/dd-update "description" --bootstrap
→ Directly create feature.md, skip CR
→ Suggest running /dd-check after completion
```

---

## Context Loading & Document Formats

→ See `.claude/commands/_base.md` for details

---

## Design Decisions

The following are deliberate design choices:

1. **Single branch, single RC**: Documents managed in git, use rebase for merges

2. **Implicit state rollback**: `/dd-update <CR-id>` on confirmed status triggers rollback

3. **Post-spec generation out of scope**: Only responsible for docs and specs

4. **Terminology console-only**: glossary.yaml maintained by humans

5. **Consistency checks on demand**: Index sync, dependency validation improved as needed

6. **Version management decoupled from release**: CR version tracking unrelated to external release versions

### Scope Boundaries

- This framework **only manages documents**, development progress tracking and code sync are out of scope
- Cross-product dependencies are out of design scope, each product is independent

### Tool vs Process

- `/dd-check` is a **tool** for humans, not a mandatory process step
- Consistency checks are non-blocking, humans decide whether to fix
