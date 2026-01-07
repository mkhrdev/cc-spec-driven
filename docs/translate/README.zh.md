<div align="center">

# Spec-Driven Document-First Development Framework

**规格驱动 · 文档先行 · AI辅助**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-blueviolet)](https://claude.ai/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

[English](https://github.com/mkhrdev/cc-spec-driven/blob/main/README.md) · [中文](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.zh.md) · [日本語](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.ja.md)

</div>

管理需求文档、追踪变更、输出规格，让下游工具基于高质量 Spec 执行代码生成，并完成e2e测试。

---

## ✨ 为什么这个

| 传统方式 | Spec-Driven |
|---------|-------------|
| 一次性写完所有文档 | 随变更逐步补充 |
| 文档和代码脱节 | 文档是唯一真相源 |
| 手动维护一致性 | AI辅助检查和更新 |
| 需求变更难追踪 | CR-ID 全程追踪 |
| 测试用例重复编写 | blessed 机制复用 |

## 🎯 核心理念

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
- git 分支实现并发，每分支独立 RC，不存在并行瓶颈
- 合并时使用rebase skill，自然语言合并规则清晰

### E2E 测试用例复用
- **blessed 机制**: 已验证的用例提升为可复用版本
- **runFlow 引用**: 自动复用依赖功能的测试用例
- **版本追踪**: 带 CR-id 命名，支持多版本共存

---

## 快速开始

```bash
# 1. 初始化产品
/dd-init my-product

# 2. 创建变更（自然语言描述）
/dd-update "添加用户登录功能，支持邮箱和手机号"

# 3. 确认并生成 RC 预览
/dd-confirm CR-001

# 4. 合并到正式文档
/dd-done CR-001
```

---

## 命令概览

### 核心命令

| 命令 | 作用 |
|------|------|
| `/dd-init` | 初始化产品 |
| `/dd-update` | 创建/修改变更 |
| `/dd-confirm` | 确认变更，生成 RC 预览 |
| `/dd-done` | 合并 RC 到正式文档 |
| `/dd-status` | 查看状态 |
| `/dd-drop` | 放弃变更 |

### 辅助命令

| 命令 | 作用 |
|------|------|
| `/dd-check` | 一致性检查 |
| `/dd-rebase` | 处理分支冲突 |
| `/dd-spec-dev` | 生成开发规格 |
| `/dd-spec-test` | 生成测试规格 |
| `/dd-test-case` | 生成测试用例 (Maestro) |

> 完整说明：[CLAUDE.md - Skills](CLAUDE.md#skills)

---

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
        ├── features/             # 功能文档
        │   ├── _deps.yaml        # 依赖图索引 (自动维护)
        │   ├── {feature}.md      # 业务需求 (正式)
        │   ├── {feature}.rc-{id}.md    # 业务需求 (CR预览版)
        │   ├── {feature}.tech.md       # 技术共识 (正式)
        │   └── {feature}.tech.rc-{id}.md # 技术共识 (CR预览版)
        ├── changes/              # 变更记录
        │   ├── _index.yaml       # 变更索引
        │   ├── CR-{id}.md        # 进行中的变更
        │   ├── archive/          # 已完成的变更
        │   └── dropped/          # 已放弃的变更
        ├── specs/                # 规格文件
        │   ├── _index.yaml       # 规格索引
        │   ├── CR-{id}.dev.md    # 开发规格
        │   ├── CR-{id}.test.md   # 测试规格
        │   ├── archive/          # 已完成的规格
        │   └── dropped/          # 已放弃的规格
        └── cases/                # 测试用例
            ├── _index.yaml       # 用例索引
            ├── config.yaml       # Maestro 配置
            ├── CR-{id}/          # 进行中的用例
            ├── blessed/          # 可复用用例
            ├── archive/          # 已完成的用例
            └── dropped/          # 已放弃的用例
```

---

## 路线图

### Phase 1: E2E 测试闭环 ✅
- [x] `/dd-spec-test` 输出 Gherkin 格式
- [x] `/dd-test-case` 生成 Maestro YAML
- [x] blessed 版本化命名与引用管理

### Phase 2: VSCode 插件
- [ ] CR 状态面板
- [ ] 依赖图可视化

---

## 与其他工具的关系

本框架是 AI 开发工具链的**上游**，不是替代品：

| 工具 | 定位 | 与本框架关系 |
|------|------|-------------|
| AWS Kiro | 单项目开发辅助 | 本框架输出 Spec，Kiro 执行代码生成 |
| Cursor | AI 编程 IDE | 本框架输出 Spec，Cursor 执行实现 |
| GitHub Spec Kit | Spec 格式标准 | 本框架管理 Spec 的生命周期 |

### 独特价值

| 特性 | 本框架 | 其他工具 |
|------|--------|---------|
| RC 预览机制 | ✅ | ❌ |
| 双向依赖追踪 | ✅ 自动 | 手动或无 |
| Context Loading 分层 | ✅ | ❌ |
| 多产品管理 | ✅ | 单项目 |
| CR 生命周期 | ✅ 完整追踪 | 无或部分 |

---

<div align="center">

**如果这个项目对你有帮助，请给个 Star！**

</div>
