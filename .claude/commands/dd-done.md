# /dd-done - 标记变更完成

将 RC 预览文档合并到正式文档，归档变更记录和相关规格。

## 用法

```
/dd-done <CR-id>
```

注意：每次只处理单个 CR，不支持批量操作。

## 前置条件

- 变更状态必须是 `confirmed`
- 相关工作已完成

## 执行步骤

1. **加载变更记录**
   - `changes/{CR-id}.md`
   - 解析 YAML frontmatter

2. **验证状态**
   - 确认当前状态是 `confirmed`
   - 如果是 `draft`，**拒绝执行**并提示:
     ```
     错误: CR-{id} 状态为 draft，不能直接标记完成。

     请先执行:
     /dd-confirm CR-{id}
     ```
   - 如果是 `done`，提示已完成
   - 如果是 `dropped`，提示已放弃

3. **合并 RC 到正式文档**

   查找所有关联的 RC 文件:
   ```
   features/*.rc-{id}.md
   features/*.tech.rc-{id}.md
   ```

   对于每个 RC 文件:

   **如果对应的正式文档存在**:
   - 用 RC 内容替换正式文档
   - 移除 frontmatter 中的 `rc_for` 字段
   - 更新页脚为正式格式 `_更新: {date} | CR-{id}_`

   **如果是新建功能** (正式文档不存在):
   - 将 RC 文件重命名为正式文档
   - 移除 frontmatter 中的 `rc_for` 字段
   - 更新页脚格式

4. **删除 RC 文件**
   - 确保所有 RC 内容已合并后删除原 RC 文件

5. **更新变更状态**
   - 修改 frontmatter status 为 `done`
   - 添加 `completed: {date}` 字段
   - 更新页脚 `_完成: {date}_`

6. **归档变更记录**
   - 将 `changes/{CR-id}.md` 移动到 `changes/archive/{CR-id}.md`

7. **归档规格文件**（如存在）
   - 将 `specs/CR-{id}.dev.md` 移动到 `specs/archive/CR-{id}.dev.md`
   - 将 `specs/CR-{id}.test.md` 移动到 `specs/archive/CR-{id}.test.md`

8. **更新索引** (YAML 格式)

   更新 `changes/_index.yaml`:
   ```yaml
   changes:
     # ... 现有记录
     - id: CR-{id}
       uuid: {uuid}
       title: {标题}
       status: done
       created: {created_date}
       updated: {date}
       completed: {date}
       features: [{feature1}, {feature2}]
   ```

   更新 `specs/_index.yaml` (如有规格):
   ```yaml
   specs:
     # ... 现有记录
     - cr: CR-{id}
       dev: true
       test: true
       status: archived
   ```

9. **输出确认**
   ```
   已完成变更: CR-{id}

   合并的文档:
   - features/login.md ← login.rc-{id}.md
   - features/payment.md ← payment.rc-{id}.md (新建)

   归档的文件:
   - changes/archive/CR-{id}.md
   - specs/archive/CR-{id}.dev.md
   - specs/archive/CR-{id}.test.md

   文档已是最新状态。
   ```

## 状态流转

```
           ┌──────────────┐
           │              │
           ▼              │
draft ──→ confirmed ──→ done (归档)
  ▲          │
  │          │ (回退)
  └──────────┘
           │
           ▼
        dropped (放弃)
```

## RC 合并详解

### 合并前
```
features/
├── login.md                # 正式文档 (旧版本)
├── login.rc-003.md         # CR-003 的预览版
├── login.tech.md           # 正式技术共识
├── login.tech.rc-003.md    # CR-003 的技术共识预览版
└── payment.rc-003.md       # CR-003 新建功能的预览版
```

### 合并后
```
features/
├── login.md                # 正式文档 (已更新)
├── login.tech.md           # 正式技术共识 (已更新)
└── payment.md              # 新建的正式文档
```

### Frontmatter 转换

RC 版本:
```yaml
---
id: uuid-xxx
title: 用户登录
deps: [auth]
affects: [dashboard]
updated: 2025-01-04
rc_for: CR-003
---
```

正式版本:
```yaml
---
id: uuid-xxx
title: 用户登录
deps: [auth]
affects: [dashboard]
updated: 2025-01-04
---
```

### 页脚转换

RC 版本:
```
_预览版: CR-003 | 生成: 2025-01-03_
```

正式版本:
```
_更新: 2025-01-04 | CR-003_
```

## 索引文件格式

### changes/_index.yaml

```yaml
changes:
  - id: CR-001
    uuid: a1b2c3d4-...
    title: 用户登录功能
    status: done
    created: 2025-01-01
    updated: 2025-01-04
    completed: 2025-01-04
    features: [login, auth]
  - id: CR-002
    uuid: e5f6g7h8-...
    title: 支付集成
    status: confirmed
    created: 2025-01-03
    updated: 2025-01-03
    features: [payment]
```

### specs/_index.yaml

```yaml
specs:
  - cr: CR-001
    dev: true
    test: true
    status: archived
  - cr: CR-002
    dev: true
    test: false
    status: active
```
