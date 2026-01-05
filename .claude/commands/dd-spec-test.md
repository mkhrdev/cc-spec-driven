# /dd-spec-test - 生成测试规格

> 公共定义见 `_base.md`

## 用法

```
/dd-spec-test <CR-id>
/dd-spec-test --init
```

## 前置条件

- 状态必须是 `confirmed`
- `--init` 例外，用于项目初始化

## 执行步骤

1. **加载 CR**: 验证状态
2. **加载上下文**: 涉及的 feature.md（或 RC），重点读取"边界"章节
3. **生成测试用例**: Given-When-Then 格式
4. **生成 spec**: `specs/CR-{id}.test.md`
5. **更新索引**: `specs/_index.yaml`

## Spec 结构

```markdown
# 测试规格: CR-{id}

## 测试范围
## 测试用例
### TC-NNN: {用例名}
**优先级**: P0|P1|P2
**类型**: E2E|API|Unit
**Given**: ...
**When**: ...
**Then**: ...

## 边界测试
## 回归检查点
```

## 优先级与类型

| 优先级 | 含义 | | 类型 | 含义 |
|--------|------|-|------|------|
| P0 | 核心流程，阻断发布 | | E2E | 端到端 |
| P1 | 重要功能，建议修复 | | API | 接口测试 |
| P2 | 边缘场景，低优先级 | | Unit | 单元测试 |

## --init 模式

基于所有现有 feature.md 生成完整测试规格 `specs/INIT.test.md`。
