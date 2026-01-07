<div align="center">

# Spec-Driven Document-First Development Framework

**Spec-Driven · Document-First · AI-Assisted**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-blueviolet)](https://claude.ai/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

[English](https://github.com/mkhrdev/cc-spec-driven/blob/main/README.md) · [中文](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.zh.md) · [日本語](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.ja.md)

---

*A spec-driven, document-first documentation management framework for managing multi-product requirement documents*

</div>

## Positioning: Upstream of AI Development Workflow

```
┌─────────────────────────────────────────┐
│  This Framework (Orchestrator)          │
│  Document asset management → Spec gen   │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  Kiro / Cursor / OpenCode (Worker)      │
│  Execute code generation based on Spec  │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  Maestro / Playwright (E2E)             │
│  Execute validation based on Test Spec  │
└─────────────────────────────────────────┘
```

## Why Choose This Framework?

### Comparison with Alternatives

| Feature | This Framework | GitHub Spec Kit | AWS Kiro | Cursor Plan |
|---------|---------------|-----------------|----------|-------------|
| **RC Preview Mechanism** | ✅ Unique | ❌ | ❌ | ❌ |
| **Bidirectional Dependency Tracking** | ✅ Automatic | ❌ Manual | ❌ | ❌ |
| **Context Loading Layers** | ✅ Optimized | ❌ | ❌ | ❌ |
| **Multi-product Management** | ✅ | ❌ Single project | ❌ Single project | ❌ Single task |
| **CR Lifecycle Management** | ✅ Complete | ❌ | ❌ | ❌ |
| **Change Tracking** | ✅ CR-ID | ❌ | ❌ Regenerate | ❌ |

### Unique Value

| Traditional Approach | Spec-Driven |
|---------------------|-------------|
| Write all docs at once | Incrementally enrich with changes |
| Docs and code disconnect | Documents are single source of truth |
| Manual consistency maintenance | AI-assisted checking and updates |
| Hard to track requirement changes | CR-ID tracking throughout |

## Core Principles

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
- Git branches for concurrency, independent RC per branch
- Use rebase for merging, clear rules
- No parallel bottlenecks

## Quick Start

```bash
# 1. Initialize product
/dd-init my-product

# 2. Describe product overview
/dd-update "We have user management, orders, and payment modules..."

# 3. Confirm change
/dd-confirm CR-001

# 4. (Optional) Generate specs
/dd-spec-dev CR-001
/dd-spec-test CR-001

# 5. Complete
/dd-done CR-001
```

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
        │
        ├── features/             # Feature documents
        │   ├── {feature}.md      # Business requirements (formal)
        │   ├── {feature}.rc-{id}.md    # Business requirements (CR preview)
        │   ├── {feature}.tech.md       # Technical consensus (formal)
        │   └── {feature}.tech.rc-{id}.md # Technical consensus (CR preview)
        │
        ├── changes/              # Change records
        │   ├── _index.yaml       # Change index
        │   ├── CR-{id}.md        # In-progress changes
        │   ├── archive/          # Completed changes
        │   └── dropped/          # Dropped changes
        │
        └── specs/                # Spec files
            ├── _index.yaml       # Spec index
            ├── CR-{id}.dev.md    # Development spec
            ├── CR-{id}.test.md   # Test spec
            ├── archive/          # Completed specs
            └── dropped/          # Dropped specs
```

## Skills

> **dd** = **D**ocument-**D**riven, abbreviation of Spec-Driven **D**ocument-First.
> All skills prefixed with `/dd-` to reflect the "document-driven" core philosophy.

### Core Skills

| Skill | Purpose | Description |
|-------|---------|-------------|
| `/dd-init` | Initialize product | Create complete directory structure |
| `/dd-status` | View status | Product/change/RC/spec statistics |
| `/dd-update` | Create/modify change | Natural language input, confirmed can rollback |
| `/dd-confirm` | Confirm change | Generate RC preview documents |
| `/dd-done` | Mark complete | Merge RC to formal documents, archive |
| `/dd-drop` | Abandon change | Delete RC and spec, move to dropped |

### Auxiliary Skills

| Skill | Purpose | Description |
|-------|---------|-------------|
| `/dd-check` | Consistency check | Console output only, non-blocking |
| `/dd-rebase` | Handle branch conflicts | Intent-based change reapplication |
| `/dd-spec-dev` | Generate dev spec | Requires confirmed status |
| `/dd-spec-test` | Generate test spec | Requires confirmed, supports --init |

## Workflow Details

### State Transitions

```
draft → confirmed → done (archived)
  │         │
  └────┬────┘
       ↓
    dropped
```

### Standard Flow

| Step | Command | Output | Description |
|------|---------|--------|-------------|
| 1 | `/dd-update "description"` | CR-{id}.md | Create change record, analyze impact scope |
| 2 | Human review | - | Multiple `/dd-update CR-{id}` adjustments possible |
| 3 | `/dd-confirm CR-{id}` | *.rc-{id}.md | Generate RC preview documents |
| 4 | `/dd-spec-dev\|test` | specs/*.md | Optional: Generate dev/test specs |
| 5 | `/dd-done CR-{id}` | Formal docs | Merge RC, archive CR and specs |

### Dependency Change Flow

```
/dd-update   →  Analyze dependency changes, record to CR
                ↓
/dd-confirm  →  Check out-of-scope dependencies
                ├─ Found → Auto-extend CR, exit awaiting review
                └─ None → Generate RC, update bidirectional deps
                ↓
/dd-done     →  Merge RC, rebuild _deps.yaml
```

### Special Modes

| Mode | Command | Purpose |
|------|---------|---------|
| Bootstrap | `/dd-update "description" --bootstrap` | Directly create feature.md, skip CR |
| Implemented | `/dd-update "description" --implemented` | Run CR flow but skip dev spec gen |
| State Rollback | `/dd-update CR-{id}` (confirmed) | Warn then delete RC/spec, rollback to draft |

## Scope Boundaries

This framework **only manages documents**:

| In Scope | Out of Scope |
|----------|--------------|
| Requirement document management | Development progress tracking |
| Change tracking (CR) | Code sync |
| Spec output | Cross-product dependencies |
| Dependency analysis | Release version management |

## Design Philosophy

Designed around "how to construct the most valuable context" so every token has maximum impact. Most so-called Spec-Driven Development is anti-pattern — dumping documents to LLM with lots of "rules" impairs attention and compliance. Getting the balance wrong easily leads to over-engineering traps.

Real good Spec-Driven Development **must be modular and progressive**. Split requirements into modules and plans, then do Spec-Driven separately for each step.

---

<div align="center">

**If this project helps you, please give it a ⭐ Star!**

</div>
