# /dd-check - 检查文档一致性

综合检查工具，检查文档间的一致性和完整性。

**注意**: 所有检查结果**仅输出到 console**，不生成报告文件，不阻断任何流程。

## 用法

```
/dd-check [product]
/dd-check [product] --type=<check-type>
/dd-check [product] --type=all
```

## 检查类型

### 已实现检查

#### 1. glossary - 术语一致性检查
检查术语表与文档使用是否一致。

- glossary.yaml 中定义的术语是否在文档中正确使用
- 文档中使用的术语是否已定义
- 同义词（aliases）使用建议统一

#### 2. format - 文档格式规范检查
检查文档是否符合模板结构。

- feature.md 是否包含必要章节（概述、相关、功能点、边界）
- feature.md 是否包含 YAML frontmatter（id, title, deps, affects, updated）
- CR.md 必填字段是否完整（uuid, status, 原始输入、变更内容、影响范围）
- 日期格式是否正确
- 页脚格式是否规范

#### 3. deps - 依赖关系检查
检查功能间的依赖关系。

- 依赖的功能文档是否存在
- 循环依赖检测（A→B→C→A）
- 双向关系一致性（A依赖B，B应声明影响A）

#### 4. refs - 文档引用检查
检查文档间的引用关系。

- CR 引用的 feature 是否存在
- feature.md 页脚的 CR 引用是否有效
- 孤立文档警告（无任何引用的功能文档）
- RC 文件与 CR 关联检查

#### 5. status - 状态一致性检查
检查变更状态的一致性。

- 长时间(>7天)停留 draft 状态的 CR
- _index.yaml 与实际文件状态是否匹配
- archive/ 和 dropped/ 文件状态校验
- 孤立 RC 文件检测（对应 CR 不存在）

## 输出格式 (Console Only)

```
=== 文档检查: {product} ===

检查类型: all

[严重] 必须修复:
  ✗ refs: CR-003 引用的 feature "payment" 不存在
  ✗ deps: "cart" 依赖 "inventory"，但该文件不存在

[警告] 建议修复:
  ⚠ deps: "login" 依赖 "auth"，但 "auth" 未声明影响 "login"
  ⚠ status: CR-002 已在 draft 状态停留 12 天
  ⚠ glossary: 发现未定义术语 "会员"

[信息] 供参考:
  ℹ refs: feature "legacy" 无 CR 引用
  ℹ format: overview.md 的 "Scope" 章节为空

统计: 5 功能 | 2 技术共识 | 2 进行中 | 10 已完成 | 1 已放弃

检查完成: 2 严重 | 3 警告 | 2 信息
```

## 设计原则

- **仅提醒，不阻断**: 检查结果仅供参考，不会阻止任何操作
- **Console 输出**: 不生成报告文件，保持文档目录整洁
- **人类决策**: 是否修复由人类判断，AI 只负责发现问题

## 示例

```
/dd-check my-product                    # 执行所有检查
/dd-check my-product --type=glossary    # 只检查术语
/dd-check my-product --type=deps        # 只检查依赖
/dd-check --type=format                 # 当前产品，只检查格式
```
