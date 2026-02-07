# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

sleuth — Claude Code プラグインのセキュリティ問題を追跡・発見するマーケットプレイスリポジトリ。プラグインのカタログ管理と配布を行う。ビルドツールやテストフレームワークは使用していない。

## Install

```bash
claude plugin marketplace add <path-to-this-repo>
```

## Architecture

### Marketplace 構造

- `.claude-plugin/marketplace.json` — マーケットプレイスのマニフェスト。`plugins` 配列に登録済みプラグインのメタデータ（name, source, description, version, author, category, tags）を記述
- `plugins/` — 各プラグインのディレクトリ。`marketplace.json` の `source` フィールドがこのディレクトリ内のサブディレクトリ名に対応

### Plugin 構造（`plugins/<plugin-name>/` 内）

各プラグインは Claude Code のプラグイン規約に従う:

- `.claude-plugin/plugin.json` — プラグインマニフェスト
- `commands/` — スラッシュコマンド定義（Markdown frontmatter + 手順）
- `agents/` — サブエージェント定義（Markdown frontmatter + プロンプト）
- `skills/` — スキル定義（`SKILL.md` + `references/` で参照資料）
- `hooks/` — フック設定（存在する場合）
- `scripts/` — 実行スクリプト（存在する場合）

### 現在のプラグイン

- **plugin-security-checker** — Claude Code プラグインのセキュリティ監査ツール。`security-check` コマンドがエントリーポイント、`security-scanner` エージェントが分析エンジン、`plugin-security` スキルがセキュリティパターンの知識ベース

## Adding a New Plugin

1. `plugins/<plugin-name>/` にプラグインディレクトリを作成
2. `.claude-plugin/plugin.json` にプラグインマニフェストを配置
3. `.claude-plugin/marketplace.json` の `plugins` 配列にエントリを追加
4. プラグインの `README.md` を作成

## PR / Issue

- PR テンプレート: `.github/PULL_REQUEST_TEMPLATE.md`（日本語）
- Issue テンプレート: `.github/ISSUE_TEMPLATE/`（bug_report, feature_request, plugin_request）
- PR チェックリスト: `plugins/` の構成確認、`marketplace.json` の更新、ドキュメント更新、ローカル動作確認
