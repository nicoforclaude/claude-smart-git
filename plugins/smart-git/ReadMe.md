# Smart Git Plugin

Intelligent git workflow automation with smart commit analysis.

## Components

### Skill: git-changes-analyzer
Analyzes git changes with readiness indicators (✅/🚧/⚠️/🗑️) and recommends atomic commit strategies.

### Agent: git-changes-analyzer-agent
Autonomous analysis of working tree with multi-commit workflow recommendations.

### Commands

- **/commit** - Smart commit workflow with linting and AI-generated messages
- **/git** - Menu for git commands
- **/git:status** - Show branch and pending changes
- **/git:catchup** - Merge dev-preview to main
- **/git:startup** - Quick status check

## Usage

The plugin activates automatically during git operations. For manual activation:

```
Skill(skill: "git-changes-analyzer")
```

For commit workflow:
```
/commit
```
