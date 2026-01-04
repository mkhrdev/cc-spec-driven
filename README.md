<div align="center">

# Spec-Driven Document-First Development Framework

**规格驱动 · 文档先行 · AI辅助**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-blueviolet)](https://claude.ai/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

[English](./docs/translate/README.en.md) · 中文 · [日本語](./docs/translate/README.ja.md)

---

*规格驱动文档先行的文档管理框架，用于管理多产品的需求文档*

</div>

## ✨ 为什么选择这个框架？

| 传统方式 | Spec-Driven |
|---------|-------------|
| 一次性写完所有文档 | 随变更逐步补充 |
| 文档和代码脱节 | 文档是唯一真相源 |
| 手动维护一致性 | AI辅助检查和更新 |
| 需求变更难追踪 | CR-ID 全程追踪 |

## 🎯 核心理念

- **📝 变更驱动** - 不要求一次写完所有文档，随变更逐步补充
- **🤖 AI辅助** - 自然语言输入，AI整理成统一格式
- **📚 文档即真相** - 确认后的文档是开发测试的唯一依据
- **🔍 先粗后细** - 先描述模块全景，再按需细化功能

## 🚀 快速开始

```bash
# 1. 初始化产品
/dd-init my-product

# 2. 描述产品全景
/dd-update "我们有用户管理、订单、支付三个模块..."

# 3. 确认变更
/dd-confirm CR-001

# 4. (可选) 生成规格
/dd-spec-dev CR-001
/dd-spec-test CR-001

# 5. 完成
/dd-done CR-001
```

## 📂 目录结构

```
spec/
├── CLAUDE.md                     # AI行为指南
├── README.md                     # 本文件
│
└── products/
    └── {product}/
        ├── project.yaml          # 产品配置 (含 next_cr_id)
        ├── glossary.yaml         # 术语表 (人类维护)
        ├── overview.md           # 产品全景
        │
        ├── features/             # 功能文档
        │   ├── {feature}.md      # 业务需求 (正式)
        │   ├── {feature}.rc-{id}.md    # 业务需求 (CR预览版)
        │   ├── {feature}.tech.md       # 技术共识 (正式)
        │   └── {feature}.tech.rc-{id}.md # 技术共识 (CR预览版)
        │
        ├── changes/              # 变更记录
        │   ├── _index.yaml       # 变更索引
        │   ├── CR-{id}.md        # 进行中的变更
        │   ├── archive/          # 已完成的变更
        │   └── dropped/          # 已放弃的变更
        │
        └── specs/                # 规格文件
            ├── _index.yaml       # 规格索引
            ├── CR-{id}.dev.md    # 开发规格
            ├── CR-{id}.test.md   # 测试规格
            ├── archive/          # 已完成的规格
            └── dropped/          # 已放弃的规格
```

## 🛠️ Skills

> **dd** = **D**ocument-**D**riven，亦即 Spec-Driven **D**ocument-First 的缩写。
> 所有 skill 以 `/dd-` 为前缀，体现「文档驱动」的核心理念。

### 核心 Skills

| Skill | 作用 | 说明 |
|-------|------|------|
| `/dd-init` | 初始化产品 | 创建完整目录结构 |
| `/dd-status` | 查看状态 | 产品/变更/RC/规格统计 |
| `/dd-update` | 创建/修改变更 | 支持自然语言输入，confirmed 可回退 |
| `/dd-confirm` | 确认变更 | 生成 RC 预览文档 |
| `/dd-done` | 标记完成 | 合并 RC 到正式文档，归档 |
| `/dd-drop` | 放弃变更 | 删除 RC 和 spec，移动到 dropped |

### 辅助 Skills

| Skill | 作用 | 说明 |
|-------|------|------|
| `/dd-check` | 综合检查 | 仅 console 输出，不阻断流程 |
| `/dd-rebase` | 处理分支冲突 | 基于意图重新应用变更 |
| `/dd-spec-dev` | 生成开发规格 | 需 confirmed 状态 |
| `/dd-spec-test` | 生成测试规格 | 需 confirmed，支持 --init |

## 🔄 工作流

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

### 状态说明

| 状态 | 文档变化 |
|------|----------|
| update | 只有 CR，无文档变更 |
| confirm | 生成 `.rc-{id}.md` 预览版 |
| done | 删除 RC，合并到正式文档 |
| drop | 删除 RC，无需回滚 |

## 📖 详细文档

- **[CLAUDE.md](./CLAUDE.md)** - AI行为指南、文档格式、工作流详情

## 🤔 设计哲学

围绕「如何构造最有价值的上下文」来设计，要让每一个 Token 发挥最大的价值。大多数所谓的 Spec-Driven Development 是反模式的，把一堆文档丢给 LLM，大量的「规则」反而会影响 LLM 的注意力和遵循能力。掌握不好那个度，容易陷进过度设计的陷阱。

真正好用 Spec-Driven Development，**一定是模块化、渐进式的**。把需求拆分成多个模块、计划，每一步再单独进行 Spec-Driven。

---

<div align="center">

**如果这个项目对你有帮助，请给个 ⭐ Star！**

</div>
