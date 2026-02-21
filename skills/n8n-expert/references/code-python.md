# Python Code Node Reference

Python patterns and limitations for n8n Code nodes.

---

## Important: JavaScript First

Use JavaScript for 95% of cases. Python only when you need specific standard library functions or are significantly more comfortable with Python.

---

## Quick Start

```python
items = _input.all()
return [{"json": {**item["json"], "processed": True}} for item in items]
```

**Rules**: Return `[{"json": {...}}]` | Webhook data under `_json["body"]` | No external libraries | Use `.get()` for safe dict access

---

## Data Access

```python
_input.all()                    # All items
_input.first()                  # First item
_input.item                     # Current item (each-item mode)
_node["Webhook"]["json"]        # Specific node
_json["body"]["field"]          # Webhook data (under "body"!)
_json.get("body", {}).get("field")  # Safe access
```

---

## Return Formats

```python
# CORRECT
return [{"json": {"field": value}}]                              # Single
return [{"json": {"id": 1}}, {"json": {"id": 2}}]               # Multiple
return [{"json": item["json"]} for item in _input.all()]         # Transformed
return []                                                         # Empty

# WRONG
return {"json": {"field": value}}     # Missing list wrapper
return [{"field": value}]             # Missing "json" key
```

---

## CRITICAL: No External Libraries

```python
# NOT AVAILABLE
import requests   # ModuleNotFoundError
import pandas     # ModuleNotFoundError
import numpy      # ModuleNotFoundError

# AVAILABLE (standard library only)
import json, datetime, re, base64, hashlib, urllib.parse, math, random, statistics
```

**Workarounds**: HTTP requests → use HTTP Request node or switch to JS (`$helpers.httpRequest`). Data analysis → use `statistics` module or manual calculations.

---

## Common Patterns

```python
# 1. Transform
return [{"json": {"id": item["json"].get("id"), "name": item["json"].get("name", "").upper()}} for item in _input.all()]

# 2. Aggregate
items = _input.all()
total = sum(item["json"].get("amount", 0) for item in items)
return [{"json": {"total": total, "count": len(items)}}]

# 3. Regex extraction
import re
emails = []
for item in _input.all():
    found = re.findall(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', item["json"].get("text", ""))
    emails.extend(found)
return [{"json": {"emails": list(set(emails))}}]

# 4. Validation
validated = []
for item in _input.all():
    data = item["json"]
    errors = []
    if not data.get("email"): errors.append("Email required")
    if not data.get("name"): errors.append("Name required")
    validated.append({"json": {**data, "valid": len(errors) == 0, "errors": errors or None}})
return validated

# 5. Statistics
from statistics import mean, median, stdev
values = [item["json"].get("value", 0) for item in _input.all() if "value" in item["json"]]
return [{"json": {"mean": mean(values), "median": median(values), "stdev": stdev(values) if len(values) > 1 else 0}}] if values else [{"json": {"error": "No values"}}]
```

---

## Python Modes

**Python (Beta)** - Recommended: `_input`, `_json`, `_node`, `_now`, `_jmespath()`
**Python (Native)**: `_items`, `_item` only. More limited.

---

## Top 5 Mistakes

1. **Importing external libraries**: Only standard library available
2. **Missing return**: Always end with `return [...]`
3. **Wrong return format**: `return [{"json": {...}}]` (list of dicts with "json" key)
4. **KeyError on dict access**: Use `.get("key", default)` instead of `["key"]`
5. **Webhook body nesting**: `_json["body"]["email"]`, not `_json["email"]`
