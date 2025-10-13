# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code agents repository containing specialized AI agents designed to enhance development workflows. The repository follows the Claude Code plugin/agent architecture and is designed for distribution through the Claude Code marketplace.

## Repository Structure

```
aztack-claude-code-agents/
├── .claude-plugin/
│   └── marketplace.json          # Plugin marketplace configuration
└── agents/
    └── {agent-name}/
        └── {agent-name}.md       # Agent definition with frontmatter
```

## Agent Architecture

### Agent Definition Format

Each agent is defined in a markdown file with YAML frontmatter following this structure:

```yaml
---
name: agent-name
description: |
  Agent description with usage examples...
model: inherit
---
```

The frontmatter fields:
- `name`: Unique identifier for the agent (kebab-case)
- `description`: Detailed description including usage examples in `<example>` tags
- `model`: Model selection (typically "inherit" to use Claude Code's default model)

The markdown body after the frontmatter contains the agent's system prompt and instructions.

### Marketplace Configuration

The `.claude-plugin/marketplace.json` file registers all agents and follows this structure:

```json
{
  "name": "repository-name",
  "owner": { "name": "...", "email": "...", "url": "..." },
  "metadata": { "description": "...", "version": "..." },
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./",
      "description": "...",
      "version": "...",
      "author": { "name": "...", "url": "..." },
      "homepage": "...",
      "repository": "...",
      "license": "MIT",
      "keywords": [...],
      "category": "...",
      "strict": false,
      "commands": [],
      "agents": ["./agents/agent-name/agent-name.md"]
    }
  ]
}
```

## Adding New Agents

When creating a new agent:

1. Create a new directory under `agents/` with the agent name (kebab-case)
2. Create the agent definition file: `agents/{agent-name}/{agent-name}.md`
3. Add YAML frontmatter with `name`, `description`, and `model` fields
4. Write the agent's system prompt in the markdown body
5. Update `.claude-plugin/marketplace.json`:
   - Add a new plugin entry or add to existing plugin's `agents` array
   - Include the path: `./agents/{agent-name}/{agent-name}.md`
   - Set appropriate metadata: description, keywords, category

## Existing Agents

### git-commit-generator

Located at: `agents/git-commit-generator/git-commit-generator.md`

A specialized agent that analyzes git diffs and generates Conventional Commits compliant commit messages. Key features:
- Analyzes `git diff` output (never uses `--staged`)
- Classifies changes by package/module and functionality
- Generates structured commit groups with unique identifiers
- Supports two commands:
  - `/commit_message [optional_diff]`: Analyze and generate commits
  - `/commit <name> <type>(<scope>): <msg>`: Commit specific file group
- Enforces strict Conventional Commits format with types: feat, fix, docs, style, refactor, perf, test, chore
- Output includes: unique name, classification summary, and ready-to-use commit messages

## Development Workflow

This repository contains no build, test, or compilation steps as it consists only of markdown agent definitions and JSON configuration files.

When modifying agents:
- Ensure agent markdown files maintain valid YAML frontmatter
- Keep agent names consistent across filename, frontmatter `name`, and marketplace.json references
- Test agents through Claude Code by using the Task tool to invoke them
- Validate marketplace.json syntax after modifications
