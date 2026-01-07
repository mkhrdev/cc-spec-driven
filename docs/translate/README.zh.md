<div align="center">

# Spec-Driven Document-First Development Framework

**规格驱动 · 文档先行 · AI辅助**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-blueviolet)](https://claude.ai/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

[English](https://github.com/mkhrdev/cc-spec-driven/blob/main/README.md) · [中文](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.zh.md) · [日本語](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.ja.md)

---

*规格驱动文档先行的文档管理框架，用于管理多产品的需求文档*

</div>

## 定位：AI 开发工作流的上游

```
┌─────────────────────────────────────────┐
│  本框架 (Orchestrator)                   │
│  文档资产管理 → Spec 生成                 │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  Kiro / Cursor / OpenCode (Worker)       │
│  基于 Spec 执行代码生成                   │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  Maestro / Playwright (E2E)              │
│  基于 Test Spec 执行验证                  │
└─────────────────────────────────────────┘
```

## 为什么选择这个框架？

### 与竞品对比

| 特性 | 本框架 | GitHub Spec Kit | AWS Kiro | Cursor Plan |
|------|--------|-----------------|----------|-------------|
| **RC 预览机制** | ✅ 独有 | ❌ | ❌ | ❌ |
| **双向依赖追踪** | ✅ 自动 | ❌ 手动 | ❌ | ❌ |
| **Context Loading 分层** | ✅ 优化 | ❌ | ❌ | ❌ |
| **多产品管理** | ✅ | ❌ 单项目 | ❌ 单项目 | ❌ 单任务 |
| **CR 生命周期管理** | ✅ 完整 | ❌ | ❌ | ❌ |
| **变更追踪** | ✅ CR-ID | ❌ | ❌ 每次从头 | ❌ |

### 独特价值

| 传统方式 | Spec-Driven |
|---------|-------------|
| 一次性写完所有文档 | 随变更逐步补充 |
| 文档和代码脱节 | 文档是唯一真相源 |
| 手动维护一致性 | AI辅助检查和更新 |
| 需求变更难追踪 | CR-ID 全程追踪 |

## 核心理念

- **变更驱动** - 不要求一次写完所有文档，随变更逐步补充
- **AI辅助** - 自然语言输入，AI整理成统一格式
- **文档即真相** - 确认后的文档是开发测试的唯一依据
- **先粗后细** - 先描述模块全景，再按需细化功能

## 框架优势

### 智能依赖管理
- **双向关系追踪**: deps（依赖）+ affects（被依赖）自动维护
- **范围扩展检查**: confirm 时自动检测遗漏的依赖变更
- **全局依赖图**: `_deps.yaml` 提供快速影响分析

### Context Loading 优化
- **分级加载策略**: Level 0-3 按需加载，优化 token 使用
- **始终加载**: project.yaml, glossary.yaml, overview.md
- **按需加载**: 仅加载相关 feature 的 frontmatter 和内容

### 安全卡设计
- **RC 预览机制**: 合并前生成预览，人类确认后再正式合并
- **隐式状态回退**: 修改 confirmed CR 时自动警告并回退
- **依赖范围保护**: 防止遗漏受影响的文档

### 并行工作友好
- git 分支实现并发，每分支独立 RC
- 合并时用 rebase，规则清晰
- 不存在并行瓶颈

## 快速开始

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

## 目录结构

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

## Skills

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

## 工作流详解

### 状态流转

```
draft → confirmed → done (归档)
  │         │
  └────┬────┘
       ↓
    dropped
```

### 标准流程

| 步骤 | 命令 | 产出 | 说明 |
|------|------|------|------|
| 1 | `/dd-update "描述"` | CR-{id}.md | 创建变更记录，分析影响范围 |
| 2 | 人类 review | - | 可多轮 `/dd-update CR-{id}` 调整 |
| 3 | `/dd-confirm CR-{id}` | *.rc-{id}.md | 生成 RC 预览文档 |
| 4 | `/dd-spec-dev\|test` | specs/*.md | 可选：生成开发/测试规格 |
| 5 | `/dd-done CR-{id}` | 正式文档 | 合并 RC，归档 CR 和 specs |

### 依赖变更流转

```
/dd-update   →  分析依赖变更，记录到 CR
                ↓
/dd-confirm  →  检查范围外依赖
                ├─ 有遗漏 → 自动扩展 CR，退出等待 review
                └─ 无遗漏 → 生成 RC，更新双向依赖
                ↓
/dd-done     →  合并 RC，重建 _deps.yaml
```

### 特殊模式

| 模式 | 命令 | 用途 |
|------|------|------|
| 冷启动 | `/dd-update "描述" --bootstrap` | 直接创建 feature.md，跳过 CR |
| 已实现 | `/dd-update "描述" --implemented` | 走 CR 流程但不生成 dev spec |
| 状态回退 | `/dd-update CR-{id}` (confirmed) | 警告后删除 RC/spec，回退到 draft |

## 范围边界

本框架**只管文档**：

| 在范围内 | 不在范围内 |
|---------|-----------|
| 需求文档管理 | 开发进度追踪 |
| 变更追踪 (CR) | 代码同步 |
| Spec 输出 | 跨产品依赖 |
| 依赖分析 | 发布版本管理 |

## 设计哲学

围绕「如何构造最有价值的上下文」来设计，要让每一个 Token 发挥最大的价值。大多数所谓的 Spec-Driven Development 是反模式的，把一堆文档丢给 LLM，大量的「规则」反而会影响 LLM 的注意力和遵循能力。掌握不好那个度，容易陷进过度设计的陷阱。

真正好用 Spec-Driven Development，**一定是模块化、渐进式的**。把需求拆分成多个模块、计划，每一步再单独进行 Spec-Driven。

---

<div align="center">

**如果这个项目对你有帮助，请给个 ⭐ Star！**

</div>
