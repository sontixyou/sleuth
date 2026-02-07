# Security Report Template

Structure the security report as follows:

```
# Plugin Security Report: {plugin-name}

## Summary
- **Plugin**: {name} v{version}
- **Source**: {github-repo or path}
- **Risk Level**: {CRITICAL|HIGH|MEDIUM|LOW|CLEAN}
- **Findings**: {count} issues ({critical} critical, {high} high, {medium} medium, {low} low)

## Findings

### [SEVERITY] Finding Title
- **Component**: {hooks|mcp|scripts|skills|agents|manifest}
- **File**: {relative path}
- **Line**: {line number if applicable}
- **Description**: What was found and why it's a risk
- **Evidence**: The specific code or configuration
- **Recommendation**: How to mitigate

## Component Overview
Summary table of what the plugin contains:
- Commands: {count}
- Agents: {count}
- Skills: {count}
- Hooks: {count} ({event types})
- MCP Servers: {count}
- Scripts: {count}

## Conclusion
Overall assessment and recommendation (install/caution/avoid)
```
