# /dd-drop - 放弃变更

> 公共定义见 `_base.md`

## 用法

```
/dd-drop <CR-id>
/dd-drop <CR-id> "<原因>"
```

## 适用状态

| 状态 | 处理 |
|------|------|
| draft | 直接放弃 |
| confirmed | 删除 RC 和 spec，然后放弃 |
| done | 不可放弃（已归档，用 git revert） |

## 执行步骤

1. **加载 CR**: 验证状态
2. **确认放弃**: 显示将执行的操作，等待确认
3. **删除 RC 文件**: `features/*.rc-{id}.md`（如 confirmed）
4. **删除 spec 文件**: `specs/CR-{id}.*.md`（如存在）
5. **更新 CR 状态**: status → `dropped`, 添加 `dropped: {date}`, `drop_reason`
6. **移动 CR**: 到 `changes/dropped/`
7. **更新索引**: `_index.yaml`

## 恢复已放弃的变更

手动将文件从 `changes/dropped/` 移回 `changes/`，修改 status 为 `draft`，移除 dropped 相关字段。
