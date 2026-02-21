# MCP Tools Reference

Complete guide for n8n-mcp MCP server tools.

---

## Node Discovery

### search_nodes
```javascript
search_nodes({
  query: "slack",           // Required
  mode: "OR",               // OR (default), AND, FUZZY
  limit: 20,                // Max results (default 20)
  source: "all",            // all, core, community, verified
  includeExamples: false    // Include template configs
})
```
Returns `nodeType` (short prefix) and `workflowNodeType` (full prefix).

### get_node

**Detail levels** (mode="info"):
| Detail | Tokens | Use |
|--------|--------|-----|
| `minimal` | ~200 | Quick metadata |
| `standard` | ~1-2K | **Default - 95% of cases** |
| `full` | ~3-8K | Complex debugging only |

**Modes**:
- `info` (default) - Schema with detail level
- `docs` - Readable markdown documentation
- `search_properties` - Find specific fields (`propertyQuery` required)
- `versions` - Version history
- `compare` - Compare two versions (`fromVersion`, `toVersion`)
- `breaking` - Breaking changes only
- `migrations` - Auto-migratable changes

```javascript
get_node({nodeType: "nodes-base.slack"})                           // Standard (default)
get_node({nodeType: "nodes-base.slack", mode: "docs"})             // Documentation
get_node({nodeType: "nodes-base.httpRequest", mode: "search_properties", propertyQuery: "auth"})
get_node({nodeType: "nodes-base.slack", includeExamples: true})    // With template configs
```

---

## Validation

### validate_node
```javascript
validate_node({
  nodeType: "nodes-base.slack",
  config: {resource: "channel", operation: "create"},
  mode: "full",            // full (default), minimal
  profile: "runtime"       // minimal, runtime (recommended), ai-friendly, strict
})
```

### validate_workflow
```javascript
validate_workflow({
  workflow: {nodes: [...], connections: {...}},
  options: {validateNodes: true, validateConnections: true, validateExpressions: true, profile: "runtime"}
})
```

### n8n_validate_workflow (by ID)
```javascript
n8n_validate_workflow({id: "workflow-id", options: {profile: "runtime"}})
```

### n8n_autofix_workflow
```javascript
n8n_autofix_workflow({id: "workflow-id", applyFixes: false})  // Preview
n8n_autofix_workflow({id: "workflow-id", applyFixes: true})   // Apply
```

---

## Workflow Management

### n8n_create_workflow
```javascript
n8n_create_workflow({
  name: "Webhook to Slack",
  nodes: [
    {id: "webhook-1", name: "Webhook", type: "n8n-nodes-base.webhook", typeVersion: 2,
     position: [250, 300], parameters: {path: "slack-notify", httpMethod: "POST"}},
    {id: "slack-1", name: "Slack", type: "n8n-nodes-base.slack", typeVersion: 2,
     position: [450, 300], parameters: {resource: "message", operation: "post", channel: "#general", text: "={{$json.body.message}}"}}
  ],
  connections: {"Webhook": {"main": [[{node: "Slack", type: "main", index: 0}]]}}
})
```
Workflows created **inactive**. Use `activateWorkflow` operation to activate.

### n8n_update_partial_workflow (MOST USED)

**17 operation types**:

Node: `addNode`, `removeNode`, `updateNode`, `moveNode`, `enableNode`, `disableNode`
Connection: `addConnection`, `removeConnection`, `rewireConnection`, `cleanStaleConnections`, `replaceConnections`
Metadata: `updateSettings`, `updateName`, `addTag`, `removeTag`
Activation: `activateWorkflow`, `deactivateWorkflow`

**Smart parameters**:
```javascript
// IF node branches
{type: "addConnection", source: "IF", target: "Handler", branch: "true"}
{type: "addConnection", source: "IF", target: "Handler", branch: "false"}

// Switch node cases
{type: "addConnection", source: "Switch", target: "Handler", case: 0}

// AI connections (8 types: ai_languageModel, ai_tool, ai_memory, ai_outputParser, ai_embedding, ai_vectorStore, ai_document, ai_textSplitter)
{type: "addConnection", source: "OpenAI Chat Model", target: "AI Agent", sourceOutput: "ai_languageModel"}
```

**Always include intent**:
```javascript
n8n_update_partial_workflow({
  id: "abc", intent: "Add error handling", operations: [...]
})
```

**Options**: `continueOnError: true` (best-effort), `validateOnly: true` (preview)

### n8n_deploy_template
```javascript
n8n_deploy_template({templateId: 2947, name: "My Workflow", autoFix: true, autoUpgradeVersions: true})
```

### n8n_workflow_versions
```javascript
n8n_workflow_versions({mode: "list", workflowId: "id", limit: 10})
n8n_workflow_versions({mode: "rollback", workflowId: "id", versionId: 123})
n8n_workflow_versions({mode: "prune", workflowId: "id", maxVersions: 10})
```

### n8n_test_workflow
```javascript
n8n_test_workflow({workflowId: "id", data: {message: "Hello!"}, waitForResponse: true})
n8n_test_workflow({workflowId: "id", triggerType: "chat", message: "Hello AI!", sessionId: "s1"})
```

### n8n_executions
```javascript
n8n_executions({action: "list", workflowId: "id", status: "error", limit: 100})
n8n_executions({action: "get", id: "exec-id", mode: "error", includeStackTrace: true})
n8n_executions({action: "delete", id: "exec-id"})
```

---

## Templates

```javascript
search_templates({query: "webhook slack"})
search_templates({searchMode: "by_nodes", nodeTypes: ["n8n-nodes-base.slack"]})
search_templates({searchMode: "by_task", task: "ai_automation"})
search_templates({searchMode: "by_metadata", complexity: "simple", maxSetupMinutes: 15})
get_template({templateId: 2947, mode: "structure"})
```

---

## Tool Availability

**Always available** (no API needed): search_nodes, get_node, validate_node, validate_workflow, search_templates, get_template, tools_documentation, ai_agents_guide

**Requires N8N_API_URL + N8N_API_KEY**: n8n_create_workflow, n8n_update_partial_workflow, n8n_validate_workflow, n8n_list_workflows, n8n_get_workflow, n8n_test_workflow, n8n_executions, n8n_deploy_template, n8n_workflow_versions, n8n_autofix_workflow

---

## Self-Help

```javascript
tools_documentation()                                           // All tools overview
tools_documentation({topic: "search_nodes", depth: "full"})     // Specific tool
tools_documentation({topic: "javascript_code_node_guide"})      // Code node guides
ai_agents_guide()                                               // AI workflow guide
n8n_health_check({mode: "diagnostic"})                          // Connection diagnostics
```
