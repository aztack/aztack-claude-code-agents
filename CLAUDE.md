# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code agents repository containing specialized AI agents designed to enhance development workflows. The repository follows the Claude Code plugin/agent architecture with a plugin-based structure where each plugin can contain multiple agents and slash commands.

## Repository Structure

```
aztack-claude-code-agents/
├── .claude/
│   └── agents/
│       └── agent-creator.md         # Meta-agent for creating agents
├── .claude-plugin/
│   └── marketplace.json             # Central plugin registry
└── plugins/
    ├── git-commit-generator/
    │   ├── agents/
    │   │   └── git-commit-generator.md
    │   └── commands/
    │       ├── commit.md
    │       ├── commit_all.md
    │       └── commit_message.md
    ├── knowledge-researcher/
    │   ├── agents/
    │   │   └── knowledge-researcher.md
    │   └── commands/
    │       └── research.md
    └── d2c-generator/
        ├── agents/
        │   └── d2c-generator.md
        └── commands/
            ├── ocr.md
            └── implComponents.md
```

## Plugin Architecture

### Plugin Structure

Each plugin follows this pattern:
```
plugins/{plugin-name}/
├── agents/
│   └── {agent-name}.md              # Agent with YAML frontmatter
└── commands/                         # Optional slash commands
    └── {command-name}.md             # Command implementation
```

### Agent Definition Format

Agent files use YAML frontmatter:

```yaml
---
name: agent-name
description: |
  Agent description with usage examples in <example> tags.

  <example>
  Context: Scenario description
  user: "User input"
  assistant: "Agent response"
  </example>
model: inherit
---

# Agent System Prompt
[Detailed instructions and behavior definition]
```

### Command Definition Format

Command files also use YAML frontmatter:

```yaml
---
description: Command description
argument-hint: "<arg-pattern>"
---

# Command Implementation
[Command behavior and processing logic]
```

### Marketplace Registration

The `.claude-plugin/marketplace.json` file registers all plugins:

```json
{
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./plugins/plugin-name",
      "description": "...",
      "version": "1.0.0",
      "author": {...},
      "keywords": [...],
      "category": "development|productivity",
      "strict": false
    }
  ]
}
```

Note: Each plugin is registered with its source directory. The plugin system automatically discovers agents and commands within the plugin's subdirectories.

## Existing Plugins

### git-commit-generator

**Purpose**: Analyzes git diffs and generates Conventional Commits compliant commit messages

**Commands**:
- `/git-commit-generator:commit_message [diff]` - Analyze changes and generate structured commit groups
- `/git-commit-generator:commit <id> <type>(<scope>): <msg>` - Commit specific file group
- `/git-commit-generator:commit_all` - Automatically commit all remaining changes in dependency-aware order

**Key Behaviors**:
- NEVER uses `git diff --staged` (always analyzes all uncommitted changes)
- Classifies changes by package/module, not by commit type
- Groups related files logically with unique identifiers
- Prioritizes shared/common modules first (dependency ordering)
- Enforces Conventional Commits format: feat, fix, docs, style, refactor, perf, test, chore

### knowledge-researcher

**Purpose**: Recursively searches markdown documentation by following internal links

**Commands**:
- `/knowledge-researcher:research <topic> [in entry-file]` - Search documentation recursively

**Key Behaviors**:
- Starts from `.llms-txt`, `docs/`, or specified entry file
- Follows markdown links recursively (max 10 levels)
- Prevents circular references and infinite loops
- Ranks results by relevance (High/Medium/Low)
- Extracts context with link chain tracking
- READ-ONLY operation (never modifies files)
- Asks permission before expanding search scope

### d2c-generator

**Purpose**: Transforms UI design images into production-ready React components using Semi Design

**Commands**:
- `/d2c-generator:ocr <image-path> [--ref ui.json] [--prd prd.md] [--style true|false]` - Analyze UI image and generate component tree JSON
- `/d2c-generator:implComponents <ui-json-path> <output-dir-or-file>` - Implement React components from UI JSON

**Key Behaviors**:
- Generates UI Component Tree JSON following strict schema
- Identifies layout types and Semi Design components
- Uses stable `identity` object for component classification
- Supports reference UI JSON for consistent component recognition
- Generates BEM-style SCSS with theme support (light/dark)
- Default language: Chinese/中文
- ALWAYS updates component.ui.json after code generation

### agent-creator

**Purpose**: Meta-agent that guides creation of new agents for this repository

**Location**: `.claude/agents/agent-creator.md` (internal agent, not a plugin)

**Key Behaviors**:
- Gathers requirements interactively
- Analyzes existing plugin structures
- Generates agent files with proper YAML frontmatter
- Updates marketplace.json registration
- Validates consistency across all references
- Follows patterns from existing plugins (especially git-commit-generator)

## Development Workflow

### No Build System

This repository contains **no build, test, or compilation steps**. It consists only of:
- Markdown files (agents and commands)
- JSON configuration (marketplace.json)

### Creating New Agents

**Automated Approach (Recommended)**:
Use the agent-creator meta-agent:
```
Can you help me create a new agent for [specific purpose]?
```

**Manual Approach**:
1. Create plugin directory: `mkdir -p plugins/{plugin-name}/{agents,commands}`
2. Create agent file: `plugins/{plugin-name}/agents/{agent-name}.md`
3. Add YAML frontmatter with `name`, `description` (with `<example>` tags), and `model`
4. Write agent system prompt in markdown body
5. Create command files if needed: `plugins/{plugin-name}/commands/{command}.md`
6. Update `.claude-plugin/marketplace.json`:
   - Add new plugin entry with `source: "./plugins/{plugin-name}"`
   - Set appropriate `category`, `keywords`, and `description`

### Testing Agents

Test agents through Claude Code using the Task tool:
```
Can you use the {agent-name} agent to [task]?
```

Test slash commands directly:
```
/{plugin-name}:{command-name} [args]
```

### Consistency Requirements

When modifying or creating agents:
- Agent names must be kebab-case and consistent across filename, frontmatter `name`, and marketplace.json
- Include concrete usage examples in `<example>` tags within the `description` field
- Use `model: inherit` to respect Claude Code's model selection
- Validate YAML frontmatter syntax
- Validate marketplace.json syntax after modifications
- Keep agent descriptions focused on when/how to trigger the agent
- Command `argument-hint` should show expected argument pattern

## Important Conventions

### Git Commit Generation

- The git-commit-generator NEVER uses `--staged` flag
- Changes are classified by **module/package**, not by commit type
- Shared/common modules are committed first (dependency ordering)
- Each change group gets a unique kebab-case identifier
- Commit messages strictly follow Conventional Commits specification

### Documentation Search

- knowledge-researcher is strictly READ-ONLY
- Always asks permission before expanding search scope
- Follows markdown links with circular reference prevention
- Maximum recursion depth: 10 levels
- Results include relevance ranking and link chain tracking

### UI Component Generation

- d2c-generator specializes in Semi Design React components
- Uses stable `identity` object for component consistency
- Generates BEM-style SCSS with theme variable support
- ALWAYS updates component.ui.json after code changes
- Default language is Chinese for this plugin

### Agent Creation

- Use agent-creator for systematic agent development
- Follow the plugin structure: `plugins/{name}/{agents,commands}/`
- Include usage examples in `<example>` tags in agent descriptions
- Register plugins in marketplace.json with proper metadata
