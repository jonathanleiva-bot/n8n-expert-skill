# Expression Syntax Reference

Complete guide for n8n expressions.

---

## Format

All dynamic content: `{{expression}}`

```javascript
{{$json.fieldName}}                      // Current node data
{{$json['field with spaces']}}           // Bracket notation for spaces
{{$json.nested.property}}                // Nested access
{{$json.items[0].name}}                  // Array access
{{$node["HTTP Request"].json.data}}      // Other node (case-sensitive, quotes required)
{{$now.toFormat('yyyy-MM-dd')}}          // Current date
{{$env.API_KEY}}                         // Environment variable
```

---

## CRITICAL: Webhook Data

Webhook data is under `.body`, NOT at root:

```javascript
// Webhook output structure:
// {headers: {...}, params: {...}, query: {...}, body: {name: "John", email: "john@example.com"}}

{{$json.body.name}}     // CORRECT
{{$json.name}}          // WRONG - undefined
```

---

## Core Variables

| Variable | Use | Example |
|----------|-----|---------|
| `$json` | Current node output | `{{$json.email}}` |
| `$node["Name"]` | Specific node output | `{{$node["HTTP Request"].json.data}}` |
| `$now` | Current timestamp (Luxon) | `{{$now.toFormat('yyyy-MM-dd')}}` |
| `$env` | Environment variables | `{{$env.API_KEY}}` |

---

## When NOT to Use Expressions

- **Code nodes**: Use JS/Python directly (`$json.field`, not `{{$json.field}}`)
- **Webhook paths**: Static only (`"my-webhook"`, not `"{{$json.id}}/webhook"`)
- **Credential fields**: Use n8n credential system

---

## Common Patterns

```javascript
// Concatenation (automatic)
Hello {{$json.body.name}}!

// JSON mode (= prefix)
{"email": "={{$json.body.email}}"}

// Conditionals
{{$json.status === 'active' ? 'Yes' : 'No'}}
{{$json.email || 'no-email@example.com'}}

// Date manipulation
{{$now.plus({days: 7}).toFormat('yyyy-MM-dd')}}
{{$now.minus({hours: 24}).toISO()}}

// String methods
{{$json.email.toLowerCase()}}
{{$json.name.toUpperCase()}}
{{$json.message.replace('old', 'new')}}

// Array
{{$json.users[0].email}}
{{$json.users.length}}
```

---

## Common Mistakes

| # | Mistake | Symptom | Fix |
|---|---------|---------|-----|
| 1 | `$json.email` | Literal text | `{{$json.email}}` |
| 2 | `{{$json.name}}` (webhook) | Undefined | `{{$json.body.name}}` |
| 3 | `{{$json.first name}}` | Syntax error | `{{$json['first name']}}` |
| 4 | `{{$node.HTTP Request.json}}` | Undefined | `{{$node["HTTP Request"].json}}` |
| 5 | `{{$node["http request"].json}}` | Undefined | Match exact case |
| 6 | `{{{$json.field}}}` | Literal braces | `{{$json.field}}` |
| 7 | `{{$json.items.0.name}}` | Syntax error | `{{$json.items[0].name}}` |
| 8 | `'{{$json.email}}'` in Code | Literal string | `$json.email` (no braces) |
| 9 | `{{$node[HTTP Request].json}}` | Syntax error | Add quotes |
| 10 | `={{$json.email}}` in text | Literal = | Remove = prefix |
| 11 | Missing `.json` in $node | Undefined | `{{$node["Name"].json.field}}` |

---

## Debugging

1. Check braces: wrapped in `{{}}`?
2. Webhook data? Add `.body`
3. Spaces? Use bracket notation
4. Case matches exactly?
5. Use expression editor (fx icon) for live preview
6. Code node? Remove `{{}}`

**Error messages**:
- "Cannot read property 'X' of undefined" → Parent object missing, check path
- "X is not a function" → Wrong variable type
- Expression shows as literal text → Missing `{{}}`
