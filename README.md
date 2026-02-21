# n8n Expert Skill for Claude Code

Unified skill for building production-ready n8n workflows using Claude Code and n8n-mcp tools.

## Install

```bash
npx skills add tu-usuario/n8n-expert-skill
```

Or manually copy `skills/n8n-expert/` to `~/.claude/skills/n8n-expert/`.

## What's Included

A single unified skill consolidating 7 specialized n8n guides:

| Reference | Covers |
|-----------|--------|
| `mcp-tools.md` | search_nodes, get_node, validate_node, workflow management |
| `expressions.md` | `{{}}` syntax, variables, common mistakes |
| `workflow-patterns.md` | 5 core patterns: webhook, HTTP, database, AI agent, scheduled |
| `validation.md` | Error catalog, profiles, auto-sanitization, recovery |
| `node-configuration.md` | Property dependencies, operation-aware config |
| `code-javascript.md` | JS Code node patterns, built-ins, error prevention |
| `code-python.md` | Python Code node patterns, standard library only |

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [n8n-mcp](https://www.npmjs.com/package/n8n-mcp) server configured in Claude Code

## Structure

```
skills/
  n8n-expert/
    SKILL.md              # Core guide (287 lines)
    references/
      mcp-tools.md        # MCP tool usage & patterns
      expressions.md      # Expression syntax & mistakes
      workflow-patterns.md # 5 architectural patterns
      validation.md       # Error types & recovery
      node-configuration.md # Property dependencies
      code-javascript.md  # JS Code node reference
      code-python.md      # Python Code node reference
```

## License

MIT
