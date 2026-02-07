# sleuth

Claude Code プラグインのセキュリティ問題を探偵のように追跡・発見するマーケットプレイス。

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
| [plugin-security-checker](./plugins/plugin-security-checker/) | Claude Code プラグインのセキュリティ監査ツール |

## Adding plugins

`plugins/` ディレクトリにプラグインを追加し、`marketplace.json` の `plugins` 配列に登録してください。
