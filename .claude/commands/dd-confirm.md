# /dd-confirm - 确认变更

人类确认变更后，生成 RC (Release Candidate) 预览版文档。

## 用法

```
/dd-confirm <CR-id>
```

## 前置条件

- 变更状态必须是 `draft`
- 人类已review并认可变更内容

## 执行步骤

1. **加载变更记录**
   - `changes/{CR-id}.md`
   - 解析 YAML frontmatter

2. **验证状态**
   - 确认当前状态是 `draft`
   - 如不是，提示当前状态并退出:
     ```
     错误: CR-{id} 状态为 {status}，无法确认。
     - confirmed: 已确认，可直接生成 spec 或 /dd-done
     - done: 已完成归档
     - dropped: 已放弃
     ```

3. **分层加载上下文**
   - 同 dd-update 的 Context Loading 策略
   - 重点加载影响范围内的 feature 文件

4. **生成 RC 预览文档**

   对于每个涉及的功能：

   **如果功能文档已存在**:
   - 加载 `features/{feature}.md`
   - 根据变更内容生成更新后的版本
   - 保存为 `features/{feature}.rc-{id}.md`

   **如果功能文档不存在**（从 CR 的"影响范围.新建"字段推导）:
   - 从 CR 的"变更内容"章节提取：概述、功能点、边界
   - 从 CR 的"影响范围"提取：依赖、影响关系
   - 创建 `features/{feature}.rc-{id}.md`

   RC 文件格式:
   ```markdown
   ---
   id: {uuid}
   title: {Feature Name}
   deps: [{依赖的feature}]
   affects: [{影响的feature}]
   updated: {date}
   rc_for: CR-{id}
   ---

   # {Feature Name}

   ## 概述
   {从变更内容提取的2-3句话描述}

   ## 相关
   - 依赖: [{从影响范围提取}]
   - 影响: [{从影响范围提取}]

   ## 功能点
   {从变更内容提取的具体描述}

   ## 边界
   - {从变更内容提取的边界条件}

   ---
   _预览版: CR-{id} | 生成: {date}_
   ```

5. **生成 tech.md RC**（如有 tech.md 更新）
   - 如果 CR 涉及技术共识变更
   - 创建 `features/{feature}.tech.rc-{id}.md`

6. **更新变更状态**
   - 修改 `changes/{CR-id}.md` 中 frontmatter 的 status 为 `confirmed`
   - 更新 `updated` 字段
   - 添加页脚 `_更新: {date}_`

7. **输出确认**
   ```
   已确认变更: CR-{id}

   生成的 RC 文档:
   - features/login.rc-{id}.md (修改预览)
   - features/payment.rc-{id}.md (新建预览) ← 请review
   - features/login.tech.rc-{id}.md (技术共识预览)

   下一步:
   - review RC 文档内容
   - 使用 /dd-spec-dev CR-{id} 生成开发规格
   - 使用 /dd-spec-test CR-{id} 生成测试规格
   - 使用 /dd-done CR-{id} 合并到正式文档
   ```

## RC 机制说明

### 为什么使用 RC？

- **预览而非提交**: confirm 时生成预览版，done 时才正式合并
- **安全回退**: drop 时只需删除 RC 文件，无需 git revert
- **并行支持**: 多个 CR 可以同时存在，各自有独立的 RC 文件
- **清晰追踪**: RC 文件名包含 CR-ID，明确关联

### RC 文件命名

```
{feature}.rc-{id}.md       # 功能文档 RC
{feature}.tech.rc-{id}.md  # 技术共识 RC
```

示例:
```
features/
├── login.md               # 正式文档
├── login.rc-003.md        # CR-003 的预览版
├── login.tech.md          # 正式技术共识
└── login.tech.rc-003.md   # CR-003 的技术共识预览版
```

### RC 与正式文档的差异

RC 文档的 frontmatter 包含额外字段:
```yaml
rc_for: CR-{id}  # 标识这是哪个 CR 的预览版
```

页脚格式:
```
_预览版: CR-{id} | 生成: {date}_
```

## 注意事项

- 新建功能的 RC 文档，AI会尽力从 CR 提取结构化信息
- 如果信息不足，会使用 `{待补充}` 占位
- RC 文档标记需要人工review
- **不再自动更新 glossary.yaml**，术语由人类维护
