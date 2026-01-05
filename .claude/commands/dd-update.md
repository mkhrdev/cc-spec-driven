# /dd-update - 更新文档

接收人类的自然语言输入，创建或修改变更记录。

## 用法

```
/dd-update "<描述>"                    # 创建新变更
/dd-update "<描述>" --implemented      # 标记为已实现功能
/dd-update <CR-id>                     # 修改已有变更
/dd-update <CR-id> "<补充描述>"        # 在已有变更上追加内容
```

## 参数

- `描述`: 任意格式的需求或功能描述
- `--implemented`: 标记为已实现的功能（只补文档，不生成开发规格）
- `<CR-id>`: 指定修改已有变更记录

## 执行步骤

### 创建新变更

1. **接收输入**
   - 人类可以用任意格式描述
   - 可以是一句话、列表、甚至截图描述

2. **生成CR-ID 和 UUID**
   - 读取 `project.yaml` 的 `next_cr_id`
   - 使用后自增保存
   - 生成 UUID 用于跨分支标识

3. **分层加载上下文**

   ```
   Level 0: 文件列表
   └── glob features/*.md → 获取文件名列表（排除 .rc-*.md）

   Level 1: Meta 信息
   └── 读取每个文件的 YAML frontmatter (前 20 行)
       提取: id, title, deps, affects, updated

   Level 2: 结构信息 (按需)
   └── 如果 meta.deps 包含目标功能，读取该文件的所有 ## 标题

   Level 3: 详细内容 (按需)
   └── 分析时需要具体内容，则读取完整章节
   ```

   **始终加载**:
   - `products/{product}/overview.md`
   - `products/{product}/glossary.yaml`

4. **AI分析**
   - 识别涉及哪些功能
   - 判断是否需要新建功能文档
   - 判断是否需要调整全景

5. **术语检查** (console 提醒)
   - 发现文档中使用了未在 glossary 定义的术语时
   - **仅在 console 输出提醒**，不记录到 CR
   - 示例输出:
     ```
     ⚠️ 发现未定义术语: "会员等级", "积分"
     建议: 请考虑更新 glossary.yaml
     ```

6. **多轮澄清**（如需要）
   - 发现需求不明确时，询问人类确认
   - 澄清内容记录到 CR 的"澄清记录"章节

7. **生成变更记录**

   创建 `changes/CR-{id}.md`:
   ```markdown
   # CR-{id}: {标题}

   ---
   uuid: {uuid}
   status: draft
   type: change | implemented
   created: {date}
   updated: {date}
   ---

   ## 原始输入
   {人类的原始输入，原样保留}

   ## 澄清记录
   {多轮澄清的问答记录，如无则留空}

   ## 变更内容
   {AI整理后的格式化内容}

   ## 影响范围
   - 功能: [{feature}]
   - 新建: [{feature}]

   ---
   _创建: {date}_
   ```

8. **创建 feature.tech.md**（如涉及新功能）
   - 读取 `project.yaml` 中配置的 repo
   - 尝试获取各仓库的技术上下文 (README, 配置文件)
   - 创建 `features/{feature}.tech.md`

   ```markdown
   ---
   id: {uuid}
   feature: {feature-name}
   cr: CR-{id}
   updated: {date}
   ---

   # {Feature} - 技术共识

   ## 涉及仓库
   - {repo}: {影响描述}

   ## 技术决策
   {待讨论 - 列出可能的方案供选择}

   ## 接口约定
   {待定义}

   ## 注意事项
   {待补充}

   ---
   _创建: {date} | CR-{id}_
   ```

9. **输出确认**
   ```
   已创建变更: CR-{id}

   影响范围:
   - {feature1} (修改)
   - {feature2} (新建)

   下一步:
   - review 变更内容
   - 使用 /dd-confirm CR-{id} 确认
   ```

### 修改已有变更

1. **加载变更记录**
   - `changes/CR-{id}.md`
   - 读取 YAML frontmatter 中的 status

2. **状态处理**

   | 当前状态 | 处理方式 |
   |---------|---------|
   | draft | 直接追加修改 |
   | confirmed | 触发回退流程 (见下方) |
   | done | 拒绝修改，提示已归档 |
   | dropped | 拒绝修改，提示已放弃 |

3. **confirmed 状态回退流程**

   ```
   警告: CR-{id} 已确认，修改将触发以下操作:
   - 状态回退到 draft
   - 删除已生成的 RC 文件:
     - features/{feature}.rc-{id}.md
     - features/{feature}.tech.rc-{id}.md
   - 删除已生成的 spec 文件 (如有):
     - specs/CR-{id}.dev.md
     - specs/CR-{id}.test.md

   是否继续? (y/n)
   ```

   确认后:
   - 删除相关 RC 文件
   - 删除相关 spec 文件
   - 修改状态为 draft
   - 继续正常修改流程

4. **分层加载上下文**
   - 同创建新变更的步骤3

5. **合并修改**
   - 如有新描述，追加到"原始输入"章节
   - 重新分析影响范围
   - 更新"变更内容"章节

6. **更新 feature.tech.md**（如有）
   - 若涉及的功能有 .tech.md 文件
   - 根据需求变更更新技术共识

7. **更新变更记录**
   - 更新 frontmatter 的 `updated` 字段
   - 更新页脚 `_更新: {date}_`

8. **输出确认**
   ```
   已更新变更: CR-{id}

   变更内容:
   - {修改点1}
   - {修改点2}
   ```

## glossary 使用说明

- 加载 `glossary.yaml` 作为术语参考
- 发现需求中使用了未定义术语时，**仅 console 提醒**
- glossary.yaml 由人类自行维护

## Context Loading 详解

### Level 0: 文件列表
```bash
glob: features/*.md
排除: features/*.rc-*.md, features/*.tech.md
结果: [login.md, payment.md, order.md, ...]
```

### Level 1: Meta 信息
读取每个文件的前 20 行，解析 YAML frontmatter:
```yaml
---
id: uuid-xxx
title: 用户登录
deps: [auth, sms]
affects: [dashboard]
updated: 2025-01-04
---
```

### Level 2: 结构信息
如果需要了解某个 feature 的结构:
```
## 概述
## 相关
## 功能点
## 边界
```

### Level 3: 详细内容
按需读取具体章节的完整内容

## 示例

```
/dd-update "用户登录支持手机号和邮箱两种方式，需要验证码"
/dd-update "增加微信登录功能"
/dd-update "我们的系统有用户、订单、支付、库存四个模块"
/dd-update CR-002 "补充：验证码有效期5分钟"  # CR-002 是示例 ID
```
