# JavaScript Code Node Reference

Patterns, built-ins, and error prevention for n8n Code nodes.

---

## Quick Start

```javascript
const items = $input.all();
return items.map(item => ({json: {...item.json, processed: true}}));
```

**Rules**: Return `[{json: {...}}]` | Webhook data under `$json.body` | No `{{}}` in Code nodes

---

## Mode Selection

**Run Once for All Items** (default, 95% of cases): `$input.all()`, `$input.first()`
**Run Once for Each Item**: `$input.item` - only when items are completely independent

---

## Data Access

```javascript
$input.all()                              // All items (batch)
$input.first()                            // First item only
$input.item                               // Current item (each-item mode)
$node["Webhook"].json                     // Specific node data
$json.body.field                          // Webhook data (under .body!)
```

---

## Return Formats

```javascript
// CORRECT
return [{json: {field: value}}];                    // Single result
return [{json: {id: 1}}, {json: {id: 2}}];         // Multiple
return items.map(i => ({json: {id: i.json.id}}));   // Transformed
return [];                                          // Empty (valid)

// WRONG
return {json: {field: value}};     // Missing array wrapper
return [{field: value}];           // Missing json key
return "processed";                // Not an array of objects
```

---

## Built-in Functions

### $helpers.httpRequest()
```javascript
const response = await $helpers.httpRequest({
  method: 'POST',
  url: 'https://api.example.com/data',
  headers: {'Authorization': 'Bearer ' + $env.API_KEY, 'Content-Type': 'application/json'},
  body: {name: $json.body.name},
  qs: {page: 1},
  timeout: 10000,
  simple: false  // Don't throw on HTTP errors
});
return [{json: {data: response}}];
```

### DateTime (Luxon)
```javascript
const now = DateTime.now();
now.toISO()                                    // "2025-01-20T15:30:00.000Z"
now.toFormat('yyyy-MM-dd')                     // "2025-01-20"
now.plus({days: 7}).toFormat('yyyy-MM-dd')     // Add 7 days
now.minus({hours: 24}).toISO()                 // Subtract 24 hours
now.startOf('month').toISO()                   // Start of month
now.setZone('America/New_York').toISO()        // Timezone conversion
DateTime.fromISO('2025-01-20').toFormat('MMMM dd, yyyy')  // Parse + format
```

### $jmespath()
```javascript
const data = $input.first().json;
$jmespath(data, 'users[*].name')                           // Extract field
$jmespath(data, 'users[?age >= `18`]')                     // Filter
$jmespath(data, 'users | sort_by(@, &score) | [0:5]')     // Sort + slice
```

### $getWorkflowStaticData()
```javascript
const staticData = $getWorkflowStaticData();
staticData.counter = (staticData.counter || 0) + 1;  // Persists across executions
```

### Available modules
- `crypto`: `require('crypto')` - hashing, random bytes
- `Buffer`: Base64 encoding/decoding
- `URL`, `URLSearchParams`: URL parsing

### NOT available
- No axios, lodash, moment, request, or any npm package
- Use `$helpers.httpRequest()` instead of axios/request
- Use `DateTime` (Luxon) instead of moment

---

## Common Patterns

```javascript
// 1. Aggregation
const total = $input.all().reduce((sum, item) => sum + (item.json.amount || 0), 0);
return [{json: {total, count: $input.all().length}}];

// 2. Filtering
return $input.all().filter(i => i.json.status === 'active').map(i => ({json: i.json}));

// 3. Data transformation
return $input.all().map(item => {
  const [first, ...rest] = item.json.name.split(' ');
  return {json: {first_name: first, last_name: rest.join(' '), email: item.json.email}};
});

// 4. HTTP request with error handling
try {
  const response = await $helpers.httpRequest({url: 'https://api.example.com/data'});
  return [{json: {success: true, data: response}}];
} catch (error) {
  return [{json: {success: false, error: error.message}}];
}

// 5. Deduplication
const seen = new Set();
return $input.all().filter(item => {
  if (seen.has(item.json.id)) return false;
  seen.add(item.json.id);
  return true;
}).map(i => ({json: i.json}));
```

---

## Top 5 Mistakes

1. **Missing return**: Always end with `return [...]`
2. **Expression syntax in Code**: Use `$json.field`, not `"{{$json.field}}"`
3. **Wrong return format**: `return [{json: {...}}]` (array of objects with json key)
4. **Missing null checks**: Use `item.json?.user?.email || 'default'`
5. **Webhook body nesting**: `$json.body.email`, not `$json.email`

---

## When to Use Code Node

Use: Complex transforms, custom logic, aggregation, recursive operations, multi-step conditionals
Don't use: Simple field mapping (Set), basic filtering (Filter), simple conditionals (IF/Switch), HTTP only (HTTP Request)
