# Onboarding Guide

> Setup guide for new products and existing projects.

## Architecture

```
spec-repo/ (this repo)
├── AGENT.md
├── .claude/commands/    # Skills
├── template/            # Copy for new products
└── products/
    └── {product}/
        ├── project.yaml
        ├── glossary.yaml
        ├── snapshot/
        └── spec/
            ├── product/
            └── test/

implementation-repo/     # Your code
├── AGENT.md            # Points to spec repo
└── src/...
```

## New Products

### 1. Copy Template

```bash
cp -r template/ products/my-product/
```

### 2. Configure project.yaml

```yaml
product:
  name: "my-product"
  description: "Description"

repos:
  frontend:
    path: "../frontend-repo"
    tech_stack:
      language: typescript
      framework: react

  backend:
    path: "../backend-repo"
    tech_stack:
      language: go
      framework: gin
```

### 3. Start Building

```bash
/product-build --new auth
```

## Existing Projects (Cold Start)

### 1. Copy Template

```bash
cp -r template/ products/my-product/
```

### 2. Create cold_start_context.yaml

```yaml
project_overview: |
  What does this project do?

core_flows: |
  Main user journeys

glossary_seeds: |
  Domain terms
```

### 3. Run Cold Start

```bash
/cold-start --analyze
```

Creates Snapshot v1 from existing code.

### 4. New Features

For new features, follow normal workflow:

```bash
/product-build --new new-feature
```

## Implementation Repos

Each implementation repo needs minimal AGENT.md:

```markdown
# AGENT.md

**Spec Repo**: `../spec`
**Product**: `my-product`

When implementing:
1. Check spec repo for ProductSpec
2. Reference TestSpec for test cases
3. Technical design in ProductSpec for APIs
```

## Checklist

### New Product

- [ ] Copy template to `products/{name}/`
- [ ] Configure `project.yaml`
- [ ] Run `/product-build --new`

### Existing Project

- [ ] Copy template to `products/{name}/`
- [ ] Configure `project.yaml`
- [ ] Create `cold_start_context.yaml`
- [ ] Run `/cold-start --analyze`
- [ ] Review Snapshot v1

### Per Implementation Repo

- [ ] Add `AGENT.md` with spec repo reference
