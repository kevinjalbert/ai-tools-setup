# AI Configuration Setup

The goal of this project is to:

1. Simplify the management of AI configurations (rules, commands, and MCP servers) for AI tools
2. Allow developers to maintain a single source of truth for their AI configuration
3. Provide an easy setup process for both global and project-specific settings

## How It Works

You have a single configuration directory that contains your rules, commands, and MCP server configuration. A single script will propagate your configuration to the correct locations for both AI tools.

Currently, the script supports the following AI tools:

| AI Tool                                                  | Configurations                     | MCP Servers | Project Scope | Global Scope |
|----------------------------------------------------------|-----------------------------------|------------|----------------|--------------|
| [Claude Code](https://github.com/anthropics/claude-code) | [Link](#claude-code-configuration) | ✅          | ❌            | ✅            |
| [Cursor](https://www.cursor.com)                         | [Link](#cursor-configuration)      | ✅          | ✅ (symlinks) | ❌            |

## How to Use

1. [Setup the configuration directory](#configuration-directory-structure) and add your rules, commands, and MCP server configuration.
2. Run `scripts/setup --help` to see all the options. You can control what configurations are setup.
3. Run `scripts/setup` with appropriate flags to propagate your AI configurations to the correct locations.

If needed, you can reset your configuration by running `scripts/reset`. This will remove all the symlinks and removes globally configured files. This shouldn't be needed as the setup script is idempotent.

### Configuration Directory Structure

Your configuration directory (`~/ai_configs` by default) should be structured to hold your rules, commands, and MCP server configuration. The directory can be overridden by setting the `AI_CONFIGS_DIR` environment variable when running the `setup` script.

```bash
~/ai_configs/
├── rules/                   # Directory containing rule files (.mdc)
│   ├── rule1.mdc
│   └── rule2.mdc
├── commands/                # Directory containing command files (.mdc)
│   ├── command1.mdc
│   └── command2.mdc
└── mcp-servers-config.json  # Configuration for MCP servers
```

It is important to note that the rules and commands are written as `.mdc` files. These files are used by the `scripts/setup` script to create the appropriate configuration files for the AI tools.

#### Example .mdc file

```markdown
---
description: JavaScript coding standards for the project
globs: ["*.js", "*.jsx", "*.ts", "*.tsx"]
alwaysApply: false
---

You are a helpful AI assistant for JavaScript programming tasks. Always focus on readability and maintainability.
```

Even though some AI tools might not use `.mdc` files, we just keep the content (for context) and copy them as `.md` files.

#### Example `mcp-servers-config.json`

```json
{
  "example-custom-mcp": {
      "type": "stdio",
      "command": "node",
      "args": ["path/to/your/custom-mcp-server.js"],
      "env": {
        "DEBUG": "false"
      }
  }
}
```

## AI Tool Specifics

### Claude Code Configuration

- **Global Rules**: Rules (.mdc files) are concatenated into `~/.claude/CLAUDE.md`
- **Global Commands**: Commands (.mdc files) are copied to `~/.claude/commands/` with `.md` extension
- **MCP Servers**: Server configurations (defined as `mcp-servers-config.json`) are added to `~/.claude.json`

### Cursor Configuration

- **Project-Based Rules**: Rules directory is symlinked to `.cursor/rules/local/rules` in your project (based on `pwd`)
- **Project-Based Commands**: Commands directory is symlinked to `.cursor/rules/local/commands` in your project (based on `pwd`)
- **MCP Servers**: Server configurations (defined as `mcp-servers-config.json`) is copied to `~/.cursor/mcp.json`

#### Limitations of Cursor

Cursor currently only supports project-based rules, not global rules. This means:

- Rules and commands must be set up in each project directory where you want to use them.
- This tool creates symlinks in the project directory to the central rules/commands.
- Changes to your configuration files are automatically reflected in your projects due to the symlinks.

You might want to globally ignore the added local Cursor rules: `**/.cursor/rules/local/**` in your version control system.

## Idempotent Updates

`scripts/setup` is designed to be idempotent, meaning you can run it multiple times without issues. This is useful when:

- You've made changes to your rules or commands in your configuration directory
- You want to apply your (project-based) configuration to a new project directory
- You need to update MCP server configurations

For Cursor's project-based configurations, changes to your rule and command files are automatically reflected in your projects because they are symlinked. You don't need to rerun the setup script for these changes to take effect.
