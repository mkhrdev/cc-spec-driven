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
| done | Cannot drop archived CR (use `git revert` if needed) |

## Execution Steps

1. **Load CR**: Validate status
2. **Confirm drop**: Display operations to be performed, wait for confirmation
3. **Delete RC files**: `features/*.rc-{id}.md` (if confirmed)
4. **Delete spec files**: `specs/CR-{id}.*.md` (if exist)
5. **Move test cases**: `cases/CR-{id}/` to `cases/dropped/CR-{id}/` (if exist)
6. **Clean up references** (if test cases exist):
   - Update referenced party's `referenced_by`, remove current CR
   - If other CRs reference this CR, warn and clean up referencing CRs' `refs`
7. **Update CR status**: status → `dropped`, add `dropped: {date}`, `drop_reason`
8. **Move CR**: to `changes/dropped/`
9. **Update indices**: `changes/_index.yaml`, `specs/_index.yaml`, `cases/_index.yaml`

## Recover Dropped Change

Manually move file from `changes/dropped/` back to `changes/`, change status to `draft`, remove dropped-related fields.
