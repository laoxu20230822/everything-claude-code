# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Everything Claude Code (ECC) is a Claude Code plugin providing production-ready agents, skills, hooks, commands, and rules evolved from 10+ months of intensive use. This is the repository for the plugin itself - you are working on the plugin that enhances Claude Code, not a user project.

### Key Architecture

```
everything-claude-code/
├── .claude-plugin/      # Plugin manifest and marketplace config
├── agents/              # Subagent definitions (Markdown with frontmatter)
├── commands/            # Slash commands invoked by user
├── skills/              # Knowledge modules loaded by context
├── rules/               # Always-follow guidelines (common/ + language-specific)
├── hooks/               # Hook configurations (hooks.json + Node.js implementations)
├── scripts/             # Cross-platform Node.js hook implementations
│   ├── lib/            # Shared utilities (utils.js, package-manager.js)
│   └── hooks/          # Individual hook scripts
├── tests/               # Node.js test suite
└── .opencode/          # OpenCode-specific configuration
```

### Critical: No "hooks" in plugin.json

**IMPORTANT REGRESSION:** Do NOT add a `"hooks"` field to `.claude-plugin/plugin.json`. Claude Code v2.1+ automatically loads `hooks/hooks.json` by convention. Explicitly declaring it causes a duplicate detection error. This is enforced by a regression test.

## Development Commands

### Testing

```bash
# Run entire test suite
node tests/run-all.js

# Run individual test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

### Code Quality

```bash
# Lint (uses ESLint from devDependencies)
npm run lint  # or: npx eslint .

# Format Markdown
npx markdownlint '**/*.md'
```

### Package Manager Detection

The plugin supports automatic package manager detection (npm, pnpm, yarn, bun) via:
1. Environment variable `CLAUDE_PACKAGE_MANAGER`
2. Project config `.claude/package-manager.json`
3. `package.json`'s `packageManager` field
4. Lock file detection (package-lock.json, yarn.lock, pnpm-lock.yaml, bun.lockb)
5. Global config fallback

To test detection:
```bash
node scripts/setup-package-manager.js --detect
```

## Plugin Architecture

### Agents

Located in `agents/*.md` - each is a Markdown file with YAML frontmatter:

```markdown
---
name: agent-name
description: When to invoke this agent
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: sonnet
---

You are a specialist for X...
```

**Model selection guidelines:**
- `haiku` - Simple, high-frequency tasks
- `sonnet` - Coding and orchestration (default)
- `opus` - Complex reasoning and architecture

### Commands

Located in `commands/*.md` - invoked via `/command-name`:

```markdown
---
description: Brief description for /help
---

# Command Name

Usage, workflow, and output format.
```

### Skills

Located in `skills/*/SKILL.md` - knowledge modules with frontmatter:

```markdown
---
name: skill-name
description: When this skill applies
---

# Skill Title

Content is loaded into context when relevant.
```

### Hooks

**Configuration:** `hooks/hooks.json` defines matchers and actions.
**Implementation:** `scripts/hooks/*.js` contains Node.js scripts (cross-platform).

**Hook Types:**
| Type | Trigger | Common Use |
|------|----------|--------------|
| `PreToolUse` | Before tool execution | Validation, warnings, blocking |
| `PostToolUse` | After tool execution | Formatting, type checking, notifications |
| `SessionStart` | New session | Load context, detect package manager |
| `Stop` | Response complete | Final checks, audits |
| `SessionEnd` | Session ends | Save state, extract patterns |
| `PreCompact` | Before context compaction | Persist important state |

**Matcher Syntax:**
```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|js)$\"",
  "hooks": [{"type": "command", "command": "node script.js"}]
}
```

**Cross-Platform Scripts:** All hook scripts use Node.js (not bash) for Windows/macOS/Linux compatibility. Use utilities from `scripts/lib/utils.js`.

### Rules

Organized into `common/` (language-agnostic) plus `typescript/`, `python/`, `golang/`:

```
rules/
├── common/           # Always install these
│   ├── coding-style.md    # Immutability, file organization
│   ├── git-workflow.md    # Commit format
│   ├── testing.md         # 80% coverage, TDD
│   └── security.md        # No hardcoded secrets
├── typescript/       # TS/JS specific
├── python/           # Python specific
└── golang/           # Go specific
```

Rules are NOT distributed via the plugin (Claude Code limitation). Users install manually to `~/.claude/rules/`.

## Key Files

### Plugin Manifest
- `.claude-plugin/plugin.json` - Plugin metadata, component paths
- `.claude-plugin/marketplace.json` - Marketplace catalog

### Core Utilities
- `scripts/lib/utils.js` - Cross-platform file system, git, date utilities
- `scripts/lib/package-manager.js` - Package manager detection
- `scripts/lib/session-aliases.js` - Session alias management
- `scripts/lib/session-manager.js` - Session persistence

### OpenCode Support
- `.opencode/opencode.json` - OpenCode configuration
- `.opencode/README.md` - OpenCode-specific docs
- `.opencode/instructions/INSTRUCTIONS.md` - Consolidated rules for OpenCode

## Contribution Patterns

### Adding a New Hook
1. Add matcher to `hooks/hooks.json`
2. Implement in `scripts/hooks/name.js` (Node.js, cross-platform)
3. Add test to `tests/hooks/`
4. Document in `.opencode/instructions/INSTRUCTIONS.md` if user-facing

### Adding a New Agent/Command/Skill
1. Create `.md` file in appropriate directory
2. Follow template structure from `CONTRIBUTING.md`
3. Add to `.claude-plugin/plugin.json` (for agents/commands)
4. Test with Claude Code locally

## Important Constraints

1. **No hardcoded secrets** - Use environment variables
2. **Cross-platform** - All scripts must work on Windows, macOS, Linux
3. **No "hooks" in plugin.json** - See "Critical" section above
4. **Node.js for hooks** - Not shell scripts (use `scripts/lib/utils.js`)
5. **Rules manual install** - Documented in README, not auto-distributed

## Versioning

Plugin version in `.claude-plugin/plugin.json` should be updated on releases.
Current: 1.4.1

## Testing Your Changes

When modifying the plugin itself:
1. Copy to `~/.claude/plugins/everything-claude-code/`
2. Restart Claude Code
3. Test agents/commands/hooks in a real session
4. Run `node tests/run-all.js` before committing
