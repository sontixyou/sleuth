# sleuth

A marketplace that tracks and uncovers security issues in Claude Code plugins — like a detective.

## Install

```bash
claude plugin marketplace add <path-to-this-repo>
```

## Structure

```
sleuth/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── plugin-security-checker/
└── README.md
```

## Plugins

| Plugin | Description |
|--------|-------------|
| [plugin-security-checker](./plugins/plugin-security-checker/) | Security audit tool for Claude Code plugins |

## Adding plugins

Add your plugin to the `plugins/` directory and register it in the `plugins` array of `marketplace.json`.
