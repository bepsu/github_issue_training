# Task Management System with Claude Code

GitHub Issues + GitHub Projects を使ったタスク管理システム。Claude Code がタスクの作成・選択・実行を支援します。

## クイックスタート

### 1. テンプレートから自分のリポジトリを作成

1. [このリポジトリ](https://github.com/masatomix/claude-github_issue-task-template)にアクセス
2. 右上の緑色「**Use this template**」ボタン → 「Create a new repository」を選択
3. リポジトリ名を入力して「Create repository」をクリック

### 2. 作成したリポジトリをクローン

```bash
git clone https://github.com/{your-username}/{your-repo}.git
cd {your-repo}
```

### 3. GitHub CLI の認証（未認証の場合）

```bash
gh auth login
```

### 4. Claude Code を起動

```bash
claude
```

### 5. GitHub Project の作成（GitHub UIで実施）

**重要**: GitHub UIからカンバンテンプレートを使用してProjectを作成してください。

1. https://github.com にアクセスし、右上の「+」ボタン → 「New project」を選択
2. 「Featured」セクションから「Kanban」テンプレートを選択
3. Project名を入力（例: "Task Management"）して作成

> CLIではなくUIで作成する理由: カンバンテンプレートならStatus選択肢やビューが最初から設定済みのため

### 6. Claude Code に初期設定を依頼

Claude Code に「タスク管理の初期設定をして」と伝えてください。以下を対話的に設定してくれます：

- 設定ファイル（`.claude/task-config.json`）の作成
- 追加Projectフィールドの作成（Priority, Due date, Estimate）
- ラベルの作成

### 7. タスク管理開始

初期設定が完了したら、Claude Code に話しかけてタスク管理を始めましょう：

- 「タスクを作成して」
- 「次にやるべきタスクを選んで」
- 「タスク一覧を見せて」
- 「#1 のタスクに着手して」

## ワークフロー

```
[タスク作成] → [Backlog] → [Ready] → [In Progress] → [Done/Close]
```

## ドキュメント

- [CLAUDE.md](CLAUDE.md) - Claude Code への指示（メイン）
- [docs/task-management/SETUP.md](docs/task-management/SETUP.md) - 詳細なセットアップ手順
- [docs/task-management/COMMANDS.md](docs/task-management/COMMANDS.md) - gh コマンドリファレンス
- [docs/task-management/WORKFLOW.md](docs/task-management/WORKFLOW.md) - タスク選択・実行の詳細ガイド

## 前提条件

- [GitHub CLI](https://cli.github.com/)
- [Claude Code](https://claude.ai/code)

## ライセンス

MIT
