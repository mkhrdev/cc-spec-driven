<div align="center">

# cc-spec-driven

**Claude Code Plugin · Spec-Driven · Document-First**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)](https://docs.anthropic.com/en/docs/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

[English](https://github.com/mkhrdev/cc-spec-driven/blob/main/README.md) · [中文](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.zh.md)

</div>

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin for managing requirement documents, tracking changes, and outputting specs — enabling downstream tools to generate code based on high-quality specs and complete E2E testing.

---

## Installation

```bash
# Install
claude plugins:install mkhrdev/cc-spec-driven

# Update
claude plugins:update cc-spec-driven
```

After installation, type `/dd-` in Claude Code to see all available commands.

---

## ✨ Why This

| Traditional Approach | Spec-Driven |
|---------------------|-------------|
| Write all docs at once | Incrementally enrich with changes |
| Docs and code disconnect | Documents are single source of truth |
| Manual consistency maintenance | AI-assisted checking and updates |
| Hard to track requirement changes | CR-ID tracking throughout |
| Rewrite test cases repeatedly | blessed mechanism for reuse |

## 🎯 Core Principles

- **Change-Driven** - No need to write all docs upfront, enrich incrementally with changes
- **AI-Assisted** - Natural language input, AI organizes into unified format
- **Documents as Truth** - Confirmed documents are the sole basis for development and testing
- **Rough to Fine** - Describe module overview first, refine features on demand

## Framework Advantages

### Intelligent Dependency Management
- **Bidirectional relationship tracking**: deps (dependencies) + affects (dependents) auto-maintained
- **Scope extension check**: Auto-detect missed dependency changes during confirm
- **Global dependency graph**: `_deps.yaml` provides quick impact analysis

### Context Loading Optimization
- **Layered loading strategy**: Level 0-3 on-demand loading, optimizes token usage
- **Always load**: project.yaml, glossary.yaml, overview.md
- **On-demand load**: Only load frontmatter and content of relevant features

### Safety Guard Design
- **RC Preview Mechanism**: Generate preview before merge, formal merge after human confirmation
- **Implicit State Rollback**: Auto-warn and rollback when modifying confirmed CR
- **Dependency Scope Protection**: Prevent overlooking affected documents

### Parallel Work Friendly
- Git branches for concurrency, independent RC per branch, no parallel bottlenecks
- Use rebase skill for merging, natural language merge rules are clear

### E2E Test Case Reuse
- **blessed mechanism**: Promote verified cases to reusable versions
- **runFlow reference**: Auto-reuse test cases from dependency features
- **Version tracking**: CR-id naming, supports multiple versions coexistence

---

## Quick Start

```bash
# 1. Initialize product
/dd-init my-product

# 2. Create change (natural language description)
/dd-update "Add user login feature, supporting email and phone number"

# 3. Confirm and generate RC preview
/dd-confirm CR-001

# 4. Merge to official docs
/dd-done CR-001
```

---

## Command Overview

### Core Commands

| Command | Purpose |
|---------|---------|
| `/dd-init` | Initialize product |
| `/dd-update` | Create/modify change |
| `/dd-confirm` | Confirm change, generate RC preview |
| `/dd-done` | Merge RC to official docs |
| `/dd-status` | View status |
| `/dd-drop` | Abandon change |

### Auxiliary Commands

| Command | Purpose |
|---------|---------|
| `/dd-check` | Consistency check |
| `/dd-rebase` | Handle branch conflicts |
| `/dd-spec-dev` | Generate dev spec |
| `/dd-spec-test` | Generate test spec |
| `/dd-test-case` | Generate test cases (Maestro) |

> Full documentation: [CLAUDE.md - Skills](CLAUDE.md#skills)

---

## Directory Structure

```
spec/
├── CLAUDE.md                     # AI behavior guide
├── README.md                     # This file
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

## Complete User Guide

<!-- GUIDE:concepts:15 -->
### Core Concepts

| Term | Description |
|------|-------------|
| CR (Change Request) | Change request, unique identifier for a requirement change |
| RC (Release Candidate) | Preview document, generated after confirm, merged after done |
| feature | Feature document, describes an independent business feature |
| deps | Dependencies: which features I depend on |
| affects | Dependents: which features depend on me |
| blessed | Verified reusable test cases |
<!-- /GUIDE:concepts -->

<!-- GUIDE:workflow:21 -->
### State Transitions

```
  feature ╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌▶ /done
     │◀───────────────────────────┬─────────────┐                               ▲
     ▼                            │             │                               │
  /update ◀────▶ /confirm ────▶ /spec ────▶ /test-case ────▶ [dev] ────▶ /test-run(todo)
     │              │          (dev/test)
     └──────┬───────┘             ▲             ▲
            ▼                     ╎             ╎
         /drop ╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┴╌╌╌╌╌╌╌╌╌╌╌╌╌┘
```

- `draft`: Initial state, editable
- `confirmed`: Confirmed, RC preview generated
- `done`: Completed, RC merged to official docs
- `dropped`: Abandoned
<!-- /GUIDE:workflow -->

<!-- GUIDE:dd-init:30 -->
### /dd-init

Initialize product directory structure.

**Usage**:
```
/dd-init <product-name>
```

**Scenarios**:

| Scenario | Input | Behavior |
|----------|-------|----------|
| Normal init | `/dd-init my-app` | Create complete directory structure and initial files |
| Product exists | `/dd-init my-app` | Reject, prompt product already exists |
| Invalid name | `/dd-init "my app"` | Reject, prompt name format requirements |

**Generated files**:
- `project.yaml` - Product config (next_cr_id: 1)
- `glossary.yaml` - Glossary
- `overview.md` - Product overview
- `features/_deps.yaml` - Dependency graph
- `changes/_index.yaml` - Change index
- `specs/_index.yaml` - Spec index
- `cases/_index.yaml` - Cases index
- `cases/config.yaml` - Maestro config
<!-- /GUIDE:dd-init -->

<!-- GUIDE:dd-status:40 -->
### /dd-status

View product status and recommended actions.

**Usage**:
```
/dd-status              # List all products
/dd-status <product>    # Specific product details
```

**Scenarios**:

| Scenario | Output |
|----------|--------|
| No products | Prompt no products, suggest `/dd-init` |
| Multiple products | Product list + change statistics |
| Single product | Feature list, RC list, change status, recommendations |
| Found stale RC | Mark ⚠️ stale, suggest `/dd-rebase` |

**Recommended next steps**:

| Current State | Recommended Action |
|---------------|-------------------|
| No draft CR | `/dd-update "description"` |
| Has draft CR | `/dd-confirm CR-{id}` |
| Has confirmed (no spec) | `/dd-spec-dev` or `/dd-spec-test` |
| Has confirmed (has spec) | `/dd-done CR-{id}` |
| Has stale RC | `/dd-rebase CR-{id}` |

**RC expiration detection**: RC's `updated` < official doc's `updated` → stale
<!-- /GUIDE:dd-status -->

<!-- GUIDE:dd-update:75 -->
### /dd-update

Create or modify change request.

**Usage**:
```
/dd-update "<description>"              # Create new change
/dd-update "<description>" --implemented # Implemented feature (skip spec-dev)
/dd-update "<description>" --bootstrap   # Cold start (directly create feature.md)
/dd-update <CR-id>                       # Modify existing change
/dd-update <CR-id> "<additional desc>"   # Append content
```

**Create new change scenarios**:

| Scenario | Behavior |
|----------|----------|
| Normal create | Generate CR-{id}, analyze dependencies, multi-round clarification |
| Vague description | Trigger clarification, require specifics |
| Involves new feature | Create CR + feature.tech.md |
| Involves existing feature | Analyze impact scope |
| Dependency changes found | Record in CR's `## Dependency Changes` |
| Undefined term | Console reminder, non-blocking |

**--bootstrap scenarios**:

| Scenario | Behavior |
|----------|----------|
| Cold start new feature | Directly create feature.md, no CR |
| Cold start existing feature | Update existing feature.md |
| After completion | Suggest running `/dd-check` |

**Modify existing change scenarios**:

| CR Status | Behavior |
|-----------|----------|
| draft | Directly append content |
| confirmed | ⚠️ Warn then rollback: delete RC/spec, status→draft |
| done | ❌ Reject: archived |
| dropped | ❌ Reject: abandoned |

**confirmed rollback warning**:
```
Warning: CR-{id} is confirmed, modification will trigger:
- Status rollback to draft
- Delete RC files and spec files
Continue? (y/n)
```

**Impact scope analysis**:
```
Impact scope analysis:
- Direct impact: profile (deps login)
- Indirect impact: checkout (via profile)
  ↳ Suggest reviewing if update needed
```
<!-- /GUIDE:dd-update -->

<!-- GUIDE:dd-confirm:55 -->
### /dd-confirm

Confirm change, generate RC preview.

**Usage**:
```
/dd-confirm <CR-id>
/dd-confirm <CR-id> --dry-run    # Preview only, don't create
```

**Precondition**: Status must be `draft`

**Scenarios**:

| Scenario | Behavior |
|----------|----------|
| Normal confirm | Generate RC files, status→confirmed |
| --dry-run | Only output files to be generated |
| Status confirmed | ❌ Already confirmed |
| Status done | ❌ Archived |
| Status dropped | ❌ Abandoned |
| CR not found | ❌ CR doesn't exist |
| Out-of-scope deps found | ⚠️ Auto-expand CR, exit, require re-run |

**Out-of-scope dependency handling**:

When `## Dependency Changes` involves features not in `## Impact Scope`:
1. Auto-update CR (expand impact scope)
2. Output prompt and exit
3. User reviews then re-runs

```
⚠️ Found out-of-scope dependencies, CR-{id} updated:

Impact scope added:
- checkout (needs affects update)

CR updated, please review and re-run:
/dd-confirm CR-{id}
```

**Generate RC**:

| Case | Handling |
|------|----------|
| Feature exists | Load feature.md, apply changes, save as .rc-{id}.md |
| New feature | Extract info from CR, create .rc-{id}.md |
<!-- /GUIDE:dd-confirm -->

<!-- GUIDE:dd-done:55 -->
### /dd-done

Mark change complete, merge RC to official docs.

**Usage**:
```
/dd-done <CR-id>
/dd-done <CR-id> --dry-run    # Preview only, don't execute
```

**Precondition**: Status must be `confirmed`

**Scenarios**:

| Scenario | Behavior |
|----------|----------|
| Normal complete | Merge RC, archive, update indexes |
| --dry-run | Only output operations to execute |
| Status draft | ❌ Please `/dd-confirm` first |
| Status done | ❌ Archived |
| Status dropped | ❌ Abandoned |
| Has spec files | Archive together to specs/archive/ |
| Has test cases | Archive together to cases/archive/ |
| Referenced by other CR | Promote cases to blessed/ |

**Execution steps**:
1. Merge RC to official docs (remove `rc_for`, update footer)
2. Delete RC files
3. Rebuild `_deps.yaml`
4. Update CR status→done, add `completed`
5. Archive CR→changes/archive/
6. Archive spec→specs/archive/ (if exists)
7. Archive cases→cases/archive/ (if exists)
8. Promote reusable cases to blessed/ (if has referenced_by)
9. Update all indexes

**blessed promotion**:
- Condition: `cases/_index.yaml` has `referenced_by`
- Naming: `{feature}.{cr-id}.{platform}.yaml`
- Comment: `# Promoted from: CR-{id} | {date}`
<!-- /GUIDE:dd-done -->

<!-- GUIDE:dd-drop:45 -->
### /dd-drop

Abandon change.

**Usage**:
```
/dd-drop <CR-id>
/dd-drop <CR-id> "<reason>"
```

**Scenarios**:

| CR Status | Behavior |
|-----------|----------|
| draft | Directly abandon, move to dropped/ |
| confirmed | Delete RC/spec, then abandon |
| done | ❌ Archived, use git revert |
| dropped | ❌ Already abandoned |

**Confirmation prompt**:
```
About to abandon CR-{id}:
- Delete RC file: features/login.rc-{id}.md
- Delete spec: specs/CR-{id}.dev.md
- Move cases: cases/CR-{id}/ → cases/dropped/

Continue? (y/n)
```

**Reference cleanup**:

| Case | Handling |
|------|----------|
| This CR references other CRs | Clean referenced party's `referenced_by` |
| Other CRs reference this CR | ⚠️ Warn, clean referencer's `refs` |

**Restore abandoned**: Manually move from dropped/, change status to draft
<!-- /GUIDE:dd-drop -->

<!-- GUIDE:dd-check:50 -->
### /dd-check

Check document consistency (console output only, non-blocking).

**Usage**:
```
/dd-check [product]
/dd-check [product] --scope=<docs|cases|all>
/dd-check [product] --type=<check-type>
```

**--scope**:

| Value | Check Content |
|-------|---------------|
| docs | Requirement docs (default) |
| cases | Test cases |
| all | All |

**docs check types**:

| --type | Content |
|--------|---------|
| glossary | Glossary and document consistency |
| format | Document format (sections, frontmatter) |
| deps | Dependencies (existence, cycles, bidirectional, _deps.yaml) |
| refs | References (CR↔feature, orphan docs) |
| status | Status consistency (long-term draft, index sync) |

**cases check types**:

| --type | Content |
|--------|---------|
| runflow | runFlow path validity |
| index | Index consistency (refs/referenced_by) |
| blessed | blessed version check |

**Output format**:
```
=== Document Check: my-app ===

[Critical] Must fix:
  ✗ deps: login.deps contains non-existent feature

[Warning] Should fix:
  ⚠ format: profile.md missing ## Boundaries

Check complete: 1 critical | 1 warning | 0 info
```
<!-- /GUIDE:dd-check -->

<!-- GUIDE:dd-rebase:45 -->
### /dd-rebase

Handle branch merge conflicts, reapply change intent.

**Usage**:
```
/dd-rebase <CR-id>
```

**Scenarios**:

| Trigger | Behavior |
|---------|----------|
| `/dd-status` shows stale | Reapply change intent |
| Post git merge doc conflicts | Regenerate based on intent |

**Conflict types**:

| Type | Description |
|------|-------------|
| Concept conflict | Different definitions of same concept |
| Metric conflict | Different values/limits settings |
| Order conflict | Different ordering of processes/steps |
| Scope conflict | Different feature boundary definitions |

**Execution flow**:
1. Load CR (extract change intent)
2. Load current main branch docs
3. Intent extraction (priority: original input > change content > clarification records)
4. Conflict analysis
5. Output analysis report
6. **Human confirmation** of handling approach
7. Regenerate

**Note**: CR retains original status after rebase, adds "Rebase Record" section
<!-- /GUIDE:dd-rebase -->

<!-- GUIDE:dd-spec-dev:45 -->
### /dd-spec-dev

Generate development spec.

**Usage**:
```
/dd-spec-dev <CR-id>
```

**Preconditions**:
- Status must be `confirmed`
- Skip when type=implemented

**Scenarios**:

| Scenario | Behavior |
|----------|----------|
| Normal generate | Generate specs/CR-{id}.dev.md |
| Status draft | ❌ Please `/dd-confirm` first |
| type=implemented | ⚠️ Skip |
| Spec exists | Ask to overwrite |
| tech.md has pending decisions | Interactive confirmation |
| External repo unreachable | Mark "to be confirmed" |

**Spec structure**:
```markdown
# Development Spec: CR-{id}

## Overview
## Prerequisites
## Technical Constraints
## Technical Decisions
## Implementation Tasks
### Task N: {task name}
**Goal**: {user story}
**Context**: related files, interfaces
**Acceptance criteria**: [ ] criteria
**Implementation hints**: reuse xxx
## Implementation Order
## References
```
<!-- /GUIDE:dd-spec-dev -->

<!-- GUIDE:dd-spec-test:50 -->
### /dd-spec-test

Generate test spec (Gherkin format).

**Usage**:
```
/dd-spec-test <CR-id>
/dd-spec-test --init    # Generate initial test spec based on all features
```

**Precondition**: Status must be `confirmed` (--init exception)

**Scenarios**:

| Scenario | Behavior |
|----------|----------|
| Normal generate | Generate specs/CR-{id}.test.md |
| --init | Generate INIT.test.md based on all features |
| Status error | ❌ Please `/dd-confirm` first |
| Spec exists | Ask to overwrite |

**TC numbering rules**:

| Range | Purpose |
|-------|---------|
| TC-001~099 | Main flow tests |
| TC-100~199 | Boundary tests |
| TC-200~299 | Exception tests |

**Given/When/Then principles**:
- Use business language, no UI details
- Given describes state, not how to reach it
- When describes intent, not specific actions
- Then describes verifiable results

**Spec structure**:
```markdown
# Test Spec: CR-{id}

## Test Scope
Features: login, profile
Platforms: [web] [ios] [android]

## Test Cases
### TC-001: User successful login
**Priority**: P0
**Given**: User is not logged in
**When**: User enters correct credentials
**Then**: Homepage displayed

## Regression Checkpoints
- [profile] Can access profile after login
```
<!-- /GUIDE:dd-spec-test -->

<!-- GUIDE:dd-test-case:50 -->
### /dd-test-case

Generate Maestro test cases.

**Usage**:
```
/dd-test-case <CR-id>
/dd-test-case <CR-id> --platform=<ios|android|web|all>
/dd-test-case <CR-id> --dry-run
/dd-test-case <CR-id> --force
```

**Preconditions**:
- Status must be `confirmed`
- `specs/CR-{id}.test.md` must exist

**Scenarios**:

| Scenario | Behavior |
|----------|----------|
| Normal generate | Generate Maestro YAML |
| --platform=ios | Generate iOS only |
| --dry-run | Only output file list |
| --force | Skip overwrite prompt |
| Files exist | Ask overwrite/skip/cancel |
| No test spec | ❌ Please `/dd-spec-test` first |

**Dependency reuse**:

| Scenario | Handling |
|----------|----------|
| Dep feature has blessed cases | runFlow reference latest version |
| Dep feature has in-progress cases | runFlow reference |
| Dep feature has no cases | Generate inline prerequisite steps |

**Regression testing**:
- Parse `## Regression Checkpoints`
- Find existing cases
- Generate runFlow with `regression` tag

**Output**:
- Files: `cases/CR-{id}/{feature}.{platform}.yaml`
- Report: `cases/CR-{id}/REPORT.md`
- Summary: file count, runFlow count, regression count, TODO count
<!-- /GUIDE:dd-test-case -->

<!-- GUIDE:file-formats:55 -->
### File Format Reference

**project.yaml**:
```yaml
name: my-app
description: My application
created: 2026-01-07
language: zh
repos: []
next_cr_id: 1
```

**feature.md frontmatter**:
```yaml
---
id: {uuid}
title: User Login
deps: [auth]
affects: [profile, checkout]
updated: 2026-01-07
---
```

**CR-{id}.md frontmatter**:
```yaml
---
uuid: {uuid}
status: draft | confirmed | done | dropped
type: change | implemented
created: 2026-01-07
updated: 2026-01-07
completed: 2026-01-08    # when done
dropped: 2026-01-08      # when dropped
---
```

**_deps.yaml**:
```yaml
# Auto-generated, do not edit manually
updated: 2026-01-07
graph:
  login:
    deps: [auth]
    affects: [profile]
```

**cases/_index.yaml**:
```yaml
cases:
  - cr: CR-001
    status: done
    feature: login
    platforms: [ios, android]
    refs: []
    referenced_by: [CR-003]
```
<!-- /GUIDE:file-formats -->

<!-- GUIDE:advanced:35 -->
### Advanced Tips

**Context Loading Tiers**:

| Level | Content | Trigger |
|-------|---------|---------|
| Level 0 | File list | Always |
| Level 1 | frontmatter | Target + direct deps |
| Level 2 | Section titles | deps/affects involved |
| Level 3 | Full content | When modification needed |

**Always loaded**: project.yaml, glossary.yaml, overview.md, _deps.yaml

**Dependency management**:
- deps: What I depend on
- affects: What depends on me
- _deps.yaml: Global dependency view, quick impact analysis

**Test case reuse**:
- blessed/: Verified reusable cases
- runFlow: Auto-reference dependency feature's cases
- Prefer latest version in blessed/
<!-- /GUIDE:advanced -->

<!-- GUIDE:faq:25 -->
### FAQ

**Q: How to restore an abandoned CR?**
A: Manually move from dropped/, change status to draft

**Q: Can I modify a confirmed CR?**
A: Yes, `/dd-update` will warn then rollback to draft

**Q: How to handle stale RC?**
A: Run `/dd-rebase` to reapply change intent

**Q: How to skip dev spec?**
A: Use `--implemented` flag for already implemented features

**Q: Can I modify spec after generation?**
A: Yes, manually modify is fine, but recommend triggering regeneration via `/dd-update`
<!-- /GUIDE:faq -->

---

## Roadmap

### Phase 1: E2E Testing Loop ✅
- [x] `/dd-spec-test` output in Gherkin format
- [x] `/dd-test-case` generate Maestro YAML
- [x] blessed versioned naming and reference management

### Phase 2: VSCode Extension
- [ ] CR status panel
- [ ] Dependency graph visualization

---

## Relationship with Other Tools

This framework is **upstream** of AI development toolchains, not a replacement:

| Tool | Positioning | Relationship with This Framework |
|------|-------------|----------------------------------|
| AWS Kiro | Single-project dev assistant | This framework outputs Spec, Kiro generates code |
| Cursor | AI programming IDE | This framework outputs Spec, Cursor implements |
| GitHub Spec Kit | Spec format standard | This framework manages Spec lifecycle |

### Unique Value

| Feature | This Framework | Other Tools |
|---------|---------------|-------------|
| RC Preview Mechanism | ✅ | ❌ |
| Bidirectional Dependency Tracking | ✅ Auto | Manual or None |
| Context Loading Tiers | ✅ | ❌ |
| Multi-Product Management | ✅ | Single Project |
| CR Lifecycle | ✅ Full Tracking | None or Partial |

---

<div align="center">

**If this project helps you, please give it a Star!**

</div>
