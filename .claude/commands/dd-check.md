# /dd-check - 检查文档一致性

> 公共定义见 `_base.md`

所有检查结果**仅输出到 console**，不生成文件，不阻断流程。

## 用法

```
/dd-check [product]
/dd-check [product] --type=<check-type>
/dd-check [product] --type=all
```

## 检查类型

| 类型 | 检查内容 |
|------|---------|
| glossary | 术语表与文档使用一致性 |
| format | 文档格式规范（必要章节、frontmatter） |
| deps | 依赖关系（存在性、循环、双向一致、_deps.yaml同步） |
| refs | 引用关系（CR↔feature、孤立文档、RC↔CR） |
| status | 状态一致性（长期 draft、索引同步） |

---

## _deps.yaml 验证

检查 `features/_deps.yaml` 与各 feature.md 的一致性：

1. **同步检查**: _deps.yaml 中的依赖关系是否与 feature.md 的 frontmatter 一致
2. **双向检查**: A.deps 包含 B → B.affects 应包含 A
3. **存在性检查**: 引用的 feature 是否存在

如发现不一致，输出具体差异。

---

## 输出格式

```
=== 文档检查: {product} ===

[严重] 必须修复:
  ✗ {类型}: {问题描述}

[警告] 建议修复:
  ⚠ {类型}: {问题描述}

[信息] 供参考:
  ℹ {类型}: {问题描述}

统计: {n} 功能 | {m} 技术共识 | {p} 进行中 | {q} 已完成

检查完成: {x} 严重 | {y} 警告 | {z} 信息
```

## 设计原则

- **仅提醒，不阻断**: 检查结果仅供参考
- **Console 输出**: 不生成报告文件
- **人类决策**: 是否修复由人类判断
