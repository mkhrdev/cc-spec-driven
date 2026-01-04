# /dd-spec-test - 生成测试规格

基于变更记录和功能文档，生成测试规格。

## 用法

```
/dd-spec-test <CR-id>
/dd-spec-test --init
```

## 前置条件

- **变更状态必须是 `confirmed`**（draft 状态无法生成）
- `--init` 模式例外，用于项目初始化时生成全功能测试规格

## 执行步骤

1. **加载变更记录**
   - `changes/{CR-id}.md`
   - 解析 YAML frontmatter

2. **验证状态**
   - 如果是 `draft`，**拒绝执行**并提示:
     ```
     错误: CR-{id} 状态为 draft，无法生成测试规格。

     请先执行:
     /dd-confirm CR-{id}
     ```
   - 如果是 `done`，提示该 CR 已完成
   - 如果是 `dropped`，提示该 CR 已放弃

3. **加载上下文**
   - 涉及的 `features/{feature}.md` 或 RC 版本
   - 重点读取 "边界" 章节
   - CR 的 "变更内容" 和 "影响范围"

4. **生成测试规格**

   创建 `specs/CR-{id}.test.md`:

   ```markdown
   # 测试规格: CR-{id}

   ---
   uuid: {uuid}
   cr: CR-{id}
   status: active
   generated: {timestamp}
   ---

   ## 测试范围
   - 新增功能: [login]
   - 影响功能: [dashboard, settings]

   ## 测试用例

   ### TC-001: 手机号登录成功
   **优先级**: P0
   **类型**: E2E

   **Given** (前置条件):
   - 用户已注册手机号 13800138000
   - 用户未登录

   **When** (操作):
   - 访问登录页面
   - 输入手机号 13800138000
   - 点击获取验证码
   - 输入正确验证码
   - 点击登录

   **Then** (预期结果):
   - 跳转到首页
   - 显示用户昵称
   - 本地存储包含 token

   ### TC-002: 邮箱登录成功
   **优先级**: P0
   **类型**: E2E

   **Given** (前置条件):
   - 用户已注册邮箱 test@example.com
   - 用户未登录

   **When** (操作):
   - 访问登录页面
   - 切换到邮箱登录
   - 输入邮箱 test@example.com
   - 点击获取验证码
   - 输入正确验证码
   - 点击登录

   **Then** (预期结果):
   - 跳转到首页
   - 显示用户昵称

   ### TC-003: 验证码过期
   **优先级**: P1
   **类型**: API

   **Given** (前置条件):
   - 用户请求了验证码
   - 等待超过 5 分钟

   **When** (操作):
   - 使用过期验证码尝试登录

   **Then** (预期结果):
   - 返回错误码 AUTH_CODE_EXPIRED
   - 提示 "验证码已过期"

   ## 边界测试

   ### TC-EDGE-001: 验证码格式错误
   **Given**: 用户在登录页面
   **When**: 输入非数字验证码 "abcd"
   **Then**: 显示 "验证码格式错误"

   ### TC-EDGE-002: 手机号格式校验
   **Given**: 用户在登录页面
   **When**: 输入非法手机号 "123"
   **Then**: 显示 "请输入正确的手机号"

   ### TC-EDGE-003: 验证码发送频率限制
   **Given**: 用户刚发送过验证码
   **When**: 立即再次点击发送
   **Then**: 按钮禁用，显示倒计时 60 秒

   ## 回归检查点
   - [ ] dashboard: 登录后数据正常加载
   - [ ] settings: 已登录状态下设置页面正常

   ## 参考资料
   - 功能边界: `features/login.md` → 边界章节
   - CR 详情: `changes/CR-{id}.md`
   ```

5. **更新规格索引**

   更新 `specs/_index.yaml`:
   ```yaml
   specs:
     # ... 现有记录
     - cr: CR-{id}
       dev: true
       test: true
       status: active
   ```

6. **输出确认**
   ```
   已生成测试规格: specs/CR-{id}.test.md

   包含:
   - {n} 个功能测试用例
   - {m} 个边界测试用例
   - {k} 个回归检查点

   优先级分布:
   - P0 (必须通过): {x} 个
   - P1 (应该通过): {y} 个
   - P2 (可以通过): {z} 个
   ```

## 项目初始化模式

```
/dd-spec-test --init
```

用于项目初始化时，基于所有现有功能文档生成完整测试规格。

**执行步骤**:
1. 读取所有 `features/*.md` 文件（排除 RC 和 tech.md）
2. 为每个功能生成测试用例
3. 创建 `specs/INIT.test.md`

**输出格式**:
```markdown
# 初始化测试规格

---
uuid: {uuid}
type: init
generated: {timestamp}
---

## 功能: login

### TC-LOGIN-001: ...

## 功能: payment

### TC-PAYMENT-001: ...
```

## Given-When-Then 格式说明

采用 BDD (Behavior-Driven Development) 风格:

| 部分 | 含义 | 示例 |
|------|------|------|
| Given | 前置条件/初始状态 | 用户已登录 |
| When | 触发操作 | 点击退出按钮 |
| Then | 预期结果 | 跳转到登录页 |

**优势**:
- 可读性强，非技术人员也能理解
- 可直接转化为自动化测试代码
- 与 Cucumber/Gherkin 格式兼容

## 测试类型说明

| 类型 | 含义 | 自动化工具 |
|------|------|-----------|
| E2E | 端到端测试 | Playwright, Cypress |
| API | 接口测试 | pytest, Jest |
| Unit | 单元测试 | pytest, Jest, Vitest |

## 优先级说明

| 级别 | 含义 | 阻断性 |
|------|------|--------|
| P0 | 核心流程，必须通过 | 阻断发布 |
| P1 | 重要功能，应该通过 | 建议修复 |
| P2 | 边缘场景，可以通过 | 低优先级 |

## 注意事项

- Test Spec **不涉及实现方案的不确定性**，只关注功能行为验证
- 边界测试用例从 feature.md 的 "边界" 章节提取
- 回归检查点关注变更可能影响的现有功能
- 测试用例 ID 格式: `TC-{序号}` 或 `TC-EDGE-{序号}`
