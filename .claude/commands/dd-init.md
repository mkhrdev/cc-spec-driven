# /dd-init - 初始化产品

初始化一个新产品的目录结构。

## 用法

```
/dd-init <product-name>
```

## 执行步骤

1. **验证产品名**
   - 检查 `products/{product-name}/` 是否已存在
   - 如存在则报错退出

2. **创建目录结构**
   ```
   products/{product-name}/
   ├── project.yaml
   ├── glossary.yaml
   ├── overview.md
   ├── features/
   ├── changes/
   │   ├── _index.yaml
   │   ├── archive/
   │   └── dropped/
   └── specs/
       ├── _index.yaml
       ├── archive/
       └── dropped/
   ```

3. **生成初始文件**

   **project.yaml**:
   ```yaml
   name: "{product-name}"
   description: ""
   created: "{date}"
   language: "zh"
   repos: []
   next_cr_id: 1
   ```

   **glossary.yaml**:
   ```yaml
   version: "1.0"
   terms:
     # - term: 用户
     #   definition: 系统的注册使用者
     #   aliases: []
     #   related: []
   ```

   **overview.md**:
   ```markdown
   # {Product Name}

   {待补充}

   ## 模块
   - {待补充}

   ## Scope
   - In: {待补充}
   - Out: {待补充}
   ```

   **changes/_index.yaml**:
   ```yaml
   # 变更索引
   changes: []
   ```

   **specs/_index.yaml**:
   ```yaml
   # 规格索引
   specs: []
   ```

4. **输出确认**
   ```
   已创建产品: {product-name}

   目录结构:
   - features/      # 功能文档
   - changes/       # 变更记录
   - specs/         # 规格文件

   下一步:
   - 编辑 overview.md 描述产品全景
   - 使用 /dd-update 描述产品功能
   ```
