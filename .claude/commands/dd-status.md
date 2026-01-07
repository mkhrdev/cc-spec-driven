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

If inconsistencies found (e.g., missing index, orphan RC), show notice at the end and suggest running `/dd-check`.
