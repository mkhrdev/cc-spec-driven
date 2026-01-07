# /dd-init - Initialize Product

> Common definitions in `_base.md`

## Usage

```
/dd-init <product-name>
```

## Execution Steps

1. **Validate**: Check if `products/{product-name}/` already exists
2. **Create directory structure**: Follow directory structure in `_base.md`
3. **Generate initial files**:
   - `project.yaml`: name, description, created, language:"en", repos:[], next_cr_id:1
   - `glossary.yaml`: version:"1.0", terms:[]
   - `overview.md`: Title + placeholder content
   - `features/_deps.yaml`: Empty dependency graph
   - `changes/_index.yaml`: changes:[]
   - `specs/_index.yaml`: specs:[]

After creation, prompt to edit overview.md and use /dd-update.
