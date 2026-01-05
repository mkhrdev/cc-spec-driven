# /dd-init - 初始化产品

> 公共定义见 `_base.md`

## 用法

```
/dd-init <product-name>
```

## 执行步骤

1. **验证**: 检查 `products/{product-name}/` 是否已存在
2. **创建目录结构**: 按 `_base.md` 的目录结构创建
3. **生成初始文件**:
   - `project.yaml`: name, description, created, language:"zh", repos:[], next_cr_id:1
   - `glossary.yaml`: version:"1.0", terms:[]
   - `overview.md`: 标题 + 待补充占位
   - `features/_deps.yaml`: 空依赖图
   - `changes/_index.yaml`: changes:[]
   - `specs/_index.yaml`: specs:[]

创建完成后提示编辑 overview.md 和使用 /dd-update。
