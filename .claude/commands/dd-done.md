# /dd-done - 标记变更完成

> 公共定义见 `_base.md`

## 用法

```
/dd-done <CR-id>
```

每次只处理单个 CR。

## 前置条件

状态必须是 `confirmed`，否则提示先执行 `/dd-confirm`。

---

## 执行步骤

1. **加载 CR**: 验证状态
2. **合并 RC 到正式文档**:
   - 查找 `features/*.rc-{id}.md` 和 `*.tech.rc-{id}.md`
   - 用 RC 内容替换正式文档（或重命名为正式文档）
   - 移除 `rc_for` 字段
   - 更新页脚为 `_更新: {date} | CR-{id}_`
3. **删除 RC 文件**
4. **更新 `_deps.yaml`**: 从合并后的文档重建依赖图
5. **更新 CR 状态**: status → `done`, 添加 `completed: {date}`
6. **归档**: 移动 CR 到 `changes/archive/`
7. **归档 spec**: 移动 `specs/CR-{id}.*.md` 到 `specs/archive/`（如存在）
8. **更新索引**: `changes/_index.yaml`, `specs/_index.yaml`

---

## _deps.yaml 维护

从所有正式文档的 frontmatter 重建 `features/_deps.yaml`:

```yaml
updated: {date}

graph:
  {feature}:
    deps: [从 feature.md 的 deps 字段]
    affects: [从 feature.md 的 affects 字段]
```

确保双向一致性（A.deps 包含 B → B.affects 应包含 A）。
