# /dd-status - 查看状态

显示当前产品的状态概览。

## 用法

```
/dd-status [product-name]
```

## 执行步骤

### 无参数时：列出所有产品

1. 扫描 `products/` 目录
2. 统计每个产品的变更和规格
3. 输出:
   ```
   产品列表:

   | 产品 | 进行中 | 已完成 | 已放弃 |
   |------|--------|--------|--------|
   | my-product | 2 | 10 | 1 |
   | another | 0 | 5 | 0 |
   ```

### 指定产品时：显示详细状态

1. **扫描文件**
   - `products/{product}/features/*.md` - 功能文档（排除 .rc-*.md）
   - `products/{product}/features/*.rc-*.md` - RC 预览文档
   - `products/{product}/changes/CR-*.md` - 进行中的变更
   - `products/{product}/changes/archive/` - 已完成的变更
   - `products/{product}/changes/dropped/` - 已放弃的变更
   - `products/{product}/specs/CR-*.md` - 进行中的规格
   - `products/{product}/specs/archive/` - 已完成的规格

2. **读取索引** (YAML 格式)
   - `products/{product}/changes/_index.yaml`
   - `products/{product}/specs/_index.yaml`

3. **输出状态报告**
   ```
   === {Product Name} ===

   ## 功能文档
   - login (login.md, login.tech.md)
   - payment (payment.md)
   - cart (cart.md, cart.tech.md)

   ## RC 预览文档
   - login.rc-003.md ← CR-003
   - payment.rc-003.md ← CR-003 (新建)

   ## 进行中的变更
   | ID | 标题 | 状态 | RC | 规格 |
   |----|------|------|-----|------|
   | CR-003 | 添加微信支付 | confirmed | 2 | dev |
   | CR-002 | 修复登录bug | draft | - | - |

   ## 最近完成 (最新5个)
   | ID | 标题 | 完成日期 |
   |----|------|----------|
   | CR-001 | 用户登录 | 2025-01-01 |

   ## 已放弃
   | ID | 标题 | 放弃日期 | 原因 |
   |----|------|----------|------|
   | CR-004 | 取消的功能 | 2025-01-02 | 需求变更 |

   ## 统计
   - 功能文档: 3 个 (2 个有技术共识)
   - RC 预览: 2 个
   - 变更: 2 进行中 / 1 已完成 / 1 已放弃
   - 规格: 2 开发规格 / 1 测试规格
   ```

## 一致性检查

状态报告同时检查：
- `_index.yaml` 与实际文件是否一致
- RC 文件是否有对应的 CR
- 如发现不一致，在报告末尾提示:
  ```
  ⚠️ 发现不一致:
  - changes/_index.yaml 缺少 CR-003 记录
  - login.rc-005.md 的对应 CR 不存在

  建议执行 /dd-check 进行详细检查
  ```

## 索引文件格式

### changes/_index.yaml

```yaml
changes:
  - id: CR-001
    uuid: a1b2c3d4-...
    title: 用户登录功能
    status: done
    created: 2025-01-01
    completed: 2025-01-04
    features: [login, auth]
  - id: CR-002
    uuid: e5f6g7h8-...
    title: 支付集成
    status: confirmed
    created: 2025-01-03
    features: [payment]
```

### specs/_index.yaml

```yaml
specs:
  - cr: CR-001
    dev: true
    test: true
    status: archived
  - cr: CR-002
    dev: true
    test: false
    status: active
```

## 注意

- 此命令通过扫描文件获取状态，不完全依赖 _index.yaml
- 如发现索引与实际文件不一致，会提示
- RC 列显示该 CR 关联的 RC 文件数量
- 规格列显示该 CR 已生成的规格类型 (dev/test)
