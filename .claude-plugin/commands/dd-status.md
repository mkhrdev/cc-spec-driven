---
description: View product documentation status and recommended actions
allowed-tools: [Read, Glob, Grep]
arguments:
  - name: product-name
    description: Product name (optional, lists all products if not specified)
    required: false
---

# /dd-status - View Status

> Common definitions in `_base.md`

## Usage

```
/dd-status [product-name]
```

## No arguments: List all products

Scan `products/` directory, output product list with change statistics.

## With product: Show detailed status

Scan and output:
- Feature documents list (features/*.md, exclude .rc-*.md)
- RC preview documents list
- In-progress changes (with status, RC count, spec status)
- Recent completed changes (latest 5)
- Statistics
- **Recommended next steps** (see below)

If inconsistencies found (e.g., missing index, orphan RC), show notice at the end and suggest running `/dd-check`.

---

## RC Staleness Detection

For CRs in `confirmed` status, check if their RC files are stale:

**Detection rule**:
- Compare RC file's `updated` time with corresponding formal document's `updated` time
- If formal document update time > RC file generation time → RC marked as `stale`

**Output format**:
```
In-progress changes:
- CR-001 [confirmed] ⚠️ stale - formal doc updated, suggest /dd-rebase
- CR-002 [confirmed] ✓ synced
- CR-003 [draft]
```

---

## Recommended Next Steps

Based on current status, provide operation suggestions:

| Status | Recommended Action |
|--------|-------------------|
| No draft CR | `/dd-update "description"` create new change |
| Has draft CR | `/dd-confirm CR-{id}` confirm change |
| Has confirmed CR (no spec) | `/dd-spec-dev CR-{id}` or `/dd-spec-test CR-{id}` |
| Has confirmed CR (has spec) | `/dd-done CR-{id}` or continue generating test cases |
| Has stale CR | `/dd-rebase CR-{id}` resolve conflicts |

**Output format**:
```
Recommended next steps:
→ /dd-confirm CR-003 (draft pending confirmation)
→ /dd-done CR-001 (confirmed, spec generated)
```
