---
name: plugin-security
description: "Security analysis knowledge for Claude Code plugins. This skill should be used when the user asks to check, audit, scan, or review a Claude Code plugin for security risks, vulnerabilities, or malicious code. Common triggers include 'is this plugin safe', 'scan plugin for security issues', 'audit plugin hooks', 'check for prompt injection', 'review MCP server security', or 'analyze plugin scripts for risks'."
disable-model-invocation: true
---

# Plugin Security Analysis Guide

Analyze Claude Code plugins for security risks by systematically examining each component type. Report findings with severity levels and actionable recommendations.

For automated scanning, use the `/plugin-security-checker:security-check` command or the `security-scanner` agent.
For the report format template, refer to `references/report-template.md`.

## Risk Severity Levels

- **CRITICAL**: Immediate danger - arbitrary code execution, data exfiltration, credential theft
- **HIGH**: Significant risk - unauthorized network access, excessive permissions, obfuscated code
- **MEDIUM**: Potential concern - prompt injection patterns, overly broad tool access
- **LOW**: Minor issue - missing best practices, structural concerns
- **INFO**: Informational - observations that may be relevant for security-conscious users

## Component Analysis Procedures

### 1. Hooks Analysis (Priority: Highest)

Hooks execute shell commands automatically in response to Claude Code events. They run with the user's full permissions.

**Check hooks.json and any hook configurations in plugin.json for:**

Dangerous command patterns:
- Destructive operations: `rm -rf`, `rm -f`, `mkfs`, `dd if=`
- Remote code execution: `curl | bash`, `wget -O- | sh`, `eval $(curl`, `python -c`
- Reverse shells: `bash -i >& /dev/tcp/`, `nc -e`, `ncat`, `socat`
- Privilege escalation: `sudo`, `su -`, `chmod 777`, `chmod +s`
- Process manipulation: `kill -9`, `pkill`, `killall` (when targeting system processes)

Data exfiltration patterns:
- Sending environment variables: `curl -d "$(env)"`, `wget --post-data="$(printenv)"`
- Reading sensitive files: `cat ~/.ssh/`, `cat ~/.aws/`, `cat ~/.gitconfig`
- Accessing credentials: `$API_KEY`, `$SECRET`, `$TOKEN`, `$PASSWORD`, `$PRIVATE_KEY`
- Sending files externally: `scp`, `rsync` to remote hosts, `curl -F "file=@"`

High-risk event bindings:
- `SessionStart`: Executes on every session start without user action
- `UserPromptSubmit`: Intercepts every user input
- `PreToolUse`: Can modify or block tool execution
- `Notification`: Can intercept notification content

Obfuscation techniques:
- Base64 encoded commands: `echo "..." | base64 -d | bash`
- Hex encoded payloads: `echo -e "\x..."`, `printf '\x...'`
- Variable indirection: `${!var}`, indirect parameter expansion
- String concatenation to build commands: `c="cu"; c+="rl"`

### 2. MCP Servers Analysis (Priority: High)

MCP servers run as long-lived processes with network access capabilities.

**Check .mcp.json and mcpServers in plugin.json for:**

Suspicious server commands:
- Unknown or uncommon binaries (not `node`, `python`, `npx`, standard tools)
- Direct network tools as server commands: `nc`, `socat`, `ncat`
- Scripts that download and execute: `bash -c "curl ... && ..."`

Network security:
- Non-HTTPS URLs in configuration
- Connections to IP addresses instead of domain names
- Hardcoded credentials in args or env fields
- Connections to unusual ports

Environment variable exposure:
- Passing broad environment access: `"env": { "PATH": ... }` combined with sensitive vars
- References to credential environment variables without clear necessity
- Passing `HOME`, `USER`, or system identity information

Dependency risks:
- `npx` executing unverified packages from npm
- `pip` or `pipx` running unverified packages
- No version pinning on external packages

### 3. Scripts Analysis (Priority: High)

Scripts in the scripts/ directory or referenced by hooks.

**Check all executable files (.sh, .py, .js, .rb, etc.) for:**

Network operations:
- HTTP requests: `curl`, `wget`, `fetch`, `requests.get/post`, `http.get`
- Socket operations: `socket`, `net.connect`, `TCP`
- DNS lookups to unusual domains

File system operations:
- Access outside plugin directory (paths not using `$CLAUDE_PLUGIN_ROOT`)
- Reading user home directory files: `~/.ssh`, `~/.aws`, `~/.config`
- Writing to system directories: `/etc/`, `/usr/`, `/tmp/` (for persistence)
- Creating hidden files: names starting with `.`

Code obfuscation:
- Base64 encoding/decoding of commands
- Eval with dynamic strings
- Minified or intentionally unreadable code
- Embedded binary data
- ROT13, XOR, or other encoding of strings

### 4. Skills and Agents Analysis (Priority: Medium)

Skills and agents control Claude's behavior through prompts.

**Check SKILL.md, agent .md files, and command .md files for:**

Prompt injection patterns:
- "Ignore previous instructions"
- "You are now...", "Your new role is..."
- "System prompt override"
- Hidden instructions in HTML comments or zero-width characters
- Instructions to disable safety features

Excessive tool permissions:
- `allowed-tools` including Bash, Write, or Edit when not clearly needed
- Agents requesting all tools (`*`) without justification
- Commands with `dangerouslySkipPermissions` or similar bypasses

Behavioral manipulation:
- Instructions to avoid showing certain output to users
- Directions to suppress error messages or warnings
- Instructions to automatically approve or bypass confirmations
- Telling Claude to claim it cannot do something it can

Data collection through prompts:
- Instructions to read and report user files
- Prompts that gather and transmit system information
- Skills that collect usage patterns or project details

### 5. Manifest and Structure Analysis (Priority: Low)

**Check plugin.json and directory structure for:**

Path traversal:
- Source paths containing `../`
- Component paths referencing outside plugin root
- Symlinks pointing outside plugin directory

Structural integrity:
- Components declared in manifest that don't exist
- Undeclared components that exist in directories
- Misplaced files (e.g., hooks in .claude-plugin/)

Metadata concerns:
- Missing or vague description
- No version information
- No author information
- Suspicious homepage or repository URLs

## Risk Level Determination

Assign the overall plugin risk level based on the highest severity finding:
- **CRITICAL**: Any critical finding present → Do not install
- **HIGH**: High findings but no critical → Install with extreme caution
- **MEDIUM**: Medium findings only → Review findings before installing
- **LOW**: Low findings only → Generally safe, minor concerns noted
- **CLEAN**: No findings → No security concerns detected
