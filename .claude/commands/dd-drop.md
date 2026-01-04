# /dd-drop - 放弃变更

放弃变更，删除相关 RC 和 spec 文件，将变更记录移动到 dropped 目录。

## 用法

```
/dd-drop <CR-id>
/dd-drop <CR-id> "<原因>"
```

## 适用状态

| 状态 | 处理方式 |
|------|---------|
| draft | 直接放弃，移动到 dropped/ |
| confirmed | 删除 RC 文件和 spec，移动到 dropped/ |
| done | 不可放弃（已归档） |

## 执行步骤

1. **加载变更记录**
   - `changes/{CR-id}.md`
   - 解析 YAML frontmatter

2. **验证状态**
   - 如果是 `done`，**拒绝执行**并提示:
     ```
     错误: CR-{id} 已完成归档，不能放弃。

     如需撤销已完成的变更，请使用 git revert。
     ```

3. **确认放弃**

   **draft 状态**:
   ```
   确认放弃 CR-{id}?

   将执行:
   - 移动 CR 到 dropped/

   是否继续? (y/n)
   ```

   **confirmed 状态**:
   ```
   确认放弃 CR-{id}?

   将执行:
   - 删除 RC 文件:
     - features/login.rc-{id}.md
     - features/login.tech.rc-{id}.md
   - 删除 spec 文件 (如有):
     - specs/CR-{id}.dev.md
     - specs/CR-{id}.test.md
   - 移动 CR 到 dropped/

   注意: 由于使用 RC 机制，正式文档未被修改，无需回滚。

   是否继续? (y/n)
   ```

4. **记录放弃原因**（可选）
   - 如果提供了原因参数，记录到 CR
   - 如果没有提供，可选择是否补充原因

5. **删除 RC 文件**（如 confirmed 状态）
   - 查找并删除 `features/*.rc-{id}.md`
   - 查找并删除 `features/*.tech.rc-{id}.md`

6. **删除 spec 文件**（如存在）
   - 删除 `specs/CR-{id}.dev.md`
   - 删除 `specs/CR-{id}.test.md`

7. **更新变更状态**
   - 修改 frontmatter status 为 `dropped`
   - 添加 `dropped: {date}` 字段
   - 如有原因，添加 `drop_reason: {reason}` 字段
   - 更新页脚 `_放弃: {date}_`

8. **移动变更记录**
   - 将 `changes/{CR-id}.md` 移动到 `changes/dropped/{CR-id}.md`

9. **更新索引** (YAML 格式)

   更新 `changes/_index.yaml`:
   ```yaml
   changes:
     # ... 现有记录
     - id: CR-{id}
       uuid: {uuid}
       title: {标题}
       status: dropped
       created: {created_date}
       updated: {date}
       dropped: {date}
       drop_reason: {reason}  # 可选
       features: [{feature1}, {feature2}]
   ```

10. **输出确认**
    ```
    已放弃变更: CR-{id}

    删除的文件:
    - features/login.rc-{id}.md
    - features/login.tech.rc-{id}.md
    - specs/CR-{id}.dev.md
    - specs/CR-{id}.test.md

    移动到:
    - changes/dropped/CR-{id}.md

    正式文档未受影响。
    ```

## 状态流转

```
           ┌──────────────┐
           │              │
           ▼              │
draft ──→ confirmed ──→ done (归档)
  │          │
  │          │
  └────┬─────┘
       │
       ▼
    dropped (放弃)
```

## RC 机制的优势

使用 RC 机制后，drop 操作变得更加安全：

| 场景 | 旧方式 | RC 方式 |
|------|--------|---------|
| draft 状态 drop | 无影响 | 无影响 |
| confirmed 状态 drop | 需要 git revert 正式文档 | 只需删除 RC 文件 |
| 正式文档状态 | 可能已被修改 | 保持原样 |

## 恢复已放弃的变更

如果需要恢复已放弃的变更：

1. 手动将文件从 `changes/dropped/` 移回 `changes/`
2. 修改 frontmatter status 为 `draft`
3. 移除 `dropped` 和 `drop_reason` 字段
4. 继续正常工作流

## 示例

```
/dd-drop CR-003
/dd-drop CR-003 "需求变更，此功能不再需要"
/dd-drop CR-005 "与 CR-006 合并"
```
