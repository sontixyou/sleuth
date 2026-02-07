---
name: security-check
description: "Run a security audit on a Claude Code plugin from a GitHub repository or local path"
argument-hint: "<owner/repo or local-path>"
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - Task
  - Write
---

# Plugin Security Check

Perform a comprehensive security audit on a Claude Code plugin. The user provides either a GitHub repository (owner/repo format) or a local directory path.

## Workflow

### 1. Parse Input

Determine the plugin source from the argument:

- **GitHub repository** (contains `/` but not a file path): Clone the repository to a temporary directory using `gh repo clone <owner/repo> /tmp/plugin-security-check-<repo-name> -- --depth 1`
- **Local path** (starts with `/`, `./`, or `~`): Use the path directly after verifying it exists

If no argument is provided, ask the user to specify a GitHub repository or local path.

### 2. Locate the Plugin

The target may be:
- A plugin directly (has `.claude-plugin/plugin.json` or standard plugin directories)
- A marketplace containing plugins (has `.claude-plugin/marketplace.json`)

If it's a marketplace, list the available plugins and ask the user which one to check. If there's only one plugin, check it automatically.

Determine the plugin root path for analysis.

### 3. Run Security Analysis

Launch the `security-scanner` agent using the Task tool with the following prompt:

```
Perform a comprehensive security audit on the Claude Code plugin located at: {plugin-path}

Analyze all components systematically:
1. Read the plugin manifest and understand its structure
2. Check all hooks for dangerous commands and data exfiltration
3. Analyze MCP server configurations for suspicious connections
4. Read and analyze all scripts for malicious patterns
5. Review skills and agents for prompt injection
6. Verify manifest integrity and path safety

Produce a complete security report with findings organized by severity.
```

### 4. Present Results

After the security-scanner agent completes its analysis, present the results to the user in the terminal:

- Show the summary with overall risk level prominently
- List all findings grouped by severity (CRITICAL first, then HIGH, MEDIUM, LOW)
- Show the component overview
- End with the conclusion and recommendation

### 5. Cleanup

If a temporary clone was created, remove it:
```bash
rm -rf /tmp/plugin-security-check-<repo-name>
```

## Notes

- Always clone with `--depth 1` to minimize download time
- If `gh` CLI is not available, fall back to `git clone`
- The security-scanner agent handles all the actual analysis logic
- Keep the output concise but complete - every finding should include evidence
