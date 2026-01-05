# DD 框架公共定义

本文件定义所有 `/dd-*` 命令共享的策略和格式。各命令应引用此文件而非重复定义。

---

## Context Loading 策略

```
Level 0: 文件列表
└── glob features/*.md（排除 .rc-*.md, .tech.md）

Level 1: Meta 信息
└── 读取每个文件的 YAML frontmatter（前 20 行）

Level 2: 结构信息
└── 读取所有 ## 标题

Level 3: 详细内容
└── 按需读取具体章节
```

**始终加载**: `project.yaml`, `glossary.yaml`, `overview.md`, `features/_deps.yaml`
**始终 Level 0**: 所有 feature 文件列表
**Level 1**: 仅加载【目标feature + _deps.yaml中直接依赖】的frontmatter

**Level 2 触发**: deps/affects 中的功能、CR 影响范围内的功能
**Level 3 触发**: 需修改具体章节、需理解技术细节

---

## 状态流转

```
draft → confirmed → done (归档)
  │         │
  └────┬────┘
       ↓
    dropped
```

- `update → confirm`: `/dd-confirm`
- `confirm → update`: `/dd-update CR-{id}` 触发回退
- `confirm → done`: `/dd-done`
- `* → dropped`: `/dd-drop`

---

## 文件格式速查

### feature.md

```yaml
---
id: {uuid}
title: {标题}
deps: [{依赖}]
affects: [{影响}]
updated: {date}
---
```

章节: `## 概述` `## 相关` `## 功能点` `## 边界`
页脚: `_更新: {date} | CR-{id}_`

### feature.rc-{id}.md

同 feature.md，额外字段:
```yaml
rc_for: CR-{id}
```
页脚: `_预览版: CR-{id} | 生成: {date}_`

### feature.tech.md

```yaml
---
id: {uuid}
feature: {feature-name}
cr: CR-{id}
updated: {date}
---
```

章节: `## 涉及仓库` `## 技术决策` `## 接口约定` `## 注意事项`

### CR-{id}.md

```yaml
---
uuid: {uuid}
status: draft | confirmed | done | dropped
type: change | implemented
created: {date}
updated: {date}
---
```

章节: `## 原始输入` `## 澄清记录` `## 变更内容` `## 影响范围` `## 依赖变更`

### 依赖变更记录格式

```markdown
## 依赖变更
- {feature}.deps: +[added] -[removed]
- {feature}.affects: +[added] -[removed]
```

---

## 索引文件格式

### changes/_index.yaml

```yaml
changes:
  - id: CR-001
    uuid: {uuid}
    title: {标题}
    status: done | confirmed | draft | dropped
    created: {date}
    updated: {date}
    completed: {date}    # done 时
    dropped: {date}      # dropped 时
    features: [{feature}]
```

### specs/_index.yaml

```yaml
specs:
  - cr: CR-001
    dev: true | false
    test: true | false
    status: active | archived
```

### features/_deps.yaml

依赖图索引，由 `/dd-done` 自动维护，`/dd-check` 验证一致性。

```yaml
# 自动生成，禁止手动编辑
# 更新时机: /dd-done
updated: {date}

graph:
  {feature}:
    deps: [{依赖的feature}]
    affects: [{影响的feature}]
```

**作用**：
- 提供全局依赖视图，减少 Level 1 加载量
- 无需读取每个文件的 frontmatter 来构建依赖图

---

## 目录结构

```
products/{product}/
├── project.yaml
├── glossary.yaml
├── overview.md
├── features/
│   ├── _deps.yaml           # 依赖图索引（自动维护）
│   ├── {feature}.md
│   ├── {feature}.rc-{id}.md
│   ├── {feature}.tech.md
│   └── {feature}.tech.rc-{id}.md
├── changes/
│   ├── _index.yaml
│   ├── CR-{id}.md
│   ├── archive/
│   └── dropped/
└── specs/
    ├── _index.yaml
    ├── CR-{id}.dev.md
    ├── CR-{id}.test.md
    ├── archive/
    └── dropped/
```

---

## 通用行为

### 术语检查
发现未定义术语时，**仅 console 输出**，不记录到 CR:
```
⚠️ 发现未定义术语: "xxx"
建议: 请考虑更新 glossary.yaml
```

### 状态验证提示模板

**done 状态**:
```
错误: CR-{id} 已完成归档。
```

**dropped 状态**:
```
错误: CR-{id} 已放弃。
```

**需要 confirmed**:
```
错误: CR-{id} 状态为 draft，无法执行此操作。
请先执行: /dd-confirm CR-{id}
```

---

## 通用输出格式

### 成功操作
```
已{操作}: {目标}

{详情列表}

下一步:
- {建议命令}
```

### 错误
```
错误: {原因}
建议: {修复建议}
```

### 依赖范围扩展（confirm 专用）

当 `/dd-confirm` 发现依赖变更涉及 CR 范围外的文档时，自动扩展 CR 后退出：

```
⚠️ 发现范围外依赖，已更新 CR-{id}:

影响范围新增:
- {feature} (需更新 {deps|affects})

依赖变更新增:
- {feature}.{deps|affects}: +[{added}]

CR 已更新，请 review 后重新执行:
/dd-confirm CR-{id}
```

**流程**:
1. 检查 CR 中 `## 依赖变更` 涉及的所有 feature
2. 如果某 feature 不在 `## 影响范围` 中 → 扩展范围
3. 补充对应的双向依赖变更
4. 更新 CR 文件
5. 输出提示并退出（不生成 RC）

---

## 影响范围分析

`/dd-update` 时输出影响分析（仅供参考，不阻断）：

```
影响范围分析:
- 直接影响: {feature} (deps {target})
- 间接影响: {feature} (通过 {中间feature})
  ↳ 建议 review 是否需要更新
```

**计算方式**:
- 直接影响: _deps.yaml 中 deps 包含目标 feature 的所有 feature
- 间接影响: 直接影响的 feature 被哪些 feature 依赖（仅展示一层）
