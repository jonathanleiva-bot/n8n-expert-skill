# Validation Reference

Error catalog, profiles, auto-sanitization, and recovery strategies.

---

## Profiles

| Profile | Use When | Strictness |
|---------|----------|------------|
| `minimal` | Quick checks during editing | Lenient |
| `runtime` | **Pre-deployment (RECOMMENDED)** | Balanced |
| `ai-friendly` | AI-generated configs | Fewer false positives |
| `strict` | Production/critical | Maximum |

---

## Error Types (Must Fix)

### missing_required (45% of errors)
```json
{"type": "missing_required", "property": "channel", "message": "Channel name is required"}
```
**Fix**: Add field. Use `get_node` to see required fields.

### invalid_value (28%)
```json
{"type": "invalid_value", "property": "operation", "current": "send", "allowed": ["post", "update", "delete"]}
```
**Fix**: Use valid enum value. Values are case-sensitive.

### type_mismatch (12%)
```json
{"type": "type_mismatch", "property": "limit", "expected": "number", "current": "100"}
```
**Fix**: Convert type (`"100"` → `100`, `"true"` → `true`).

### invalid_expression (8%)
```json
{"type": "invalid_expression", "property": "text", "current": "$json.name"}
```
**Fix**: Wrap in `{{}}`, check node references, verify `.body` for webhooks.

### invalid_reference (5%)
```json
{"type": "invalid_reference", "message": "Node 'Transform Data' does not exist"}
```
**Fix**: Update reference or use `cleanStaleConnections`.

---

## Warnings (Should Fix)

- `best_practice` - Error handling, retry logic
- `deprecated` - Old typeVersion
- `performance` - Unbounded queries, missing LIMIT

---

## Auto-Sanitization

**Runs on**: Every workflow create/update.

**Fixes automatically**:
- Binary operators (equals, contains) → removes `singleValue`
- Unary operators (isEmpty, isNotEmpty) → adds `singleValue: true`
- IF v2.2+ / Switch v3.2+ → adds `conditions.options` metadata

**Cannot fix**: Broken connections, branch count mismatches, corrupt states.

**Recovery**: `cleanStaleConnections` operation, `n8n_autofix_workflow`

---

## Validation Loop

```
Configure → validate_node (23s thinking) → Fix errors → validate again (58s fixing) → Repeat (2-3 iterations normal)
```

---

## False Positives

Use `ai-friendly` profile to reduce. Acceptable to ignore:
- "Missing error handling" in simple/test workflows
- "No retry logic" for idempotent operations
- "Missing rate limiting" for internal/low-volume APIs
- "Unbounded query" on small known datasets

---

## Recovery Strategies

1. **Start fresh**: Get requirements from `get_node`, build minimal config, add incrementally
2. **Binary search**: Remove half nodes, test, isolate problem
3. **Clean connections**: `{type: "cleanStaleConnections"}`
4. **Auto-fix**: `n8n_autofix_workflow({id, applyFixes: true})`

---

## Workflow-Level Errors

- **Broken connections**: Target node not found → `cleanStaleConnections`
- **Circular dependencies**: A → B → A → restructure workflow
- **Multiple triggers**: Only one executes → remove extras or split workflows
- **Disconnected nodes**: Not in flow → connect or remove
