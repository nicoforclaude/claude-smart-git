# Claude Smart Git

Intelligent git workflow tools for Claude Code with smart commit analysis and recommendations.

## Overview

`claude-smart-git` provides intelligent git workflow automation for Claude Code, featuring smart commit analysis, readiness assessment, and automated commit message generation.

## What's Included

### Skills (1)

#### **git-changes-analyzer**
- Analyzes git changes with readiness indicators (✅/🚧/⚠️/🗑️)
- Recommends atomic commit strategies
- Auto-triggers when analyzing or creating commits
- Aliases: `git`, `commits`

### Agents (1)

#### **git-changes-analyzer-agent**
- Autonomous analysis of git changes
- Multi-commit workflow recommendations
- Working tree review and startup checks
- Provides structured commit recommendations

### Commands (5)

#### **/commit** (frequently used)
- Smart commit workflow with automated analysis
- Linting integration
- AI-generated commit messages
- Pre-commit hook handling

#### **/git** (menu)
- Interactive menu for git commands
- Quick access to status, catchup, and startup

#### **/git:status**
- Show current branch
- Check for pending changes
- Quick status overview

#### **/git:catchup**
- Merge dev-preview into main branch
- Automated branch synchronization

#### **/git:startup**
- Quick git status check at session start
- Ensures clean working tree awareness

## Installation

### Prerequisites
- Claude Code CLI with plugin support
- Git repository initialized

### Plugin Installation

1. **Clone the marketplace**:
   ```bash
   git clone https://github.com/nicoforclaude/claude-smart-git.git
   cd claude-smart-git
   ```

2. **Install the plugin**:

   The marketplace contains the `smart-git` plugin. Claude Code will automatically discover plugins in the `.claude-plugin` structure.

   **Option A: Link the entire marketplace** (recommended for development):
   ```bash
   # From your workspace
   ln -s /path/to/claude-smart-git ~/.claude/marketplaces/claude-smart-git
   ```

   **Option B: Copy plugin directly**:
   ```bash
   cp -r plugins/smart-git ~/.claude/plugins/
   ```

3. **Restart Claude Code** to load the plugin

### Upgrading from v0.1.0

If you previously installed components directly, remove them first:
```bash
rm -rf ~/.claude/skills/git-changes-analyzer
rm ~/.claude/agents/git-changes-analyzer-agent.md
rm ~/.claude/commands/commit.md
rm ~/.claude/commands/git.md
rm -rf ~/.claude/commands/git/
```

Then follow the plugin installation steps above.

## Usage

### Automatic Activation

The `git-changes-analyzer` skill activates automatically:
- When analyzing commits
- Before creating commits
- During git workflow operations

### Manual Activation

Use the `Skill` tool to manually activate:

```
Skill(skill: "git-changes-analyzer")
```

### Using Commands

```bash
# Commit with smart analysis (most frequently used)
/commit

# Open git commands menu
/git

# Or use specific git commands directly
/git:status
/git:catchup
/git:startup
```

### Using Agents

Launch the agent with the `Task` tool:

```
Task(subagent_type: "git-changes-analyzer-agent", prompt: "Analyze current changes")
```

## Features

### Smart Commit Analysis

The git-changes-analyzer provides:
- **✅ Ready**: Changes are coherent and ready to commit
- **🚧 In Progress**: Work is incomplete or mixed
- **⚠️ Needs Review**: Suspicious patterns detected
- **🗑️ Cleanup**: Temporary or debug code present

### Atomic Commit Recommendations

Analyzes working tree and suggests logical commit groupings:
- Separate features from fixes
- Isolate refactoring from functionality
- Group related changes together

### Intelligent Commit Messages

Automatically generates:
- Conventional commit format
- Contextual descriptions
- Focused on "why" not just "what"

## Philosophy

1. **Atomic commits** - One logical change per commit
2. **Smart analysis** - AI-powered readiness assessment
3. **Clean history** - Meaningful commit messages
4. **Workflow automation** - Reduce manual git overhead

## Support

For issues or feature requests, please open an issue on [GitHub](https://github.com/nicoforclaude/claude-smart-git/issues).

## License

MIT License - See [LICENSE](LICENSE) file for details.

## Version History

- **0.2.0** - Restructured to plugin marketplace architecture
  - Migrated to `.claude-plugin` structure
  - Added `smart-git` plugin containing all components
  - Updated installation instructions for plugin approach
  - **Breaking change**: Requires plugin installation method
- **0.1.0** - Initial release with git-changes-analyzer skill, agent, and 5 commands
