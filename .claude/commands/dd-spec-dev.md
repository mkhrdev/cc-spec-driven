# /dd-spec-dev - 生成开发规格

基于变更记录和功能文档，生成面向开发的技术规格。

## 用法

```
/dd-spec-dev <CR-id>
```

## 前置条件

- **变更状态必须是 `confirmed`**（draft 状态无法生成）
- 如果 CR 类型为 `implemented`，会提示跳过

## 执行步骤

1. **加载变更记录**
   - `changes/{CR-id}.md`
   - 解析 YAML frontmatter

2. **验证状态**
   - 如果是 `draft`，**拒绝执行**并提示:
     ```
     错误: CR-{id} 状态为 draft，无法生成开发规格。

     请先执行:
     /dd-confirm CR-{id}
     ```
   - 如果是 `done`，提示该 CR 已完成
   - 如果是 `dropped`，提示该 CR 已放弃

3. **检查类型**
   - 如果类型是 `implemented`，显示提示并退出:
     ```
     提示: CR-{id} 标记为已实现功能，无需生成开发规格。
     ```

4. **加载上下文**
   - 涉及的 `features/{feature}.md` 或 RC 版本
   - 涉及的 `features/{feature}.tech.md` 或 RC 版本
   - `project.yaml` 获取仓库配置

5. **读取外部仓库信息**

   对于 `project.yaml` 中配置的每个仓库:
   ```yaml
   repos:
     - name: frontend
       path: "../frontend"
     - name: backend
       path: "../backend"
   ```

   **读取顺序**:
   1. `README.md` - 项目概述和技术栈
   2. 配置文件 - 技术版本信息:
      - `package.json` (Node.js)
      - `pyproject.toml` / `requirements.txt` (Python)
      - `go.mod` (Go)
      - `Cargo.toml` (Rust)
   3. 按需读取具体代码文件

   **仓库不可达时**:
   - 标注 "仓库信息待确认"
   - 继续生成 spec，在技术约束中标注不确定项

6. **处理 tech.md 待决策项**

   读取 tech.md 中的 `**待人类确认**` 标记:
   ```markdown
   ### 决策1: 验证码存储
   **方案A**: Redis (推荐)
   **方案B**: 数据库
   **待人类确认**: 生成 spec 时选择
   ```

   **交互确认**:
   ```
   技术决策确认:

   决策1: 验证码存储
   - A: Redis (推荐)
   - B: 数据库

   请选择 (A/B): _
   ```

   **对比 repo 实际情况**:
   - 如果 repo 中已有 Redis 依赖 → 提示 "推荐方案A (与现有技术栈一致)"
   - 如果 repo 中没有 Redis → 提示 "方案A需要新增依赖"

7. **生成开发规格**

   创建 `specs/CR-{id}.dev.md`:

   ```markdown
   # 开发规格: CR-{id}

   ---
   uuid: {uuid}
   cr: CR-{id}
   status: active
   generated: {timestamp}
   ---

   ## 概述
   {一句话描述变更目标}

   ## 前置条件
   - 依赖的 CR: [{如有}]
   - 依赖的外部服务: [{如有}]

   ## 技术约束
   - 语言/框架版本:
     - frontend: React 18.x, TypeScript 5.x
     - backend: Python 3.11, FastAPI 0.100+
   - 架构风格: {从 tech.md 或 repo 推断}
   - 编码规范: {从 repo 读取}

   ## 技术决策
   以下决策已在生成时确认:
   - 决策1: 验证码存储 → **Redis** (与现有技术栈一致)
   - 决策2: 前端状态管理 → **Zustand**

   ## 实现任务

   ### Task 1: {任务名}
   **目标**: {用户故事或功能点}

   **上下文**:
   - 相关文件: `src/components/Login.tsx`
   - 相关接口: `POST /api/auth/login`
   - 参考实现: `src/components/Register.tsx` (类似模式)

   **验收条件**:
   - [ ] 支持手机号登录
   - [ ] 支持邮箱登录
   - [ ] 验证码 5 分钟过期

   **实现提示**:
   - 复用现有的 `useAuth` hook
   - 验证码服务调用 `SmsService.sendCode()`

   ### Task 2: {任务名}
   **目标**: ...

   **上下文**:
   - 相关文件: ...
   - 相关接口: ...

   **验收条件**:
   - [ ] ...

   **实现提示**:
   - ...

   ## 实现顺序
   1. Task 2: 后端 API (无依赖)
   2. Task 1: 前端组件 (依赖 Task 2)

   ## 参考资料
   - CR 详情: `changes/CR-{id}.md`
   - 功能文档: `features/{feature}.md` (或 RC 版本)
   - 技术共识: `features/{feature}.tech.md`
   ```

8. **更新规格索引**

   更新 `specs/_index.yaml`:
   ```yaml
   specs:
     # ... 现有记录
     - cr: CR-{id}
       dev: true
       test: false
       status: active
   ```

9. **输出确认**
   ```
   已生成开发规格: specs/CR-{id}.dev.md

   包含:
   - {n} 个开发任务
   - 涉及 {m} 个仓库
   - {k} 个技术决策已确认

   下一步:
   - review 开发规格
   - 使用 /dd-spec-test CR-{id} 生成测试规格
   ```

## 技术约束来源优先级

1. **tech.md** (首选) - 团队已达成的技术共识
2. **repo 实际情况** (校验) - 确保与现有代码一致
3. **推断** (兜底) - 基于 repo 配置文件推断

**冲突处理**:
- 如果 tech.md 与 repo 不一致，提示人类确认
- 如果无法确定，标注 `{待确认}` 占位

## Spec 设计原则

基于行业最佳实践，开发规格应满足:

1. **自包含**: 其他模型看 spec 就能开发，无需额外上下文
2. **可执行**: 验收条件明确，可直接转化为测试用例
3. **有序性**: 任务之间的依赖关系清晰
4. **可追溯**: 提供参考资料链接，需要更多上下文时可查阅

## 外部仓库读取示例

```
读取 frontend 仓库信息...
- README.md: React + TypeScript 项目
- package.json:
  - react: 18.2.0
  - typescript: 5.3.0
  - zustand: 4.4.0
  - tailwindcss: 3.4.0

读取 backend 仓库信息...
- README.md: FastAPI 项目
- pyproject.toml:
  - python: 3.11
  - fastapi: 0.109.0
  - redis: 5.0.0 ← 已有 Redis 依赖
```

## 注意事项

- spec 与 tech.md 冲突时，优先以 repo 实际情况为准
- 技术决策选择会固化到 spec 中，后续不再变动
- 如果需要修改技术决策，需要重新生成 spec
