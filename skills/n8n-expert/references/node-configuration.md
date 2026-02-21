# Node Configuration Reference

Operation-aware configuration with property dependencies.

---

## Core Concept: Operation-Aware Config

Resource + operation determine required fields:

```javascript
// Slack: "post" needs channel+text, "update" needs messageId+text
{resource: "message", operation: "post", channel: "#general", text: "Hello!"}
{resource: "message", operation: "update", messageId: "123", text: "Updated!"}
```

---

## Configuration Workflow

```
1. get_node (standard detail) → see operations + fields
2. Configure required fields for chosen operation
3. validate_node → check config
4. If field unclear → get_node({mode: "search_properties", propertyQuery: "..."})
5. Fix errors → validate again (2-3 cycles normal)
```

---

## Property Dependencies (displayOptions)

Fields appear/disappear based on other values:

```javascript
// HTTP Request: method controls body visibility
{method: "GET", url: "..."}                    // no body fields
{method: "POST", url: "...", sendBody: true}   // body required

// sendBody controls body requirement
{sendBody: true}  → body field appears and is required
{sendBody: false} → body field hidden
```

**Finding dependencies**: `get_node({mode: "search_properties", propertyQuery: "body"})`

---

## Common Node Patterns

### Resource/Operation Nodes (Slack, Google Sheets, Airtable)
```javascript
{resource: "<entity>", operation: "<action>", ...operationSpecificFields}
```

### HTTP-Based (HTTP Request, Webhook)
```javascript
{method: "POST", url: "...", authentication: "none", sendBody: true, body: {contentType: "json", content: {...}}}
// POST/PUT/PATCH → sendBody available; sendBody=true → body required; auth != "none" → credentials required
```

### Database (Postgres, MySQL, MongoDB)
```javascript
{operation: "executeQuery", query: "SELECT ..."}       // query required
{operation: "insert", table: "users", columns: "..."}  // table+values required
```

### Conditional Logic (IF, Switch)
```javascript
// Binary (equals, contains): value1 + value2
{conditions: {string: [{value1: "={{$json.status}}", operation: "equals", value2: "active"}]}}

// Unary (isEmpty, isNotEmpty): value1 only + singleValue: true (auto-added by sanitization)
{conditions: {string: [{value1: "={{$json.email}}", operation: "isEmpty"}]}}
```

---

## get_node Detail Decision Tree

```
Starting config? → get_node (standard, default)
  ↓ Has what you need? → Configure
  ↓ Looking for specific field? → search_properties mode
  ↓ Still need more? → get_node({detail: "full"})
```

---

## Anti-Patterns

- **Over-configure upfront**: Start minimal, add as needed
- **Skip validation**: Always validate before deploy
- **Ignore operation context**: Different operations = different fields
- **Copy configs blindly**: Validate after copying
- **Manually fix auto-sanitization**: Let it handle operator structure
