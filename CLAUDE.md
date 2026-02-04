# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

GitHub Issues + GitHub Projects を使ったタスク管理システムの**テンプレートリポジトリ**。

このリポジトリをフォークまたはコピーして、自分のプロジェクトに適用してください。
設定ファイル（`.claude/task-config.json`）で管理対象のリポジトリと Project を指定できます。

Claude Codeがタスクの作成・選択・実行を支援する。

## 設定ファイル

`.claude/task-config.json` に設定を保存（テンプレート: `.claude/task-config.json.template`）

```json
{
  "repository": "owner/repo-name",
  "project_number": 1,
  "default_assignee": "username"
}
```

設定値の取得:
```bash
CONFIG_FILE=".claude/task-config.json"
REPO=$(jq -r '.repository' "$CONFIG_FILE")
PROJECT_NUMBER=$(jq -r '.project_number' "$CONFIG_FILE")
OWNER=$(echo "$REPO" | cut -d'/' -f1)
```

## ワークフロー

```
[タスク作成] → [Backlog] → [Ready] → [In Progress] → [Done/Close]
```

## タスク作成

ユーザーから依頼されたら以下を確認してIssue起票:
- **必須**: タイトル、概要・背景、受け入れ条件
- **オプション**: 優先度(High/Medium/Low)、期限、見積もり工数、ラベル、担当者

```bash
# Issue作成
gh issue create --repo "$REPO" --title "タイトル" --body "本文" --label "task"

# Projectに追加
gh project item-add $PROJECT_NUMBER --owner "$OWNER" --url "https://github.com/$REPO/issues/ISSUE_NUMBER"
```

Issueテンプレート:
```markdown
## 概要
（何をするか簡潔に）

## 背景・目的
（なぜこのタスクが必要か）

## 受け入れ条件
- [ ] 条件1
- [ ] 条件2

## 備考
（参考リンク、注意点など）
```

## タスク選択

選択優先順位: 1) Priority: High → 2) Status: Ready → 3) 期限が近い → 4) 見積もり工数が小さい

ユーザーに「自分のタスクのみ？全体から？」を確認すること。

```bash
gh project item-list $PROJECT_NUMBER --owner "$OWNER" --format json --limit 100
```

## タスク着手

1. Statusを「In Progress」に変更
2. featureブランチ作成: `feature/{issue番号}-{短い説明}`
3. 成果物フォルダ作成（必要な場合）:
   - **調査・設計**: `docs/research/{issue番号}_{タスク名}/`
   - **アプリ開発**: `docs/app/{issue番号}_{タスク名}/`

```bash
git checkout -b feature/42-add-user-auth
mkdir -p docs/research/42_add-user-auth  # 調査・設計の場合
mkdir -p docs/app/42_add-user-auth       # アプリ開発の場合
```

## タスク完了

1. 変更をコミット・プッシュ
2. PR作成（`Closes #番号` を含める → マージ時にIssue自動Close）
3. **Issueに成果物リンクを追記**（調査レポート、ドキュメント等がある場合）

```bash
gh pr create --repo "$REPO" --title "feat: 機能説明 (#42)" --body "Closes #42" --base main

# 成果物リンクをIssueに追記
gh issue edit 42 --repo "$REPO" --body "$(gh issue view 42 --repo "$REPO" --json body -q .body)

## 成果物
- [調査レポート](docs/research/42_task-name/README.md)"
```

### 成果物リンクの追記ルール

タスク完了時、成果物（調査レポート、設計書、ドキュメント等）がある場合は、Issueの本文に「## 成果物」セクションを追加し、リンクを記載する。

```markdown
## 成果物
- [調査レポート](docs/research/{issue番号}_{タスク名}/README.md)
- [設計書](docs/app/{issue番号}_{タスク名}/design.md)
```

## ラベル

| ラベル | 説明 |
|--------|------|
| `task` | 一般的なタスク |
| `bug` | バグ修正 |
| `research` | 調査・リサーチ |
| `docs` | ドキュメント作成・更新 |
| `improvement` | 改善・リファクタリング |

## Projectフィールド

| フィールド | タイプ | 値 |
|-----------|--------|-----|
| Status | Single Select | Backlog, Ready, In Progress, In review, Done |
| Priority | Single Select | High, Medium, Low |
| Target date | Date | 日付（期限） |
| Estimate | Number | 時間（1, 2, 4, 8, 16 など） |

## 初期設定

ユーザーから「タスク管理の初期設定をして」と依頼されたら、[SETUP.md](docs/task-management/SETUP.md) の手順に従って対話的にセットアップを実行する。

主な作業:
1. gh 認証確認
2. **GitHub Project 作成（GitHub UIでカンバンテンプレートから作成を案内）**
   - CLIではなくUIで作成する理由: Status選択肢やビューが最初から設定済みのため
3. Project番号の確認（`gh project list --owner "@me"`）
4. 設定ファイル（`.claude/task-config.json`）作成
5. ラベル作成
6. 追加フィールド作成（Priority, Due date, Estimate）- UIまたはCLI

## 前提条件

- `gh auth login` で認証済みであること

## 関連ドキュメント

- [SETUP.md](docs/task-management/SETUP.md) - 初期セットアップ手順
- [COMMANDS.md](docs/task-management/COMMANDS.md) - ghコマンドリファレンス
- [WORKFLOW.md](docs/task-management/WORKFLOW.md) - タスク選択・実行ガイド詳細

---

## 進捗メモ（2026-02-04 更新）

### 完了タスク
- [x] #1 プライム・ブレインズ社を調査する → [調査レポート](docs/research/1_prime-brains-research/README.md)

### 次にやるべきタスク（優先順）
1. **#2 本番環境のセキュリティ監査を実施する** - High, 期限超過
2. **#3 新規APIエンドポイントの設計** - High
3. #4 ユーザーマニュアルを更新する - Medium, 期限超過
4. #5 テストカバレッジを改善する - Medium
5. #6 開発環境のDockerイメージを最適化 - Low

### メモ
- サンプルタスク（#2〜#6）はテスト用に作成したもの
- 実際のタスクを追加する場合は「タスクを作成して」と依頼
