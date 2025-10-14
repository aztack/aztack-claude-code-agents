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
├── .claude-plugin/
│   └── marketplace.json          # Plugin marketplace configuration
├── agents/
│   └── git-commit-generator.md
├── CLAUDE.md                      # Development guidance for Claude Code
└── README.md
```

## Development

### Adding New Agents

1. Create a new directory under `agents/` with kebab-case naming:
   ```bash
   mkdir -p agents/my-new-agent
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

3. Update `.claude-plugin/marketplace.json` to register the agent:
   ```json
   {
     "agents": [
       "./agents/my-new-agent.md"
     ]
   }
   ```

### Testing Agents

Test agents directly in Claude Code using the Task tool:
```bash
# In Claude Code conversation
Can you use the git-commit-generator agent to analyze my changes?
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

### v1.0.0 (Initial Release)

- Added git-commit-generator agent with full Conventional Commits support
- Established repository structure and development guidelines
- Created marketplace configuration for distribution

---

**Note:** These agents are designed to work with Claude Code CLI. For more information about Claude Code, visit [claude.com/claude-code](https://claude.com/claude-code).
