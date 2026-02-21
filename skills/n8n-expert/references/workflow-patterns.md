# Workflow Patterns Reference

5 core architectural patterns for n8n workflows.

---

## 1. Webhook Processing (Most Common - 35%)

**Pattern**: Webhook → Validate → Transform → Respond/Notify

```javascript
// Minimal example
nodes: [
  {name: "Webhook", type: "n8n-nodes-base.webhook", parameters: {path: "form-submit", httpMethod: "POST"}},
  {name: "Set", type: "n8n-nodes-base.set", parameters: {/* map fields */}},
  {name: "Slack", type: "n8n-nodes-base.slack", parameters: {resource: "message", operation: "post", channel: "#notifications", text: "={{$json.body.message}}"}}
]
```

**Key**: Webhook data is under `$json.body`

**Use for**: Form submissions, payment webhooks, Slack commands, GitHub events

---

## 2. HTTP API Integration

**Pattern**: Trigger → HTTP Request → Transform → Action → Error Handler

```javascript
// Fetch + transform + store
nodes: [
  {name: "Manual Trigger", type: "n8n-nodes-base.manualTrigger"},
  {name: "HTTP Request", type: "n8n-nodes-base.httpRequest", parameters: {method: "GET", url: "https://api.example.com/users"}},
  {name: "Set", type: "n8n-nodes-base.set", parameters: {/* transform */}},
  {name: "Postgres", type: "n8n-nodes-base.postgres", parameters: {operation: "insert", table: "users"}}
]
```

**Use for**: Fetching external data, syncing services, data pipelines

---

## 3. Database Operations

**Pattern**: Schedule → Query → Transform → Write → Verify

```javascript
nodes: [
  {name: "Schedule", type: "n8n-nodes-base.scheduleTrigger", parameters: {rule: {interval: [{field: "minutes", minutesInterval: 15}]}}},
  {name: "Postgres", type: "n8n-nodes-base.postgres", parameters: {operation: "executeQuery", query: "SELECT * FROM new_records"}},
  {name: "IF", type: "n8n-nodes-base.if", parameters: {/* check records exist */}},
  {name: "MySQL", type: "n8n-nodes-base.mySql", parameters: {operation: "insert"}}
]
```

**Use for**: ETL, data sync, backup workflows

---

## 4. AI Agent Workflow

**Pattern**: Trigger → AI Agent (Model + Tools + Memory) → Output

```javascript
nodes: [
  {name: "Webhook", type: "n8n-nodes-base.webhook"},
  {name: "AI Agent", type: "@n8n/n8n-nodes-langchain.agent"},
  {name: "OpenAI", type: "@n8n/n8n-nodes-langchain.lmChatOpenAi"},
  {name: "HTTP Tool", type: "@n8n/n8n-nodes-langchain.toolHttpRequest"},
  {name: "Memory", type: "@n8n/n8n-nodes-langchain.memoryBufferWindow"},
  {name: "Response", type: "n8n-nodes-base.respondToWebhook"}
]
// AI connections use sourceOutput: "ai_languageModel", "ai_tool", "ai_memory"
```

**Use for**: Chatbots, content generation, data analysis with AI

---

## 5. Scheduled Tasks (28%)

**Pattern**: Schedule → Fetch → Process → Deliver → Log

```javascript
nodes: [
  {name: "Schedule", type: "n8n-nodes-base.scheduleTrigger", parameters: {rule: {interval: [{field: "hours", hoursInterval: 24}]}}},
  {name: "HTTP Request", type: "n8n-nodes-base.httpRequest", parameters: {/* fetch analytics */}},
  {name: "Code", type: "n8n-nodes-base.code", parameters: {/* aggregate */}},
  {name: "Email", type: "n8n-nodes-base.emailSend", parameters: {/* send report */}}
]
```

**Use for**: Daily reports, periodic data fetching, maintenance

---

## Data Flow Patterns

```
Linear:    Trigger → Transform → Action → End
Branching: Trigger → IF → [True Path] / [False Path]
Parallel:  Trigger → [Branch 1] + [Branch 2] → Merge
Loop:      Trigger → Split In Batches → Process → Loop
Error:     Main Flow → [Success] / [Error Trigger → Handler]
```

---

## Common Components

**Triggers**: Webhook (35%), Schedule (28%), Manual (22%), Polling (15%)
**Transform**: Set (68%), Code (42%), IF (38%), Switch (18%)
**Output**: HTTP Request (45%), Slack (32%), Database (28%), Email (24%)
**Error**: Error Trigger, IF checks, Stop and Error, Continue On Fail

---

## Workflow Creation Checklist

**Plan**: Identify pattern → List nodes (search_nodes) → Plan data flow → Plan error handling
**Build**: Create with trigger → Add data sources → Configure auth → Add transforms → Add outputs → Add error handling
**Validate**: validate_node each → validate_workflow → Test with sample data
**Deploy**: Review settings → activateWorkflow → Monitor executions

---

## Common Gotchas

1. **Webhook data**: Under `$json.body`, not `$json`
2. **Multiple items**: Use "Execute Once" or `$json[0]` for single item
3. **Auth**: Use Credentials section, not parameters
4. **Execution order**: Use v1 (connection-based, recommended)
5. **Expressions as text**: Missing `{{}}`
