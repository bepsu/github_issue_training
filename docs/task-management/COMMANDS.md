# コマンドリファレンス

Claude Codeがタスク管理で使用する `gh` コマンドのリファレンスです。

## 環境変数の設定

設定ファイル（`.claude/task-config.json`）から読み込んで使用します。

```bash
# 設定ファイルから値を取得
CONFIG_FILE=".claude/task-config.json"
REPO=$(jq -r '.repository' "$CONFIG_FILE")
PROJECT_NUMBER=$(jq -r '.project_number' "$CONFIG_FILE")
OWNER=$(echo "$REPO" | cut -d'/' -f1)
```

---

## Issue操作

### Issue作成

```bash
gh issue create \
  --repo "$REPO" \
  --title "タイトル" \
  --body "本文" \
  --label "task" \
  --assignee "username"
```

### Issue一覧

```bash
# 全てのオープンIssue
gh issue list --repo "$REPO" --state open

# 自分にアサインされたもの
gh issue list --repo "$REPO" --assignee "@me"

# ラベルでフィルタ
gh issue list --repo "$REPO" --label "bug"

# JSON形式で取得
gh issue list --repo "$REPO" --json number,title,labels,assignees
```

### Issue詳細取得

```bash
gh issue view ISSUE_NUMBER --repo "$REPO"

# JSON形式
gh issue view ISSUE_NUMBER --repo "$REPO" --json title,body,labels,state
```

### Issueを閉じる

```bash
gh issue close ISSUE_NUMBER --repo "$REPO"
```

---

## Project操作

### Project一覧

```bash
gh project list --owner "$OWNER"
```

### Projectアイテム一覧

```bash
gh project item-list $PROJECT_NUMBER \
  --owner "$OWNER" \
  --format json \
  --limit 100
```

### ProjectにIssueを追加

```bash
gh project item-add $PROJECT_NUMBER \
  --owner "$OWNER" \
  --url "https://github.com/$REPO/issues/ISSUE_NUMBER"
```

### Projectアイテムの編集

フィールドの値を変更するには、まずフィールドIDとオプションIDを取得する必要があります。

#### フィールド情報の取得

```bash
gh project field-list $PROJECT_NUMBER --owner "$OWNER" --format json
```

#### アイテムIDの取得

```bash
gh project item-list $PROJECT_NUMBER --owner "$OWNER" --format json | \
  jq '.items[] | select(.content.number == ISSUE_NUMBER) | .id'
```

#### フィールド値の更新（Single Select）

```bash
gh project item-edit \
  --project-id "PROJECT_ID" \
  --id "ITEM_ID" \
  --field-id "FIELD_ID" \
  --single-select-option-id "OPTION_ID"
```

#### フィールド値の更新（Number）

```bash
gh project item-edit \
  --project-id "PROJECT_ID" \
  --id "ITEM_ID" \
  --field-id "FIELD_ID" \
  --number 8
```

#### フィールド値の更新（Date）

```bash
gh project item-edit \
  --project-id "PROJECT_ID" \
  --id "ITEM_ID" \
  --field-id "FIELD_ID" \
  --date "2024-12-31"
```

---

## ブランチ・PR操作

### ブランチ作成

```bash
git checkout -b feature/ISSUE_NUMBER-short-description
```

### 変更をプッシュ

```bash
git push -u origin feature/ISSUE_NUMBER-short-description
```

### PR作成

```bash
gh pr create \
  --repo "$REPO" \
  --title "feat: 機能の説明 (#ISSUE_NUMBER)" \
  --body "## 概要
変更の説明

## 関連Issue
Closes #ISSUE_NUMBER

## 変更内容
- 変更点1
- 変更点2" \
  --base main
```

### PR一覧

```bash
gh pr list --repo "$REPO"
```

### PRをマージ

```bash
gh pr merge PR_NUMBER --repo "$REPO" --merge
```

---

## ラベル操作

### ラベル作成

```bash
gh label create "label-name" \
  --repo "$REPO" \
  --color "HEXCOLOR" \
  --description "説明"
```

### ラベル一覧

```bash
gh label list --repo "$REPO"
```

---

## 便利なワンライナー

### タスク選択用：優先度・期限でソートしたタスク一覧

```bash
gh project item-list $PROJECT_NUMBER --owner "$OWNER" --format json | \
  jq '[.items[] | select(.status == "Ready" or .status == "Backlog")] | 
      sort_by(.priority // "Low", .dueDate // "9999-12-31")'
```

### 今日期限のタスク

```bash
TODAY=$(date +%Y-%m-%d)
gh project item-list $PROJECT_NUMBER --owner "$OWNER" --format json | \
  jq --arg today "$TODAY" '[.items[] | select(.dueDate == $today)]'
```

### 特定ユーザーにアサインされたIn Progressタスク

```bash
gh issue list --repo "$REPO" --assignee "username" --json number,title | \
  jq '.[] | select(.status == "In Progress")'
```

---

## Project IDの取得方法

`gh project item-edit` には Project ID（node ID）が必要です。

```bash
gh project list --owner "$OWNER" --format json | \
  jq '.projects[] | select(.number == PROJECT_NUMBER) | .id'
```

---

## エラーハンドリング

### コマンド失敗時の対処

```bash
# エラー内容を確認
gh api graphql -f query='...' 2>&1

# 認証状態の確認
gh auth status

# スコープの追加
gh auth refresh -s project -s repo
```

---

## 参考リンク

- [GitHub CLI マニュアル](https://cli.github.com/manual/)
- [gh project コマンド](https://cli.github.com/manual/gh_project)
- [gh issue コマンド](https://cli.github.com/manual/gh_issue)
