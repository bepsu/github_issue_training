# 初期セットアップ手順

このドキュメントでは、タスク管理システムの初期セットアップ方法を説明します。

## 前提条件

- GitHub CLIがインストールされていること
- `gh auth login` で認証済みであること

```bash
# 確認コマンド
gh auth status
```

---

## 1. リポジトリの作成

```bash
# Privateリポジトリを作成
gh repo create {your-repo} --private --clone

# 既存リポジトリを使う場合はclone
gh repo clone {your-username}/{your-repo}
```

---

## 2. GitHub Projectの作成（GitHub UIで作成）

**重要**: CLIではなく、GitHub UIからカンバンテンプレートを使用してProjectを作成してください。

### 理由
- CLIで作成すると Status の選択肢が「Todo / In Progress / Done」の3つのみ
- UIのカンバンテンプレートなら、必要なStatus選択肢やビューが最初から設定済み
- CLIでのStatus選択肢の追加・変更は GraphQL API が必要で複雑

### 手順

1. https://github.com にアクセスし、右上の「+」ボタン → 「New project」を選択
2. 「Featured」セクションから「Kanban」テンプレートを選択
3. Project名を入力（例: "Task Management"）して作成

### Project番号の確認

作成後、以下のコマンドでProject番号を確認：

```bash
gh project list --owner "@me"
```

または、ProjectのURLから確認できます：
`https://github.com/users/{username}/projects/{番号}`

---

## 3. Projectフィールドの設定

カンバンテンプレートで作成した場合、Statusフィールドは設定済みです。
必要に応じて、GitHub UI または CLI で追加フィールドを作成します。

### UIでの設定（推奨）

1. Projectを開く
2. 右上の「...」→「Settings」→「Custom fields」
3. 「New field」から以下を追加：
   - **Priority**: Single select（High, Medium, Low）
   - **Due date**: Date
   - **Estimate**: Number

### CLIでの設定（オプション）

```bash
OWNER="@me"  # または Organization名
PROJECT_NUMBER=1  # 作成したProject番号

# Priority
gh project field-create $PROJECT_NUMBER --owner "$OWNER" --name "Priority" --data-type "SINGLE_SELECT" --single-select-options "High,Medium,Low"

# Due date
gh project field-create $PROJECT_NUMBER --owner "$OWNER" --name "Due date" --data-type "DATE"

# Estimate
gh project field-create $PROJECT_NUMBER --owner "$OWNER" --name "Estimate" --data-type "NUMBER"
```

### Priorityフィールドの修正（P0/P1/P2 → High/Medium/Low）

カンバンテンプレートで作成した場合、Priorityが「P0/P1/P2」形式になっていることがあります。
「High/Medium/Low」に変更するには、既存フィールドを削除して再作成します。

```bash
OWNER="@me"
PROJECT_NUMBER=1

# 1. 既存のPriorityフィールドIDを確認
gh project field-list $PROJECT_NUMBER --owner "$OWNER" --format json | jq '.fields[] | select(.name == "Priority")'

# 2. フィールドを削除（IDを確認して実行）
gh project field-delete --id "PVTSSF_xxxxx"

# 3. 新しいPriorityフィールドを作成
gh project field-create $PROJECT_NUMBER --owner "$OWNER" --name "Priority" --data-type "SINGLE_SELECT" --single-select-options "High,Medium,Low"
```

### フィールド一覧

| フィールド | タイプ | 値 | 備考 |
|-----------|--------|-----|------|
| Status | Single Select | Backlog, Ready, In Progress, Done | カンバンテンプレートで作成済み（必要に応じてUIで編集） |
| Priority | Single Select | High, Medium, Low | 追加作成 |
| Due date | Date | 日付 | 追加作成 |
| Estimate | Number | 時間（1, 2, 4, 8, 16 など）| 追加作成 |

---

## 4. ラベルの作成

リポジトリにラベルを作成します。

```bash
REPO="owner/repo-name"

# ラベル作成
gh label create "task" --repo "$REPO" --color "0052CC" --description "一般的なタスク"
gh label create "bug" --repo "$REPO" --color "D93F0B" --description "バグ修正"
gh label create "research" --repo "$REPO" --color "5319E7" --description "調査・リサーチ"
gh label create "docs" --repo "$REPO" --color "0075CA" --description "ドキュメント作成・更新"
gh label create "improvement" --repo "$REPO" --color "A2EEEF" --description "改善・リファクタリング"
```

---

## 5. 設定ファイルの作成

プロジェクトルートに `.claude/task-config.json` を作成します。

```bash
mkdir -p .claude
cat > .claude/task-config.json << 'EOF'
{
  "repository": "owner/repo-name",
  "project_number": 1,
  "default_assignee": "your-username"
}
EOF
```

### 設定項目

| キー | 説明 | 例 |
|------|------|-----|
| `repository` | GitHubリポジトリ（owner/repo形式） | `"myuser/my-tasks"` |
| `project_number` | GitHub Project の番号 | `1` |
| `default_assignee` | デフォルトの担当者（オプション） | `"myuser"` |

---

## 6. ディレクトリ構造の作成

```bash
# 調査・リサーチ用フォルダ
mkdir -p docs/research

# .gitkeep を配置（空フォルダをGit管理するため）
touch docs/research/.gitkeep
```

---

## 7. 動作確認

### gh コマンドの確認

```bash
# Issue作成テスト
gh issue create --repo "$REPO" --title "テストIssue" --body "動作確認用" --label "task"

# Project一覧取得
gh project item-list PROJECT_NUMBER --owner "@me" --format json

# Issue一覧
gh issue list --repo "$REPO"
```

### 確認項目

- [ ] Issueが作成できる
- [ ] IssueがProjectに追加される
- [ ] ラベルが正しく設定される
- [ ] Projectのフィールド（Status, Priority等）が存在する

---

## トラブルシューティング

### `gh project` コマンドが使えない

GitHub CLI のバージョンが古い可能性があります。

```bash
gh --version
# 2.0.0 以上が必要

# アップデート
gh upgrade
```

### 認証エラー

```bash
# 再認証
gh auth login

# スコープの追加が必要な場合
gh auth refresh -s project
```

### Projectにアイテムを追加できない

Project が同じ owner（ユーザーまたはOrg）に属しているか確認してください。

---

## 初期化完了後

セットアップが完了したら、Claude Codeに以下のように依頼できます：

- 「タスクを作成して」
- 「次にやるべきタスクを選んで」
- 「タスク一覧を見せて」
- 「#42 のタスクに着手して」

詳細は [CLAUDE.md](../../CLAUDE.md) を参照してください。
