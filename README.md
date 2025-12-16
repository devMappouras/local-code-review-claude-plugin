# Local Code Review Plugin

Local code review for uncommitted changes in .NET and Angular projects with build verification and optional unit test execution.

## Overview

This plugin provides automated code review for local uncommitted changes (staged and unstaged) in your .NET and Angular projects. It launches multiple specialized agents to review code from different perspectives, builds the project to catch compile errors, and optionally runs unit tests.

**Key Features:**
- Reviews all uncommitted changes (`git diff HEAD`)
- Auto-detects `.sln` files for .NET projects
- Auto-detects Angular projects (`angular.json`)
- Builds .NET (`dotnet build`) and Angular (`ng build`) to catch errors
- Auto-detects and runs unit tests (NUnit/XUnit)
- Confidence-based scoring (threshold: 80) to filter false positives
- Specialized for .NET/Angular best practices
- **Read-only**: Never auto-fixes - only reports issues

## Commands

### `/local-code-review`

Reviews uncommitted changes and builds the project.

**What it does:**
1. Gets uncommitted changes via `git diff HEAD`
2. Auto-detects `.sln` file in current directory or parents
3. Auto-detects Angular project (`angular.json`)
4. Launches parallel review agents:
   - CLAUDE.md compliance
   - Bug detection
   - .NET/Angular best practices
   - Security review
5. Builds .NET project (`dotnet build`)
6. Builds Angular project (`ng build`) if detected
7. Reports issues with confidence scoring (threshold: 80)

**Usage:**
```bash
/local-code-review
```

### `/local-code-review-tests`

Reviews uncommitted changes, builds the project, and runs unit tests.

**What it does:**
- Everything from `/local-code-review`
- Auto-detects test projects (NUnit/XUnit references)
- Runs unit tests (`dotnet test`)
- Reports test results (pass/fail)

**Usage:**
```bash
/local-code-review-tests
```

## Agent

### `local-code-reviewer`

Specialized agent for .NET/Angular code review that triggers proactively when code review scenarios are detected.

**Capabilities:**
- Reviews code for bugs, security issues, and best practices
- Specialized for .NET 8/10 and Angular
- Never auto-fixes - only reports issues
- Uses confidence scoring to filter false positives

## Skill

### `dotnet-angular-review`

Embedded knowledge for .NET and Angular code review best practices.

**Covers:**
- Clean Architecture patterns
- SOLID principles
- Async/await best practices
- Exception handling patterns
- Angular component patterns
- RxJS best practices
- Security (OWASP)
- Performance considerations

## Requirements

- Git repository with uncommitted changes
- .NET SDK (for `dotnet build` and `dotnet test`)
- Node.js and Angular CLI (for `ng build` if Angular project)
- CLAUDE.md files (optional but recommended)

## Installation

### Option 1: Install from Marketplace (Recommended)

1. Add the marketplace to Claude Code:
```bash
/plugin marketplace add devMappouras/local-code-review-claude-plugin
```

2. Install the plugin:
```bash
/plugin install local-code-review@devMappouras-plugins
```

### Option 2: Clone directly

1. Clone the repository to your Claude Code plugins folder:
```bash
# Windows
git clone https://github.com/devMappouras/local-code-review-claude-plugin.git "%USERPROFILE%\.claude\plugins\local-code-review"

# macOS/Linux
git clone https://github.com/devMappouras/local-code-review-claude-plugin.git ~/.claude/plugins/local-code-review
```

2. Restart Claude Code to load the plugin.

### Verify Installation

After installation, verify the plugin is loaded:
```bash
/help
```

You should see `/local-code-review` and `/local-code-review-tests` in the available commands.

## Best Practices

- Maintain clear CLAUDE.md files for better compliance checking
- Run `/local-code-review` before committing changes
- Use `/local-code-review-tests` for comprehensive review with test verification
- Trust the 80+ confidence threshold - false positives are filtered
- Request fixes explicitly after reviewing the report

## Author

Christos Mappouras

## Version

1.0.0
