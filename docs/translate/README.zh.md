<div align="center">

# cc-spec-driven

**Claude Code Plugin · 规格驱动 · 文档先行**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)](https://docs.anthropic.com/en/docs/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

[English](https://github.com/mkhrdev/cc-spec-driven/blob/main/README.md) · [中文](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.zh.md)

</div>

一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 插件，用于管理需求文档、追踪变更、输出规格，让下游工具基于高质量 Spec 执行代码生成，并完成 E2E 测试。

---

## 安装

```bash
# 安装
claude plugins:install mkhrdev/cc-spec-driven

# 更新
claude plugins:update cc-spec-driven
```

安装后，在 Claude Code 中输入 `/dd-` 即可看到所有命令。

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

## 完整使用指南

<!-- GUIDE:concepts:15 -->
### 核心概念

| 术语 | 说明 |
|------|------|
| CR (Change Request) | 变更请求，唯一标识一次需求变更 |
| RC (Release Candidate) | 预览版文档，confirm 后生成，done 后合并 |
| feature | 功能文档，描述一个独立的业务功能 |
| deps | 依赖关系：我依赖哪些功能 |
| affects | 被依赖关系：哪些功能依赖我 |
| blessed | 已验证可复用的测试用例 |
<!-- /GUIDE:concepts -->

<!-- GUIDE:workflow:21 -->
### 状态流转

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

- `draft`: 初始状态，可编辑
- `confirmed`: 已确认，生成了 RC 预览
- `done`: 已完成，RC 合并到正式文档
- `dropped`: 已放弃
<!-- /GUIDE:workflow -->

<!-- GUIDE:dd-init:30 -->
### /dd-init

初始化产品目录结构。

**用法**：
```
/dd-init <product-name>
```

**场景**：

| 场景 | 输入 | 行为 |
|------|------|------|
| 正常初始化 | `/dd-init my-app` | 创建完整目录结构和初始文件 |
| 产品已存在 | `/dd-init my-app` | 拒绝，提示产品已存在 |
| 无效名称 | `/dd-init "my app"` | 拒绝，提示名称格式要求 |

**生成文件**：
- `project.yaml` - 产品配置（next_cr_id: 1）
- `glossary.yaml` - 术语表
- `overview.md` - 产品全景
- `features/_deps.yaml` - 依赖图
- `changes/_index.yaml` - 变更索引
- `specs/_index.yaml` - 规格索引
- `cases/_index.yaml` - 用例索引
- `cases/config.yaml` - Maestro 配置
<!-- /GUIDE:dd-init -->

<!-- GUIDE:dd-status:40 -->
### /dd-status

查看产品状态和推荐操作。

**用法**：
```
/dd-status              # 列出所有产品
/dd-status <product>    # 指定产品详情
```

**场景**：

| 场景 | 输出 |
|------|------|
| 无产品 | 提示无产品，建议 `/dd-init` |
| 多产品 | 产品列表 + 各自变更统计 |
| 单产品 | 功能列表、RC 列表、变更状态、推荐操作 |
| 发现 stale RC | 标记 ⚠️ stale，建议 `/dd-rebase` |

**推荐下一步**：

| 当前状态 | 推荐操作 |
|----------|---------|
| 无 draft CR | `/dd-update "描述"` |
| 有 draft CR | `/dd-confirm CR-{id}` |
| 有 confirmed (无 spec) | `/dd-spec-dev` 或 `/dd-spec-test` |
| 有 confirmed (有 spec) | `/dd-done CR-{id}` |
| 有 stale RC | `/dd-rebase CR-{id}` |

**RC 过期检测**：RC 的 `updated` < 正式文档的 `updated` → stale
<!-- /GUIDE:dd-status -->

<!-- GUIDE:dd-update:75 -->
### /dd-update

创建或修改变更请求。

**用法**：
```
/dd-update "<描述>"                    # 创建新变更
/dd-update "<描述>" --implemented      # 已实现功能（跳过 spec-dev）
/dd-update "<描述>" --bootstrap        # 冷启动（直接创建 feature.md）
/dd-update <CR-id>                     # 修改已有变更
/dd-update <CR-id> "<补充描述>"        # 追加内容
```

**创建新变更场景**：

| 场景 | 行为 |
|------|------|
| 正常创建 | 生成 CR-{id}，分析依赖，多轮澄清 |
| 描述模糊 | 触发多轮澄清，要求具体化 |
| 涉及新功能 | 创建 CR + feature.tech.md |
| 涉及现有功能 | 分析影响范围 |
| 发现依赖变更 | 记录到 CR 的 `## 依赖变更` |
| 术语未定义 | console 提醒，不阻断 |

**--bootstrap 场景**：

| 场景 | 行为 |
|------|------|
| 冷启动新功能 | 直接创建 feature.md，无 CR |
| 冷启动已有功能 | 更新现有 feature.md |
| 完成后 | 建议执行 `/dd-check` |

**修改已有变更场景**：

| CR 状态 | 行为 |
|---------|------|
| draft | 直接追加内容 |
| confirmed | ⚠️ 警告后回退：删除 RC/spec，状态→draft |
| done | ❌ 拒绝：已归档 |
| dropped | ❌ 拒绝：已放弃 |

**confirmed 回退警告**：
```
警告: CR-{id} 已确认，修改将触发:
- 状态回退到 draft
- 删除 RC 文件和 spec 文件
是否继续? (y/n)
```

**影响范围分析**：
```
影响范围分析:
- 直接影响: profile (deps login)
- 间接影响: checkout (通过 profile)
  ↳ 建议 review 是否需要更新
```
<!-- /GUIDE:dd-update -->

<!-- GUIDE:dd-confirm:55 -->
### /dd-confirm

确认变更，生成 RC 预览版。

**用法**：
```
/dd-confirm <CR-id>
/dd-confirm <CR-id> --dry-run    # 仅预览，不创建
```

**前置条件**：状态必须是 `draft`

**场景**：

| 场景 | 行为 |
|------|------|
| 正常确认 | 生成 RC 文件，状态→confirmed |
| --dry-run | 仅输出将生成的文件 |
| 状态 confirmed | ❌ 已确认 |
| 状态 done | ❌ 已归档 |
| 状态 dropped | ❌ 已放弃 |
| CR 不存在 | ❌ CR 不存在 |
| 发现范围外依赖 | ⚠️ 自动扩展 CR，退出，要求重新执行 |

**范围外依赖处理**：

当 `## 依赖变更` 涉及的 feature 不在 `## 影响范围` 中：
1. 自动更新 CR（扩展影响范围）
2. 输出提示并退出
3. 用户 review 后重新执行

```
⚠️ 发现范围外依赖，已更新 CR-{id}:

影响范围新增:
- checkout (需更新 affects)

CR 已更新，请 review 后重新执行:
/dd-confirm CR-{id}
```

**生成 RC**：

| 情况 | 处理 |
|------|------|
| 功能已存在 | 加载 feature.md，应用变更，保存为 .rc-{id}.md |
| 新建功能 | 从 CR 提取信息，创建 .rc-{id}.md |
<!-- /GUIDE:dd-confirm -->

<!-- GUIDE:dd-done:55 -->
### /dd-done

标记变更完成，合并 RC 到正式文档。

**用法**：
```
/dd-done <CR-id>
/dd-done <CR-id> --dry-run    # 仅预览，不执行
```

**前置条件**：状态必须是 `confirmed`

**场景**：

| 场景 | 行为 |
|------|------|
| 正常完成 | 合并 RC、归档、更新索引 |
| --dry-run | 仅输出将执行的操作 |
| 状态 draft | ❌ 请先 `/dd-confirm` |
| 状态 done | ❌ 已归档 |
| 状态 dropped | ❌ 已放弃 |
| 有 spec 文件 | 一并归档到 specs/archive/ |
| 有测试用例 | 一并归档到 cases/archive/ |
| 被其他 CR 引用 | 提升用例到 blessed/ |

**执行步骤**：
1. 合并 RC 到正式文档（移除 `rc_for`，更新页脚）
2. 删除 RC 文件
3. 重建 `_deps.yaml`
4. 更新 CR 状态→done，添加 `completed`
5. 归档 CR→changes/archive/
6. 归档 spec→specs/archive/（如存在）
7. 归档用例→cases/archive/（如存在）
8. 提升可复用用例到 blessed/（如有 referenced_by）
9. 更新所有索引

**blessed 提升**：
- 条件：`cases/_index.yaml` 中有 `referenced_by`
- 命名：`{feature}.{cr-id}.{platform}.yaml`
- 注释：`# Promoted from: CR-{id} | {date}`
<!-- /GUIDE:dd-done -->

<!-- GUIDE:dd-drop:45 -->
### /dd-drop

放弃变更。

**用法**：
```
/dd-drop <CR-id>
/dd-drop <CR-id> "<原因>"
```

**场景**：

| CR 状态 | 行为 |
|---------|------|
| draft | 直接放弃，移动到 dropped/ |
| confirmed | 删除 RC/spec，再放弃 |
| done | ❌ 已归档，用 git revert |
| dropped | ❌ 已放弃 |

**确认提示**：
```
即将放弃 CR-{id}:
- 删除 RC 文件: features/login.rc-{id}.md
- 删除 spec: specs/CR-{id}.dev.md
- 移动用例: cases/CR-{id}/ → cases/dropped/

是否继续? (y/n)
```

**引用清理**：

| 情况 | 处理 |
|------|------|
| 本 CR 引用了其他 CR | 清理被引用方的 `referenced_by` |
| 其他 CR 引用了本 CR | ⚠️ 警告，清理引用方的 `refs` |

**恢复已放弃**：手动从 dropped/ 移回，修改 status 为 draft
<!-- /GUIDE:dd-drop -->

<!-- GUIDE:dd-check:50 -->
### /dd-check

检查文档一致性（仅 console 输出，不阻断）。

**用法**：
```
/dd-check [product]
/dd-check [product] --scope=<docs|cases|all>
/dd-check [product] --type=<check-type>
```

**--scope**：

| 值 | 检查内容 |
|----|---------|
| docs | 需求文档（默认） |
| cases | 测试用例 |
| all | 全部 |

**docs 检查类型**：

| --type | 内容 |
|--------|------|
| glossary | 术语表与文档一致性 |
| format | 文档格式（章节、frontmatter） |
| deps | 依赖关系（存在性、循环、双向、_deps.yaml） |
| refs | 引用关系（CR↔feature、孤立文档） |
| status | 状态一致性（长期 draft、索引同步） |

**cases 检查类型**：

| --type | 内容 |
|--------|------|
| runflow | runFlow 路径有效性 |
| index | 索引一致性（refs/referenced_by） |
| blessed | blessed 版本检查 |

**输出格式**：
```
=== 文档检查: my-app ===

[严重] 必须修复:
  ✗ deps: login.deps 包含不存在的 feature

[警告] 建议修复:
  ⚠ format: profile.md 缺少 ## 边界

检查完成: 1 严重 | 1 警告 | 0 信息
```
<!-- /GUIDE:dd-check -->

<!-- GUIDE:dd-rebase:45 -->
### /dd-rebase

处理分支合并冲突，重新应用变更意图。

**用法**：
```
/dd-rebase <CR-id>
```

**场景**：

| 触发条件 | 行为 |
|---------|------|
| `/dd-status` 显示 stale | 重新应用变更意图 |
| git merge 后文档冲突 | 基于意图重新生成 |

**冲突类型**：

| 类型 | 说明 |
|------|------|
| 概念冲突 | 同一概念的不同定义 |
| 指标冲突 | 数值/限制的不同设定 |
| 顺序冲突 | 流程/步骤的不同排列 |
| 范围冲突 | 功能边界的不同界定 |

**执行流程**：
1. 加载 CR（提取变更意图）
2. 加载当前主线文档
3. 意图提取（优先级：原始输入 > 变更内容 > 澄清记录）
4. 冲突分析
5. 输出分析报告
6. **人类确认**处理方案
7. 重新生成

**注意**：Rebase 后 CR 保持原状态，会添加"Rebase记录"章节
<!-- /GUIDE:dd-rebase -->

<!-- GUIDE:dd-spec-dev:45 -->
### /dd-spec-dev

生成开发规格。

**用法**：
```
/dd-spec-dev <CR-id>
```

**前置条件**：
- 状态必须是 `confirmed`
- type=implemented 时跳过

**场景**：

| 场景 | 行为 |
|------|------|
| 正常生成 | 生成 specs/CR-{id}.dev.md |
| 状态 draft | ❌ 请先 `/dd-confirm` |
| type=implemented | ⚠️ 跳过 |
| spec 已存在 | 询问覆盖 |
| tech.md 有待决策项 | 交互确认方案 |
| 外部仓库不可达 | 标注"待确认" |

**Spec 结构**：
```markdown
# 开发规格: CR-{id}

## 概述
## 前置条件
## 技术约束
## 技术决策
## 实现任务
### Task N: {任务名}
**目标**: {用户故事}
**上下文**: 相关文件、接口
**验收条件**: [ ] 条件
**实现提示**: 复用xxx
## 实现顺序
## 参考资料
```
<!-- /GUIDE:dd-spec-dev -->

<!-- GUIDE:dd-spec-test:50 -->
### /dd-spec-test

生成测试规格（Gherkin 格式）。

**用法**：
```
/dd-spec-test <CR-id>
/dd-spec-test --init    # 基于所有 feature 生成初始测试规格
```

**前置条件**：状态必须是 `confirmed`（--init 例外）

**场景**：

| 场景 | 行为 |
|------|------|
| 正常生成 | 生成 specs/CR-{id}.test.md |
| --init | 基于所有 feature 生成 INIT.test.md |
| 状态错误 | ❌ 请先 `/dd-confirm` |
| spec 已存在 | 询问覆盖 |

**TC 编号规则**：

| 范围 | 用途 |
|------|------|
| TC-001~099 | 主流程测试 |
| TC-100~199 | 边界测试 |
| TC-200~299 | 异常测试 |

**Given/When/Then 原则**：
- 使用业务语言，禁止 UI 细节
- Given 描述状态，不描述如何达到
- When 描述意图，不描述具体操作
- Then 描述可验证结果

**Spec 结构**：
```markdown
# 测试规格: CR-{id}

## 测试范围
涉及功能: login, profile
平台: [web] [ios] [android]

## 测试用例
### TC-001: 用户成功登录
**优先级**: P0
**Given**: 用户处于未登录状态
**When**: 用户输入正确凭据
**Then**: 显示首页

## 回归检查点
- [profile] 登录后能访问个人资料
```
<!-- /GUIDE:dd-spec-test -->

<!-- GUIDE:dd-test-case:50 -->
### /dd-test-case

生成 Maestro 测试用例。

**用法**：
```
/dd-test-case <CR-id>
/dd-test-case <CR-id> --platform=<ios|android|web|all>
/dd-test-case <CR-id> --dry-run
/dd-test-case <CR-id> --force
```

**前置条件**：
- 状态必须是 `confirmed`
- 必须存在 `specs/CR-{id}.test.md`

**场景**：

| 场景 | 行为 |
|------|------|
| 正常生成 | 生成 Maestro YAML |
| --platform=ios | 仅生成 iOS |
| --dry-run | 仅输出文件列表 |
| --force | 跳过覆盖询问 |
| 文件已存在 | 询问覆盖/跳过/取消 |
| 无 test spec | ❌ 请先 `/dd-spec-test` |

**依赖复用**：

| 场景 | 处理 |
|------|------|
| 依赖功能有 blessed 用例 | runFlow 引用最新版本 |
| 依赖功能有进行中用例 | runFlow 引用 |
| 依赖功能无用例 | 生成内联前置步骤 |

**回归测试**：
- 解析 `## 回归检查点`
- 查找已有用例
- 生成带 `regression` tag 的 runFlow

**输出**：
- 文件：`cases/CR-{id}/{feature}.{platform}.yaml`
- 报告：`cases/CR-{id}/REPORT.md`
- 摘要：文件数、runFlow 数、回归数、TODO 数
<!-- /GUIDE:dd-test-case -->

<!-- GUIDE:file-formats:55 -->
### 文件格式参考

**project.yaml**：
```yaml
name: my-app
description: 我的应用
created: 2026-01-07
language: zh
repos: []
next_cr_id: 1
```

**feature.md frontmatter**：
```yaml
---
id: {uuid}
title: 用户登录
deps: [auth]
affects: [profile, checkout]
updated: 2026-01-07
---
```

**CR-{id}.md frontmatter**：
```yaml
---
uuid: {uuid}
status: draft | confirmed | done | dropped
type: change | implemented
created: 2026-01-07
updated: 2026-01-07
completed: 2026-01-08    # done 时
dropped: 2026-01-08      # dropped 时
---
```

**_deps.yaml**：
```yaml
# 自动生成，禁止手动编辑
updated: 2026-01-07
graph:
  login:
    deps: [auth]
    affects: [profile]
```

**cases/_index.yaml**：
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
### 进阶技巧

**Context Loading 分级**：

| 级别 | 内容 | 触发时机 |
|------|------|---------|
| Level 0 | 文件列表 | 始终 |
| Level 1 | frontmatter | 目标 + 直接依赖 |
| Level 2 | 章节标题 | deps/affects 涉及 |
| Level 3 | 详细内容 | 需修改时 |

**始终加载**：project.yaml, glossary.yaml, overview.md, _deps.yaml

**依赖管理**：
- deps: 我依赖谁
- affects: 谁依赖我
- _deps.yaml: 全局依赖视图，影响范围快速分析

**测试用例复用**：
- blessed/: 已验证可复用用例
- runFlow: 自动引用依赖功能的用例
- 优先使用 blessed/ 中最新版本
<!-- /GUIDE:advanced -->

<!-- GUIDE:faq:25 -->
### FAQ

**Q: 如何恢复已放弃的 CR？**
A: 手动从 dropped/ 移回，修改 status 为 draft

**Q: confirmed 状态能修改吗？**
A: 可以，`/dd-update` 会警告后回退到 draft

**Q: 如何处理 stale RC？**
A: 执行 `/dd-rebase` 重新应用变更意图

**Q: 如何跳过开发规格？**
A: 使用 `--implemented` 标记已实现功能

**Q: spec 生成后可以修改吗？**
A: 可以手动修改，建议通过 `/dd-update` 触发重新生成
<!-- /GUIDE:faq -->

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
