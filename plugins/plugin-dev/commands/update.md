---
description: Create or modify plugins interactively
argument-hint: <what you want to change>
---

Create or modify plugins in this marketplace. Use subagents to minimize context usage.

User request: $ARGUMENTS

## Step 1: Discovery (Subagent)

Spawn a subagent (subagent_type: "Explore", model: "haiku") to discover the marketplace and its plugins:

- Find where `plugin-dev` is installed via `~/.claude/plugins/installed_plugins.json`
- Get marketplace config from `~/.claude/plugins/known_marketplaces.json`
- Verify source type is "directory" (editable)
- List all plugins with their commands and hooks

If read-only, inform user and stop.

## Step 2: Gather Requirements (Main Thread)

Present discovered plugins to user. Use **AskUserQuestion** to clarify:
- Create new plugin, add to existing, or modify?
- Command, hook, or other files?
- Name, purpose, arguments?
- Use existing command as template?

## Step 3: Plan (Subagent)

Spawn planning subagent(s) with the requirements. Parallelize if multiple independent changes.

Output should include complete file contents, not placeholders.

Present plan for approval.

## Step 4: Approval (Main Thread)

Ask user if plan looks good. Options: Proceed, Modify, Cancel.

## Step 5: Execute (Subagent)

Spawn implementation subagent(s). Parallelize independent file operations.

For new plugins, also update:
- Marketplace's `.claude-plugin/marketplace.json`
- `~/.claude/settings.json` to enable the plugin

## Step 6: Summary (Main Thread)

Report changes. Don't commit - let user review first.

## Conventions

- Plugin names: lowercase (git, learn, meta)
- Command names: action verbs (commit, explore, trace)
- Commands: `commands/<name>.md` with frontmatter
- Hooks: `hooks/<name>.sh` or `hooks.json`
