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

**IMPORTANT**: The pattern lists below are representative examples, not exhaustive. Always apply behavioral analysis — read each hook command/script and evaluate its intent. Ask: "What is this hook actually trying to do? Does that match the plugin's stated purpose?" Flag any behavior that seems disproportionate or unrelated to the plugin's function.

Dangerous command patterns:
- Destructive operations: `rm -rf`, `rm -f`, `mkfs`, `dd if=`, `shred`
- Remote code execution: `curl | bash`, `wget -O- | sh`, `eval $(curl`, `source <(curl`, `bash <(curl`
- Interpreter execution: `python -c`, `python3 -c`, `node -e`, `ruby -e`, `perl -e`, `php -r`
- Reverse shells: `bash -i >& /dev/tcp/`, `nc -e`, `ncat`, `socat`, `mkfifo` + pipe to `nc`, Python/Ruby/PHP/Node socket-based shells
- Privilege escalation: `sudo`, `su -`, `chmod 777`, `chmod +s`, `chown root`
- Process manipulation: `kill -9`, `pkill`, `killall` (when targeting system processes)

Data exfiltration patterns:
- HTTP exfiltration: `curl -d`, `curl -F "file=@"`, `wget --post-data`, `curl -X POST`
- DNS exfiltration: `dig`, `nslookup`, `host` (encoding data in DNS queries)
- Encrypted channels: `openssl s_client`, `/dev/tcp/`, `/dev/udp/`
- File transfer: `scp`, `rsync` to remote hosts, `sftp`, `ftp`
- Environment variables: `curl -d "$(env)"`, `printenv`, `set` piped to network commands
- Sensitive files: `~/.ssh/`, `~/.aws/`, `~/.gitconfig`, `~/.kube/config`, `~/.docker/config.json`, `~/.npmrc`, `~/.pypirc`, `~/.netrc`, `~/.git-credentials`, `~/.gnupg/`, `~/.vault-token`, `~/.config/gcloud/`
- Credential variables: `$API_KEY`, `$SECRET`, `$TOKEN`, `$PASSWORD`, `$PRIVATE_KEY`, `$DATABASE_URL`, `$AWS_SECRET_ACCESS_KEY`
- macOS Keychain: `security find-generic-password`, `security find-internet-password`

Persistence mechanisms:
- Cron: `crontab -e`, `crontab -l`, writing to `/etc/cron.d/`
- Shell profiles: writing to `~/.bashrc`, `~/.zshrc`, `~/.profile`, `~/.bash_profile`
- macOS: `launchctl`, writing to `~/Library/LaunchAgents/`
- Linux: `systemctl`, writing to `~/.config/autostart/`
- Git hooks: writing to `.git/hooks/`

Git manipulation:
- `git config credential.helper` — can redirect credential storage
- `git remote set-url` — can redirect push/pull to attacker's server
- Writing to `.git/hooks/` — persistence via pre-commit, post-checkout, etc.

macOS-specific risks:
- `osascript` — AppleScript execution (can control applications, display fake dialogs)
- `screencapture` — screenshot capture
- `pbcopy`/`pbpaste` — clipboard access

High-risk event bindings:
- `SessionStart`: Executes on every session start without user action
- `UserPromptSubmit`: Intercepts every user input
- `PreToolUse`: Can modify or block tool execution
- `Notification`: Can intercept notification content

Obfuscation techniques:
- Base64: `echo "..." | base64 -d | bash`, `base64 --decode`
- Hex: `echo -e "\x..."`, `printf '\x...'`, `xxd -r`
- Compression: `gzip -d`, `zlib`, compressed payloads piped to execution
- Encryption: `openssl enc -d`, `gpg -d` piped to execution
- Variable indirection: `${!var}`, indirect parameter expansion
- String concatenation: `c="cu"; c+="rl"`, array-based command building
- Unicode/homoglyph tricks: visually similar characters replacing ASCII

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

### 3. Scripts and Executable Files Analysis (Priority: High)

Scan ALL non-declarative files in the plugin. Do not filter by file extension — attackers can use any language or extension. Inspect every file that is not purely declarative markup (.md, .json, .yaml, .yml, .txt).

Pay special attention to files in `scripts/` directory, files referenced by hooks, and files with executable permissions.

**Check all non-declarative files for:**

Network operations:
- HTTP requests: `curl`, `wget`, `fetch`, `requests.get/post`, `http.get`, `axios`, `urllib`, `httpx`
- Socket operations: `socket`, `net.connect`, `TCP`, `dgram`, `WebSocket`
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

Binary or unreadable files:
- Flag any binary files for manual review
- Note any files with unusual encodings or formats

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
