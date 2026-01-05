# /dd-update - 更新文档

> 公共定义见 `_base.md`

## 用法

```
/dd-update "<描述>"                    # 创建新变更
/dd-update "<描述>" --implemented      # 已实现功能
/dd-update "<描述>" --bootstrap        # 冷启动模式
/dd-update <CR-id>                     # 修改已有变更
/dd-update <CR-id> "<补充描述>"        # 追加内容
```

---

## 创建新变更

1. **生成ID**: 读取 `project.yaml` 的 `next_cr_id`，自增保存，生成 UUID
2. **加载上下文**: 按 `_base.md` 的 Context Loading（含 `_deps.yaml`）
3. **分析影响范围**: 识别涉及的 feature、判断是否新建
4. **分析依赖变更**: 读取 `_deps.yaml`，推导双向关系，输出影响分析
5. **术语检查**: 仅 console 提醒
6. **多轮澄清**: 需求不明确时询问，记录到 CR
7. **创建 CR**: 格式见 `_base.md`
8. **创建 tech.md**: 如涉及新功能

---

## 修改已有变更

| 状态 | 处理 |
|------|------|
| draft | 直接追加 |
| confirmed | 警告后回退（删除RC/spec，状态→draft）|
| done/dropped | 拒绝 |

**confirmed 回退**:
```
警告: CR-{id} 已确认，修改将触发:
- 状态回退到 draft
- 删除 RC 文件和 spec 文件
是否继续? (y/n)
```

---

## --bootstrap 模式

直接创建/更新 feature.md，不生成 CR/spec。

1. 加载上下文（含 `_deps.yaml`）
2. 分析影响范围
3. 直接创建/更新 `features/{feature}.md`
4. 更新依赖关系到文档
5. 更新 `_deps.yaml`
6. 术语检查（仅 console 提醒）

冷启动后建议执行 `/dd-check`。

---

## --implemented

走 CR 流程，但 type 标记为 `implemented`，不生成 spec-dev。
