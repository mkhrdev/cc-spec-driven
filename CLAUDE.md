# CLAUDE.md - Spec-Driven Document-First Development Framework

## Overview

规格驱动文档先行的文档管理框架，用于管理多产品的需求文档。

**核心理念**：
- 文档是 Single Source of Truth
- 渐进式完善，先粗后细
- AI辅助维护，人类确认

**配置**：
- 变更ID前缀: `CR`
- 默认语言: `zh`

---

## 目录结构

```
spec/
├── CLAUDE.md                     # 本文件
│
└── products/
    └── {product}/
        ├── project.yaml          # 产品配置 (含 next_cr_id)
        ├── glossary.yaml         # 术语表 (人类维护)
        ├── overview.md           # 产品全景
        ├── features/             # 功能文档
        │   ├── {feature}.md      # 业务需求 (正式)
        │   ├── {feature}.rc-{id}.md    # 业务需求 (CR预览版)
        │   ├── {feature}.tech.md       # 技术共识 (正式)
        │   └── {feature}.tech.rc-{id}.md # 技术共识 (CR预览版)
        ├── changes/              # 变更记录
        │   ├── _index.yaml       # 变更索引
        │   ├── CR-{id}.md        # 进行中的变更
        │   ├── archive/          # 已完成的变更
        │   └── dropped/          # 已放弃的变更
        └── specs/                # 规格文件
            ├── _index.yaml       # 规格索引
            ├── CR-{id}.dev.md    # 开发规格
            ├── CR-{id}.test.md   # 测试规格
            ├── archive/          # 已完成的规格
            └── dropped/          # 已放弃的规格
```

---

## Context Loading

### 分层加载策略

为优化 token 使用，采用分层按需加载：

```
Level 0: 文件列表
└── glob features/*.md → 获取所有文件名

Level 1: Meta 信息
└── 读取每个文件的 YAML frontmatter (前 20 行)

Level 2: 结构信息
└── 读取所有二级标题 (## )

Level 3: 详细内容
└── 按需读取具体章节
```

### 加载决策

1. **始终加载**: `CLAUDE.md`, `project.yaml`, `glossary.yaml`
2. **始终加载 Level 0-1**: 所有 feature 文件的 meta
3. **按依赖加载 Level 2**: 当 feature.deps 包含目标功能时
4. **按需加载 Level 3**: 需要具体实现细节时

---

## Skills

### 核心 Skills

| Skill | 作用 |
|-------|------|
| `/dd-init` | 初始化产品 |
| `/dd-status` | 查看状态 |
| `/dd-update` | 创建/修改变更 |
| `/dd-confirm` | 确认变更 (生成 RC 预览) |
| `/dd-done` | 标记完成 (合并 RC 到正式) |
| `/dd-drop` | 放弃变更 |

### 辅助 Skills

| Skill | 作用 |
|-------|------|
| `/dd-check` | 综合检查 (console 输出) |
| `/dd-rebase` | 处理分支合并冲突 |
| `/dd-spec-dev` | 生成开发规格 (需 confirmed) |
| `/dd-spec-test` | 生成测试规格 (需 confirmed) |

---

## 工作流

### 状态流转

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

- `update → confirm`: `/dd-confirm`
- `confirm → update`: `/dd-update CR-{id}` (触发回退)
- `confirm → done`: `/dd-done`
- `* → drop`: `/dd-drop`

### 标准流程

```
1. /dd-update "描述"
   → AI分析影响范围
   → 创建 CR-{id}.md (draft)
   → 如涉及新功能，创建 {feature}.tech.md

2. 人类 review
   → 可能多轮调整
   → /dd-update CR-{id} "补充"

3. /dd-confirm CR-{id}
   → 生成 {feature}.rc-{id}.md 预览版
   → CR状态变 confirmed

4. (可选) 生成规格
   → /dd-spec-dev CR-{id}  (需 confirmed)
   → /dd-spec-test CR-{id} (需 confirmed)

5. /dd-done CR-{id}
   → 合并 .rc-{id}.md 到正式文档
   → 删除 RC 文件
   → 归档 CR 和 specs
```

### 放弃变更

```
/dd-drop CR-{id}
→ 删除 RC 文件 (如有)
→ 删除 spec 文件 (如有)
→ 移动 CR 到 dropped/
```

### 补充已实现功能

```
/dd-update "功能描述" --implemented
→ 只补文档，不需要 spec-dev
```

---

## 文档格式

### project.yaml

```yaml
name: "{product-name}"
description: ""
created: "{date}"
language: "zh"
repos:
  - name: frontend
    path: "../frontend"
next_cr_id: 1
```

### glossary.yaml

由人类维护，AI 识别新术语时仅 console 提醒。

```yaml
version: "1.0"
terms:
  - term: 用户
    definition: 系统的注册使用者
    aliases: [客户, 会员]
    related: [订单, 账户]
```

### overview.md

```markdown
# {Product Name}

{一句话描述}

## 模块
- {模块1}: {简述}
- {模块2}: {简述}

## Scope
- In: {在范围内}
- Out: {不在范围内}
```

### feature.md

```markdown
---
id: {uuid}
title: {Feature Name}
deps: [{依赖的feature}]
affects: [{影响的feature}]
updated: {date}
---

# {Feature Name}

## 概述
{2-3句话}

## 相关
- 依赖: [{feature}]
- 影响: [{feature}]

## 功能点
{具体描述}

## 边界
- {边界条件}

---
_更新: {date} | CR-{id}_
```

### feature.tech.md

技术共识文档，包含实现方案的多种可能性：

```markdown
---
id: {uuid}
feature: {feature-name}
cr: CR-{id}
updated: {date}
---

# {Feature} - 技术共识

## 涉及仓库
- {repo}: {影响描述}

## 技术决策

### 决策1: {决策名}
**方案A**: {方案描述}
- 优势: {优势}
- 劣势: {劣势}

**方案B**: {方案描述}
- 优势: {优势}
- 劣势: {劣势}

**待人类确认**: 生成 spec 时选择

## 接口约定
{已确定的接口定义}

## 注意事项
- {注意点}

---
_创建: {date} | CR-{id}_
```

### CR-{id}.md

```markdown
# CR-{id}: {标题}

---
uuid: {uuid}
status: draft | confirmed | done | dropped
type: change | implemented
created: {date}
updated: {date}
---

## 原始输入
{人类输入，原样保留}

## 澄清记录
{多轮澄清问答，如无则留空}

## 变更内容
{AI整理后的格式化内容}

## 影响范围
- 功能: [{feature}]
- 新建: [{feature}]

---
_创建: {date}_
_更新: {date}_
```

### _index.yaml (changes)

```yaml
changes:
  - id: CR-001
    uuid: {uuid}
    title: 用户登录功能
    status: done
    created: 2025-01-01
    updated: 2025-01-04
    features: [login, auth]
```

### _index.yaml (specs)

```yaml
specs:
  - cr: CR-001
    dev: true
    test: true
    status: archived
```

---

## 分支合并

当分支合并遇到文档冲突时：

```
/dd-rebase CR-{id}
→ AI提取变更的原始意图
→ 对比当前主线文档
→ 识别冲突类型（概念/指标/顺序/范围）
→ 提示冲突点和建议
→ 人确认后重新应用变更
```

核心思路：重新应用变更意图，而非文本合并。

---

## 版本控制

- 日常操作用变更ID (CR-001, CR-002...)
- CR-ID 在产品内自增（存于 project.yaml）
- CR 内部使用 UUID 标识（支持跨分支识别）
- Git管理文档历史
- 需要旧版本时 checkout 对应 commit
