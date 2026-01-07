<div align="center">

# Spec-Driven Document-First Development Framework

**Spec-Driven · Document-First · AI-Assisted**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-blueviolet)](https://claude.ai/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

[English](https://github.com/mkhrdev/cc-spec-driven/blob/main/README.md) · [中文](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.zh.md) · [日本語](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.ja.md)

</div>

A spec-driven, document-first documentation management framework for managing multi-product requirement documents.

**Core Positioning**: Manage requirement documents, track changes, and output specs, enabling downstream tools (Kiro, Cursor, OpenCode) to generate code based on high-quality specs and complete E2E testing.

---

## ✨ Why Choose This Framework

| Traditional Approach | Spec-Driven |
|---------------------|-------------|
| Write all docs at once | Incrementally enrich with changes |
| Docs and code disconnect | Documents are single source of truth |
| Manual consistency maintenance | AI-assisted checking and updates |
| Hard to track requirement changes | CR-ID tracking throughout |

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
        │
        ├── features/             # Feature documents
        │   ├── {feature}.md      # Business requirements (formal)
        │   └── {feature}.rc-{id}.md  # Business requirements (CR preview)
        │
        ├── changes/              # Change records
        │   ├── _index.yaml       # Change index
        │   ├── CR-{id}.md        # In-progress changes
        │   ├── archive/          # Completed changes
        │   └── dropped/          # Dropped changes
        │
        └── specs/                # Spec files
            ├── CR-{id}.dev.md    # Development spec
            ├── CR-{id}.test.md   # Test spec
            └── archive/          # Completed specs
```

---

## Roadmap

### Phase 1: Foundation
- [ ] `/dd-spec-test` output in Gherkin format
- [ ] E2E integration docs (Maestro)

### Phase 2: VSCode Extension
- [ ] CR status panel
- [ ] Dependency graph visualization

### Phase 3: E2E Testing Loop
- [ ] `/dd-spec-e2e` generate Maestro YAML
- [ ] Complete E2E integration example

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
