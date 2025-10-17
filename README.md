# Aztack's Claude Code Agents

A collection of specialized AI agents for [Claude Code](https://claude.com/claude-code) designed to enhance your development workflows with intelligent automation and best practices.

## Overview

This repository contains production-ready Claude Code agents that integrate seamlessly into your development environment. Each agent is designed to handle specific workflows with professional-grade quality and adherence to industry standards.

## Available Agents

### git-commit-generator

A professional Git workflow agent that analyzes code changes and generates [Conventional Commits](https://www.conventionalcommits.org/) compliant commit messages with intelligent change classification.

**Key Features:**
- Analyzes `git diff` output and classifies changes by package/module and functionality
- Generates structured, production-ready commit messages following Conventional Commits specification
- Supports granular control over what to commit with unique identifiers for each change group
- Automatically groups related changes logically (by module, not by commit type)
- Enforces strict commit types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

**Usage:**

```bash
# Analyze changes and generate commit messages
/commit_message

# Commit a specific change group with optional corrections
/commit <group-name> <type>(<scope>): <message>

# Commit all remaining changes automatically
/commit_all
```

**Example Workflow:**

```bash
# 1. Analyze your changes
> /commit_message

# Agent analyzes diff and outputs:
# - Unique names for each logical change group
# - Classification summary organized by package/module
# - Ready-to-use commit messages

# 2. Commit specific groups
> /commit create-shared-module feat(shared): create shared module for types and utils
> /commit add-pagination feat(api): add pagination to marketplace endpoints

# 3. Commit all remaining changes at once
> /commit_all
```

### knowledge-researcher

A specialized agent for recursively searching and extracting knowledge from markdown documentation by following internal links. Perfect for navigating complex documentation structures and finding specific information across interconnected files.

**Key Features:**
- Intelligent entry point detection (`.llms-txt`, `docs/`, `README.md`)
- Recursive link traversal with circular reference prevention
- Relevance-based result ranking (High/Medium/Low)
- Context extraction with link chain tracking
- Read-only operation ensuring safe documentation exploration
- Fallback strategies with user confirmation for expanded searches

**Usage:**

```bash
# Search with specific entry file
/research authentication API in docs/api/readme.md

# Search from default documentation root
/research database migrations

# Search with quoted phrases
/research "error handling" in docs/guide.md
```

**Example Workflow:**

```bash
# 1. Search for specific topic
> /research JWT token validation in docs/security/auth.md

# Agent recursively follows markdown links and returns:
# - Relevance-ranked findings
# - Source paths with link chains
# - Search statistics (files searched, links followed)

# 2. If nothing found, agent asks for permission to expand search
# Agent: "No results found. Search from root documentation?"
> Yes

# 3. Agent expands search to .llms-txt and presents comprehensive results
```

### agent-creator

A meta-agent that guides you through creating new agents for this repository. Streamlines the agent development workflow with automated scaffolding and quality checks.

**Key Features:**
- Interactive requirements gathering
- Automated file generation with proper YAML frontmatter
- Marketplace.json registration
- Consistency validation across all references
- Follows established patterns from existing agents

**Usage:**

```bash
# Launch the agent creator
Can you help me create a new agent for [specific purpose]?
```

## Installation

### From GitHub

```bash
# Clone the repository
git clone https://github.com/aztack/claude-code-agents.git

# Link to Claude Code agents directory (adjust path as needed)
ln -s $(pwd)/aztack-claude-code-agents ~/.claude/agents/aztack-claude-code-agents
```

### From Claude Code Marketplace

*(Coming soon - once published to the official marketplace)*

Search for "aztack-claude-code-agents" or "git-commit-generator" in the Claude Code marketplace and click install.

## Repository Structure

```
aztack-claude-code-agents/
├── .claude/
│   └── agents/
│       └── agent-creator.md      # Meta-agent for creating new agents
├── .claude-plugin/
│   └── marketplace.json          # Plugin marketplace configuration
├── plugins/
│   ├── git-commit-generator/
│   │   ├── agents/
│   │   │   └── git-commit-generator.md
│   │   └── commands/
│   │       ├── commit.md
│   │       ├── commit_all.md
│   │       └── commit_message.md
│   ├── knowledge-researcher/
│   │   ├── agents/
│   │   │   └── knowledge-researcher.md
│   │   └── commands/
│   │       └── research.md
│   └── d2c-generator/
│       └── ...
├── CLAUDE.md                      # Development guidance for Claude Code
└── README.md
```

## Development

### Adding New Agents

**Automated Approach (Recommended):**

Use the `agent-creator` meta-agent to streamline the process:

```bash
# In Claude Code conversation
Can you help me create a new agent for [specific purpose]?
```

The agent-creator will:
- Gather requirements interactively
- Generate proper file structure with YAML frontmatter
- Register the agent in marketplace.json
- Validate consistency across all references

**Manual Approach:**

1. Create a new directory under `plugins/` with kebab-case naming:
   ```bash
   mkdir -p plugins/my-new-agent/{agents,commands}
   ```

2. Create the agent definition file with YAML frontmatter:
   ```markdown
   ---
   name: my-new-agent
   description: |
     Agent description with usage examples...
   model: inherit
   ---

   [Agent system prompt and instructions here]
   ```

3. Create command files if needed (optional):
   ```bash
   # plugins/my-new-agent/commands/my_command.md
   ---
   description: Command description
   argument-hint: "<args>"
   ---

   [Command implementation]
   ```

4. Update `.claude-plugin/marketplace.json` to register the plugin:
   ```json
   {
     "plugins": [
       {
         "name": "my-new-agent",
         "source": "./plugins/my-new-agent",
         "agents": ["./agents/my-new-agent.md"],
         "commands": ["./commands/my_command.md"]
       }
     ]
   }
   ```

### Testing Agents

Test agents directly in Claude Code using the Task tool:
```bash
# In Claude Code conversation
Can you use the git-commit-generator agent to analyze my changes?

# Or test slash commands
/research "authentication" in docs/
```

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Follow the existing agent structure and naming conventions
2. Ensure YAML frontmatter is valid
3. Include comprehensive examples in the agent description
4. Test agents thoroughly before submitting
5. Update marketplace.json with appropriate metadata

## License

MIT License - see [LICENSE](LICENSE) file for details

## Author

**Wang Weihua** (aztack)
- GitHub: [@aztack](https://github.com/aztack)
- Email: aztack@163.com

## Links

- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [Conventional Commits Specification](https://www.conventionalcommits.org/)
- [Repository](https://github.com/aztack/claude-code-agents)
- [Issues](https://github.com/aztack/claude-code-agents/issues)

## Changelog

### v1.1.0 (Latest)

- Added **knowledge-researcher** agent for recursive documentation search
  - Intelligent entry point detection and link traversal
  - Relevance-based ranking with context extraction
  - Read-only safe operation with fallback strategies
- Added **agent-creator** meta-agent for streamlined agent development
  - Interactive requirements gathering
  - Automated scaffolding and validation
  - Marketplace registration and consistency checks
- Reorganized repository structure with `plugins/` directory
- Enhanced development workflow documentation

### v1.0.0 (Initial Release)

- Added **git-commit-generator** agent with full Conventional Commits support
  - `/commit_message` - Analyze and generate commit messages
  - `/commit` - Commit specific change groups
  - `/commit_all` - Bulk commit all remaining changes
- Established repository structure and development guidelines
- Created marketplace configuration for distribution

---

**Note:** These agents are designed to work with Claude Code CLI. For more information about Claude Code, visit [claude.com/claude-code](https://claude.com/claude-code).
