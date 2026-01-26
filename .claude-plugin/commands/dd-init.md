---
description: Initialize product documentation structure in current directory
allowed-tools: [Read, Write, Glob]
arguments:
  - name: product-name
    description: Product name (used for the name field in project.yaml)
    required: true
---

# /dd-init - Initialize Product

> Common definitions in `_base.md`

## Usage

```
/dd-init <product-name>
```

Initialize product documentation structure in current directory.

## Execution Steps

1. **Validate**: Check if `project.yaml` already exists in current directory
   - If exists, prompt "Current directory already initialized" and exit
2. **Create directory structure**: Create structure in current directory per `_base.md`
3. **Generate initial files**:
   - `project.yaml`: name:{product-name}, description, created, language:"en", repos:[], next_cr_id:1
   - `glossary.yaml`: version:"1.0", terms:[]
   - `overview.md`: Title + placeholder content
   - `features/_deps.yaml`: Empty dependency graph
   - `changes/_index.yaml`: changes:[]
   - `specs/_index.yaml`: specs:[]
   - `cases/_index.yaml`: cases:[]

After creation, prompt to edit overview.md and use /dd-update.
