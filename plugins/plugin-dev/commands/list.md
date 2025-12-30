---
description: List all plugins in this marketplace
---

List all plugins in the plugin-dev marketplace using a subagent.

## Discovery (Subagent)

Spawn a subagent (subagent_type: "Explore", model: "haiku") to discover and list plugins:

```
Find and list all plugins in the plugin-dev marketplace:

1. Read `~/.claude/plugins/installed_plugins.json` - find key matching `plugin-dev@*`
2. Extract marketplace name (part after `@`)
3. Read `~/.claude/plugins/known_marketplaces.json` - get marketplace path
4. List all directories in `<marketplace-path>/plugins/`
5. For each plugin:
   - Read `.claude-plugin/plugin.json` for name and description
   - List files in `commands/` directory
   - List files in `hooks/` directory (if exists)

Return a table:
| Plugin | Description | Commands | Hooks |
```

## Present Results

Display the subagent's findings as a formatted table to the user.
