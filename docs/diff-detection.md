# Diff Detection & Checksums

## Checksum System

Each spec tracks its checksum for integrity verification.

### Checksum Generation

```python
def generate_checksum(spec):
    content = spec.copy()
    # Exclude mutable fields
    del content['checksum']
    del content['metadata']['updated_at']
    del content['review_notes']

    yaml_str = yaml.dump(content, sort_keys=True)
    return hashlib.sha256(yaml_str.encode()).hexdigest()[:12]
```

### Stored Format

```yaml
_meta:
  id: PRD-1.0.0-auth
  checksum: sha256:a1b2c3d4e5f6
  generated_from:
    product:
      id: PRD-1.0.0-auth
      checksum: sha256:a1b2c3d4e5f6
```

## Spec References

### TestSpec → ProductSpec

```yaml
# TEST-1.0.0-auth.yaml
generated_from:
  product:
    id: PRD-1.0.0-auth
    checksum: sha256:a1b2c3d4e5f6
```

### Snapshot → Specs

```yaml
# snapshot_diff.yaml
source:
  product:
    id: PRD-1.0.0-auth
    checksum: sha256:a1b2c3d4e5f6
  test:
    id: TEST-1.0.0-auth
    checksum: sha256:f6e5d4c3b2a1
```

## Checksum Verification

### On Build

```
/test-build PRD-1.0.0-auth

1. Load ProductSpec
2. Calculate current checksum
3. Store in TestSpec.generated_from.product.checksum
```

### On Review

```
/test-review TEST-1.0.0-auth

1. Load TestSpec
2. Load ProductSpec
3. Calculate ProductSpec checksum
4. Compare with recorded checksum
5. If mismatch: WARNING - ProductSpec changed
```

### On Snapshot Build

```
/snapshot-build PRD-1.0.0-auth

1. Verify ProductSpec checksum
2. Verify TestSpec checksum
3. If mismatch: ERROR - specs changed since test execution
```

## Checksum Mismatch Handling

```
Checksum Verification Failed
════════════════════════════

ProductSpec: PRD-1.0.0-auth
  Recorded: sha256:a1b2c3d4e5f6
  Current:  sha256:x9y8z7w6v5u4

ProductSpec has changed since TestSpec was generated.

Options:
  1. Regenerate TestSpec
  2. Review changes and decide
  3. Abort operation
```

## Quick Reference

| Operation | Verifies |
|-----------|----------|
| `/test-build` | ProductSpec exists, is approved |
| `/test-review` | ProductSpec unchanged |
| `/snapshot-build` | ProductSpec, TestSpec unchanged |
