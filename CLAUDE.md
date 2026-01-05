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
        └── specs/                # 规格文件
            ├── _index.yaml       # 规格索引
            ├── CR-{id}.dev.md    # 开发规格
            ├── CR-{id}.test.md   # 测试规格
            ├── archive/          # 已完成的规格
            └── dropped/          # 已放弃的规格
```

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

### Command 架构

所有 `/dd-*` 命令共享 `.claude/commands/_base.md` 中的公共定义。
各命令文件只包含用法和执行步骤，格式定义引用 `_base.md`。

---

## 工作流

### 状态流转

```
draft → confirmed → done (归档)
  │         │
  └────┬────┘
       ↓
    dropped
```

### 标准流程

```
1. /dd-update "描述"     → 创建 CR (draft)
2. 人类 review           → 可能多轮调整
3. /dd-confirm CR-{id}   → 生成 RC 预览 (confirmed)
4. /dd-spec-dev|test     → 生成规格 (可选)
5. /dd-done CR-{id}      → 合并到正式文档
```

### 依赖变更流转

```
/dd-update   →  发现依赖变更，记录到 CR
/dd-confirm  →  检查范围外依赖，必要时扩展CR后退出
/dd-done     →  合并RC到正式文档，更新 _deps.yaml
```

### 冷启动模式

```
/dd-update "描述" --bootstrap
→ 直接创建 feature.md，不生成 CR
→ 完成后建议执行 /dd-check
```

---

## Context Loading & 文档格式

→ 详见 `.claude/commands/_base.md`

---

## 注意事项

以下是经过深思熟虑的设计决策：

1. **单分支单 RC**: 文档用 git 管理，如需合并用 rebase

2. **隐式状态回退**: `/dd-update <CR-id>` 对 confirmed 状态会触发回退

3. **Spec 生成后的事不归此 repo 管**: 只负责文档和规格

4. **术语仅 console 提醒**: glossary.yaml 由人类维护

5. **按需改善一致性检查**: 索引同步、依赖验证等按需完善

6. **版本管理与产品发布解绑**: CR 版本追踪与对外发布版本无关

### 范围边界

- 此框架**只管文档**，开发进度追踪、代码同步不在范围内
- 跨产品依赖不在设计范围内，每个产品独立

### 工具 vs 流程

- `/dd-check` 是给人类的**工具**，不是强制的流程步骤
- 一致性检查不阻断操作，人类决定是否修复

### 演进路线

→ 详见 `TODO.md`：设计原则、范围边界、未来改进方向
