# /dd-drop - Abandon Change

> Common definitions in `_base.md`

## Usage

```
/dd-drop <CR-id>
/dd-drop <CR-id> "<reason>"
```

## Applicable Status

| Status | Handling |
|--------|----------|
| draft | Drop directly |
| confirmed | Delete RC and spec, then drop |
| done | Cannot drop (already archived, use `git revert`) |

## Execution Steps

1. **Load CR**: Validate status
2. **Confirm drop**: Display operations to be performed, wait for confirmation
3. **Delete RC files**: `features/*.rc-{id}.md` (if confirmed)
4. **Delete spec files**: `specs/CR-{id}.*.md` (if exist)
5. **Update CR status**: status → `dropped`, add `dropped: {date}`, `drop_reason`
6. **Move CR**: to `changes/dropped/`
7. **Update index**: `_index.yaml`

## Recover Dropped Change

Manually move file from `changes/dropped/` back to `changes/`, change status to `draft`, remove dropped-related fields.
