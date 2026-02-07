# plugin-security-checker

Security scanner for Claude Code plugins. Analyzes hooks, MCP servers, scripts, and prompts for potential security risks before installation.

## Features

- **Hook Analysis**: Detects dangerous shell commands, data exfiltration, reverse shells, and obfuscated code in hook configurations
- **MCP Server Analysis**: Identifies suspicious external connections, credential exposure, and unverified dependencies
- **Script Analysis**: Scans executable files for network operations, file system access outside plugin scope, and code obfuscation
- **Prompt Analysis**: Checks skills and agents for prompt injection, excessive permissions, and behavioral manipulation
- **Manifest Analysis**: Verifies path safety, structural integrity, and metadata completeness

## Installation

```shell
/plugin install plugin-security-checker@claude-marketplace
```

## Usage

### Check a GitHub repository

```shell
/plugin-security-checker:security-check owner/repo
```

### Check a local plugin directory

```shell
/plugin-security-checker:security-check ./path/to/plugin
```

## Risk Levels

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Immediate danger - do not install |
| **HIGH** | Significant risk - install with extreme caution |
| **MEDIUM** | Potential concern - review findings before installing |
| **LOW** | Minor issues - generally safe |
| **CLEAN** | No security concerns detected |

## Prerequisites

- `gh` CLI (for GitHub repository scanning) or `git`

## Components

| Type | Name | Purpose |
|------|------|---------|
| Command | `security-check` | Main entry point - accepts repo or path |
| Agent | `security-scanner` | Deep analysis engine for plugin components |
| Skill | `plugin-security` | Security patterns knowledge base |
