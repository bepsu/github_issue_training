# タスク選択・実行ガイド

Claude Codeがタスクを選択・実行する際の詳細なロジックです。

---

## GitHub Flow ルール（重要）

### 絶対に守ること

1. **mainブランチに直接コミット・プッシュしない**
   - すべての変更はfeatureブランチ経由でPRを作成する
   - `git push origin main` は禁止

2. **1つのIssueに1つのブランチ**
   - 異なるIssueの変更を同じブランチに混ぜない
   - ブランチ名: `feature/{issue番号}-{短い説明}`

3. **PRを経由してマージ**
   - 直接mainにマージしない
   - PRにはレビューを受ける（または確認後にマージ）

### 作業フロー図

```
main ────────────────────────────────────────────►
       │                              ▲
       │ checkout -b feature/42-xxx   │ PR & Merge
       ▼                              │
       feature/42-xxx ──commit──commit─┘
```

### 間違えた場合の対処

```bash
# mainに誤ってコミットした場合
git reset --hard origin/main  # ローカルをリセット

# 別のIssueの変更を混ぜてしまった場合
git reset --hard <正しいコミット>
git push --force origin <ブランチ名>  # 注意して使用
```

---

## タスク選択アルゴリズム

ユーザーから「タスクを選んで」「次は何をする？」と聞かれた場合：

### Step 1: タスク一覧を取得

```bash
CONFIG_FILE=".claude/task-config.json"
REPO=$(jq -r '.repository' "$CONFIG_FILE")
PROJECT_NUMBER=$(jq -r '.project_number' "$CONFIG_FILE")
OWNER=$(echo "$REPO" | cut -d'/' -f1)

gh project item-list $PROJECT_NUMBER --owner "$OWNER" --format json --limit 100
```

### Step 2: ユーザーに確認

「自分にアサインされたタスクのみ表示しますか？それとも全タスクから選びますか？」

- `--mine`: 自分のタスクのみ
- `--all`: 全タスク

### Step 3: フィルタリング

以下の条件でフィルタ：
1. Status が `Backlog` または `Ready`（In Progress, Done は除外）
2. `--mine` の場合: Assignee が自分

### Step 4: スコアリング

各タスクにスコアを付ける：

| 条件 | スコア |
|------|--------|
| Status = Ready | +100 |
| Priority = High | +50 |
| Priority = Medium | +20 |
| Priority = Low | +5 |
| 期限が今日 | +40 |
| 期限が3日以内 | +30 |
| 期限が1週間以内 | +20 |
| 期限が過ぎている | +60（緊急） |
| Estimate ≤ 2h | +10（小タスク優先） |

### Step 5: 推奨タスクの提示

スコアが高い順に上位3件を提示：

```
おすすめのタスク：

1. 🔴 #42 ユーザー認証の実装
   Priority: High | Due: 明日 | Estimate: 4h
   Status: Ready

2. 🟡 #38 ドキュメント更新
   Priority: Medium | Due: 3日後 | Estimate: 2h
   Status: Ready

3. 🟢 #45 リファクタリング
   Priority: Low | Due: なし | Estimate: 8h
   Status: Backlog

どれに着手しますか？番号で指定するか、別のタスクを選んでください。
```

---

## タスク着手フロー

ユーザーがタスクを選択したら：

### Step 1: 確認

```
#42「ユーザー認証の実装」に着手します。
- featureブランチ: feature/42-user-auth
- 調査フォルダは必要ですか？（research/docs ラベルの場合は自動作成）
```

### Step 2: 環境準備

```bash
# 最新のmainを取得
git checkout main
git pull origin main

# featureブランチ作成
git checkout -b feature/42-user-auth
```

### Step 3: 調査フォルダ作成（必要な場合）

タスクのラベルが `research` または `docs` の場合、もしくはユーザーが希望した場合：

```bash
mkdir -p docs/research/42_user-auth
touch docs/research/42_user-auth/README.md
```

README.mdの初期内容：
```markdown
# #42 ユーザー認証の実装

## 調査メモ

## 設計案

## 参考リンク

## 決定事項
```

### Step 4: Statusを更新

```bash
# ProjectのStatusを「In Progress」に変更
# （gh project item-edit コマンドで実行）
```

### Step 5: 着手報告

```
✅ #42 に着手しました

- ブランチ: feature/42-user-auth
- 調査フォルダ: docs/research/42_user-auth/
- Status: In Progress に更新済み

何から始めましょうか？
```

---

## タスク完了フロー

### Step 1: 完了確認

受け入れ条件をチェック：

```
#42 の受け入れ条件を確認します：
- [x] ログイン機能の実装
- [x] ログアウト機能の実装
- [ ] パスワードリセット機能

まだ未完了の条件があります。続けますか？それともPRを作成しますか？
```

### Step 2: コミット・プッシュ

```bash
git add .
git commit -m "feat: ユーザー認証機能を実装 (#42)"
git push -u origin feature/42-user-auth
```

### Step 3: PR作成

```bash
gh pr create \
  --repo "$REPO" \
  --title "feat: ユーザー認証機能を実装 (#42)" \
  --body "## 概要
ユーザー認証機能を実装しました。

## 関連Issue
Closes #42

## 変更内容
- ログイン機能
- ログアウト機能
- セッション管理

## テスト
- [x] ログインテスト
- [x] ログアウトテスト"
```

### Step 4: 完了報告

```
✅ PRを作成しました

- PR: #43 feat: ユーザー認証機能を実装 (#42)
- URL: https://github.com/owner/repo/pull/43

レビュー後、マージしてください。
マージすると #42 は自動的にCloseされます。
```

---

## よくある会話パターン

### タスク作成

**ユーザー**: 「ログイン機能を作るタスクを追加して」

**Claude Code**:
```
タスクを作成します。いくつか確認させてください：

1. 優先度は？（High / Medium / Low）
2. 期限はありますか？
3. 見積もり工数は？（例: 4h, 8h）
4. 担当者は自分でいいですか？

デフォルト: Medium / 期限なし / 見積もりなし / 自分
```

### タスク一覧

**ユーザー**: 「今のタスク一覧を見せて」

**Claude Code**:
```
現在のタスク一覧：

📋 Backlog (3)
  - #45 リファクタリング [Low]
  - #46 テスト追加 [Medium]
  - #47 CI設定 [Medium]

✅ Ready (2)
  - #42 ユーザー認証 [High] 期限: 12/10
  - #38 ドキュメント更新 [Medium]

🔄 In Progress (1)
  - #40 API実装 [High] @自分

カンバンをブラウザで開きますか？
```

### 明示的なタスク選択

**ユーザー**: 「#42をやろう」

**Claude Code**:
```
#42「ユーザー認証の実装」ですね。

現在のStatus: Ready
Priority: High
期限: 12/10（3日後）

着手してよろしいですか？
```

---

## エラー時の対応

### 設定ファイルがない場合

```
⚠️ 設定ファイルが見つかりません。

初期設定を行いますか？
必要な情報：
1. GitHubリポジトリ（例: myuser/my-tasks）
2. Project番号（gh project list で確認できます）
```

### ghコマンドが認証されていない場合

```
⚠️ GitHub CLIが認証されていないようです。

以下のコマンドで認証してください：
gh auth login

その後、再度お試しください。
```

### Projectフィールドが存在しない場合

```
⚠️ ProjectにPriorityフィールドが見つかりません。

SETUP.md を参考に、GitHub Projects の Web UI で
カスタムフィールドを作成してください。
```
