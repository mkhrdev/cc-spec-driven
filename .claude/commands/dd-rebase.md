# /dd-rebase - Handle Branch Merge Conflicts

> Common definitions in `_base.md`

When branch merge encounters document conflicts, help reapply change intent.

**Core concept**: Reapply change intent, not text merge.

## Usage

```
/dd-rebase <CR-id>
```

## Execution Steps

1. **Load CR**: Extract original requirements and change intent
2. **Load current main branch documents**: Already contains other branch changes
3. **Intent extraction** (priority): Original Input > Change Content > Clarification Records
4. **Conflict analysis**: Identify conflict type
   - Concept conflict: Different definitions of same concept
   - Metric conflict: Different values/limits
   - Order conflict: Different process/step ordering
   - Scope conflict: Different feature boundary definitions
5. **Output analysis report**: Show change intent, current state, conflict points and suggestions
6. **Human confirmation**: Decide how to handle each conflict
7. **Regenerate**: Apply adjusted changes based on current main branch, update CR

## Notes

- After rebase, CR maintains original status (draft/confirmed)
- Will add "Rebase Records" section to CR
