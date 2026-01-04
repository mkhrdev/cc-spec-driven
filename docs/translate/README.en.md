<div align="center">

# Spec-Driven Document-First Development Framework

**Spec-Driven · Document-First · AI-Assisted**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-blueviolet)](https://claude.ai/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

English · [中文](../../README.md) · [日本語](./README.ja.md)

---

*A spec-driven, document-first framework for managing multi-product requirements*

</div>

## ✨ Why This Framework?

| Traditional Approach | Spec-Driven |
|---------------------|-------------|
| Write all docs upfront | Incrementally expand with changes |
| Docs drift from code | Docs are the single source of truth |
| Manual consistency checks | AI-assisted validation and updates |
| Hard to track requirement changes | CR-ID tracking throughout |

## 🎯 Core Principles

- **📝 Change-Driven** - No need to write everything at once; docs grow with each change
- **🤖 AI-Assisted** - Natural language input, AI formats into consistent structure
- **📚 Docs as Truth** - Confirmed documentation is the sole reference for dev and testing
- **🔍 Top-Down Refinement** - Start with module overview, then detail features as needed

## 🚀 Quick Start

```bash
# 1. Initialize a product
/dd-init my-product

# 2. Describe product overview
/dd-update "We have user management, orders, and payment modules..."

# 3. Confirm the change
/dd-confirm CR-001

# 4. (Optional) Generate specs
/dd-spec-dev CR-001
/dd-spec-test CR-001

# 5. Mark as done
/dd-done CR-001
```

## 📂 Directory Structure

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
        ├── features/             # Feature documentation
        │   ├── {feature}.md      # Business requirements (official)
        │   ├── {feature}.rc-{id}.md    # Business requirements (CR preview)
        │   ├── {feature}.tech.md       # Technical consensus (official)
        │   └── {feature}.tech.rc-{id}.md # Technical consensus (CR preview)
        │
        ├── changes/              # Change records
        │   ├── _index.yaml       # Change index
        │   ├── CR-{id}.md        # In-progress changes
        │   ├── archive/          # Completed changes
        │   └── dropped/          # Abandoned changes
        │
        └── specs/                # Specification files
            ├── _index.yaml       # Spec index
            ├── CR-{id}.dev.md    # Dev specs
            ├── CR-{id}.test.md   # Test specs
            ├── archive/          # Completed specs
            └── dropped/          # Abandoned specs
```

## 🛠️ Skills

> **dd** = **D**ocument-**D**riven, also representing the "D"s in Spec-Driven **D**ocument-First.
> All skills use the `/dd-` prefix, embodying the "document-driven" philosophy.

### Core Skills

| Skill | Purpose | Description |
|-------|---------|-------------|
| `/dd-init` | Initialize product | Creates complete directory structure |
| `/dd-status` | View status | Product/change/RC/spec statistics |
| `/dd-update` | Create/modify change | Natural language input, confirmed can rollback |
| `/dd-confirm` | Confirm change | Generates RC preview documents |
| `/dd-done` | Mark complete | Merges RC to official docs, archives |
| `/dd-drop` | Abandon change | Deletes RC and specs, moves to dropped |

### Auxiliary Skills

| Skill | Purpose | Description |
|-------|---------|-------------|
| `/dd-check` | Comprehensive check | Console output only, non-blocking |
| `/dd-rebase` | Handle branch conflicts | Re-apply changes based on intent |
| `/dd-spec-dev` | Generate dev spec | Requires confirmed status |
| `/dd-spec-test` | Generate test spec | Requires confirmed, supports --init |

## 🔄 Workflow

```
┌──────────┐      ┌──────────┐
│ feature  │╌╌╌╌╌▶│  /done   │
└──────────┘      └────▲─────┘
     │                 │
     ▼                 │
┌──────────┐      ┌───────────┐      ┌─────────────────┐
│ /update  │◀────▶│ /confirm  │─────▶│ spec (dev/test) │
└────┬─────┘      └─────┬─────┘      └─────────────────┘
     │                  │
     └────────┬─────────┘
              ▼
        ┌──────────┐
        │  /drop   │
        └──────────┘
```

### Status Description

| Status | Document Change |
|--------|-----------------|
| update | Only CR, no doc changes |
| confirm | Generates `.rc-{id}.md` preview |
| done | Deletes RC, merges to official docs |
| drop | Deletes RC, no rollback needed |

## 📖 Documentation

- **[CLAUDE.md](../../CLAUDE.md)** - AI behavior guide, document formats, workflow details

## 🤔 Design Philosophy

This framework is designed around "how to construct the most valuable context," maximizing the value of every token. Most so-called Spec-Driven Development approaches are anti-patterns—dumping piles of documents on an LLM, where excessive "rules" actually degrade the model's attention and compliance. It's easy to fall into the over-engineering trap if you don't find the right balance.

To truly leverage Spec-Driven Development effectively, **it must be modular and incremental**. Break requirements into modules and plans, then apply Spec-Driven practices to each step individually.

---

<div align="center">

**If this project helps you, please give it a ⭐ Star!**

</div>
