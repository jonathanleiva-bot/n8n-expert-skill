---
name: n8n-expert
description: Unified expert guide for building n8n workflows. Use when creating, editing, validating, or debugging n8n workflows, writing n8n expressions with {{}} syntax, configuring nodes, writing JavaScript or Python in Code nodes, using n8n-mcp MCP tools (search_nodes, get_node, validate_node, n8n_create_workflow, n8n_update_partial_workflow), searching templates, managing workflow executions, or working with any n8n automation task.
---

# n8n Expert

Unified guide for building production-ready n8n workflows using n8n-mcp tools.

---

## MCP Tools Quick Reference

| Tool | Use When | Key Params |
|------|----------|------------|
| `search_nodes` | Finding nodes by keyword | `query`, `includeExamples` |
| `get_node` | Understanding node config | `nodeType`, `detail` (standard default) |
| `validate_node` | Checking node config | `nodeType`, `config`, `profile` (runtime) |
| `n8n_create_workflow` | Creating new workflows | `name`, `nodes[]`, `connections{}` |
| `n8n_update_partial_workflow` | Editing workflows (most used) | `id`, `operations[]`, `intent` |
| `n8n_validate_workflow` | Checking full workflow | `id` |
| `n8n_deploy_template` | Deploy from template library | `templateId` |
| `search_templates` | Finding workflow templates | `query` or `searchMode` |

For detailed tool usage, parameters, and patterns: [mcp-tools.md](references/mcp-tools.md)

---

## Critical: nodeType Formats

Two different formats for different tools:

```javascript
// Search/Validate tools (short prefix)
"nodes-base.slack"
"nodes-langchain.agent"

// Workflow tools (full prefix)
"n8n-nodes-base.slack"
"@n8n/n8n-nodes-langchain.agent"
```

`search_nodes` returns both: `nodeType` (short) and `workflowNodeType` (full).

---

## Workflow Building Process

```
1. search_nodes → find nodes
2. get_node (standard) → understand config
3. n8n_create_workflow → build with nodes + connections
4. n8n_validate_workflow → verify
5. n8n_update_partial_workflow → iterate (avg 56s between edits)
6. activateWorkflow → go live
```

Always iterate. Never build in one shot.

---

## 5 Core Workflow Patterns

| Pattern | Trigger | Flow | When to Use |
|---------|---------|------|-------------|
| **Webhook Processing** | Webhook | Receive → Validate → Transform → Respond | External events, form submissions |
| **HTTP API Integration** | Any | Trigger → HTTP Request → Transform → Action | Fetching external data |
| **Database Operations** | Schedule | Query → Transform → Write → Verify | ETL, data sync |
| **AI Agent** | Webhook/Chat | Trigger → AI Agent (Model+Tools+Memory) → Output | Chatbots, AI assistants |
| **Scheduled Tasks** | Schedule | Fetch → Process → Deliver → Log | Reports, maintenance |

For detailed patterns with examples: [workflow-patterns.md](references/workflow-patterns.md)

---

## Expression Syntax

All dynamic content uses `{{expression}}`:

```javascript
// Current node data
{{$json.fieldName}}
{{$json['field with spaces']}}

// Other nodes (case-sensitive, quotes required)
{{$node["HTTP Request"].json.data}}

// CRITICAL: Webhook data is under .body
{{$json.body.name}}     // CORRECT
{{$json.name}}          // WRONG - undefined!

// Date/time (Luxon)
{{$now.toFormat('yyyy-MM-dd')}}

// Conditionals
{{$json.status === 'active' ? 'Yes' : 'No'}}
```

**Rules**:
- Always wrap in `{{}}`
- Webhook data lives under `.body`
- No `{{}}` inside Code nodes (use JS/Python directly)
- Node names are case-sensitive

For complete syntax, mistakes, and examples: [expressions.md](references/expressions.md)

---

## Node Configuration

**Progressive disclosure**: Start with `get_node` (standard detail ~1-2K tokens), escalate only if needed.

**Key concept**: Resource + operation determine which fields are required.

```javascript
// Slack: different operations need different fields
{resource: "message", operation: "post"}    // needs: channel, text
{resource: "message", operation: "update"}  // needs: messageId, text

// HTTP Request: method determines body visibility
{method: "GET", url: "..."}                 // no body
{method: "POST", url: "...", sendBody: true, body: {...}}  // body required
```

**Configuration workflow**: get_node → configure required fields → validate → fix → repeat (2-3 cycles normal).

For dependency patterns and node-specific configs: [node-configuration.md](references/node-configuration.md)

---

## Validation

**Philosophy**: Validate early, validate often. Expect 2-3 iterations.

**Profiles** (choose by stage):
- `minimal` - Quick checks during editing
- `runtime` - **Recommended** for pre-deployment
- `ai-friendly` - Fewer false positives for AI workflows
- `strict` - Production/critical workflows

**Error types**:
- `missing_required` → Add the field
- `invalid_value` → Check allowed values with get_node
- `type_mismatch` → Convert to correct type
- `invalid_expression` → Check `{{}}` syntax
- `invalid_reference` → Fix node name spelling

**Auto-sanitization**: Runs on every workflow save. Fixes operator structures (binary/unary), IF/Switch metadata. Cannot fix broken connections or branch mismatches.

For error catalog, false positives, and recovery strategies: [validation.md](references/validation.md)

---

## Code Nodes

### JavaScript (Recommended - 95% of use cases)

```javascript
// Basic template - "Run Once for All Items" mode
const items = $input.all();

return items.map(item => ({
  json: {
    ...item.json,
    processed: true
  }
}));
```

**Essential rules**:
- Must return `[{json: {...}}]` format
- Webhook data: `$json.body.field` (not `$json.field`)
- No `{{}}` expressions - use JS directly
- Built-ins: `$helpers.httpRequest()`, `DateTime` (Luxon), `$jmespath()`
- Use `$input.all()` for batch, `$input.first()` for single, `$input.item` for each-item mode

### Python (Only when needed)

```python
# Same structure but with _input instead of $input
items = _input.all()

return [{"json": {**item["json"], "processed": True}} for item in items]
```

**Python limitations**:
- No external libraries (no requests, pandas, numpy)
- Standard library only: json, datetime, re, base64, hashlib, math, statistics
- Use `.get()` for safe dict access
- Prefer JavaScript for HTTP requests and dates

For complete patterns, error prevention, and built-in functions:
- [code-javascript.md](references/code-javascript.md)
- [code-python.md](references/code-python.md)

---

## Smart Workflow Editing

Use `n8n_update_partial_workflow` with smart parameters:

```javascript
// IF node - semantic branch names
{type: "addConnection", source: "IF", target: "Handler", branch: "true"}

// Switch node - case numbers
{type: "addConnection", source: "Switch", target: "Handler", case: 0}

// Always include intent for better responses
n8n_update_partial_workflow({
  id: "abc",
  intent: "Add error handling for API failures",
  operations: [{type: "addNode", node: {...}}]
})
```

**Available operations**: addNode, updateNode, deleteNode, addConnection, removeConnection, cleanStaleConnections, activateWorkflow, deactivateWorkflow, and more.

---

## Templates

```javascript
// Search by keyword
search_templates({query: "webhook slack"})

// Search by node types
search_templates({searchMode: "by_nodes", nodeTypes: ["n8n-nodes-base.slack"]})

// Search by task
search_templates({searchMode: "by_task", task: "ai_automation"})

// Deploy directly to instance
n8n_deploy_template({templateId: 2947, name: "My Workflow", autoFix: true})
```

---

## Common Mistakes Cheat Sheet

| Mistake | Fix |
|---------|-----|
| `$json.name` (webhook) | `$json.body.name` |
| `{{$json.field}}` in Code node | `$json.field` (no braces) |
| `get_node({nodeType: "slack"})` | `get_node({nodeType: "nodes-base.slack"})` |
| `n8n-nodes-base.slack` in validate | `nodes-base.slack` (short prefix) |
| `nodes-base.slack` in create_workflow | `n8n-nodes-base.slack` (full prefix) |
| `detail: "full"` by default | `detail: "standard"` (95% sufficient) |
| Building workflow in one shot | Iterate with partial updates |
| Skipping validation | Always validate before activation |
| `return {json: {...}}` in Code | `return [{json: {...}}]` (array!) |

---

## Best Practices

**Do**:
- Use `get_node` with standard detail (default)
- Specify validation profile (`runtime` recommended)
- Use smart parameters (`branch`, `case`) for connections
- Include `intent` in workflow updates
- Validate after every significant change
- Build workflows iteratively
- Use `includeExamples: true` for real configs

**Don't**:
- Use `detail: "full"` unless necessary
- Skip nodeType prefix
- Build workflows in one shot
- Use `{{}}` expressions inside Code nodes
- Assume webhook data is at root (it's under `.body`)
- Deploy without validation

---

## Reference Files

Detailed guides loaded on demand:

- **[mcp-tools.md](references/mcp-tools.md)** - Complete MCP tool usage, parameters, patterns
- **[expressions.md](references/expressions.md)** - Expression syntax, variables, mistakes, examples
- **[workflow-patterns.md](references/workflow-patterns.md)** - 5 core patterns with detailed examples
- **[validation.md](references/validation.md)** - Error catalog, profiles, auto-sanitization, recovery
- **[node-configuration.md](references/node-configuration.md)** - Property dependencies, operation patterns
- **[code-javascript.md](references/code-javascript.md)** - JS patterns, built-ins, error prevention
- **[code-python.md](references/code-python.md)** - Python patterns, limitations, standard library
