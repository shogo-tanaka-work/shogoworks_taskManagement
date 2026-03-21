# GitHub Actions ワークフロー解説: assign-sprint.yml

## ファイルの場所
`.github/workflows/assign-sprint.yml`
- `.github/workflows/` 配下に YAML ファイルを置くと、GitHub が自動的に Actions ワークフローとして認識する

---

## 構造と解説

### 1. `name:` — ワークフロー名
```yaml
name: Auto-assign Sprint from Due Date
```
GitHub Actions タブに表示される名前。自由に変更可能。

### 2. `on:` — 実行タイミング
```yaml
on:
  issues:
    types: [opened, edited, labeled]
  workflow_dispatch:
    inputs:
      issue_number:
        description: "Issue number to process (blank = all issues)"
        required: false
```

**いつ動くか：**
| トリガー | タイミング |
|----------|-----------|
| `issues: opened` | 新しいIssueが作成されたとき |
| `issues: edited` | Issueのタイトルや本文が編集されたとき |
| `issues: labeled` | Issueにラベルが付けられたとき |
| `workflow_dispatch` | GitHub Actions画面から手動で「Run workflow」ボタンを押したとき |

**注意点:** 期限（Dateフィールド）はProjectsのカスタムフィールドなので、`issues: edited` では検知されない場合がある。Projects上で期限だけ変えた場合は手動実行が確実。

### 3. `env:` — 環境変数
```yaml
env:
  PROJECT_NUMBER: "2"
  PROJECT_OWNER: "shogo-tanaka-work"
```
プロジェクト番号やオーナー名を変数化。複数箇所で参照するので1箇所で管理。

### 4. `jobs:` — 実際の処理
```yaml
jobs:
  assign-sprint:
    runs-on: ubuntu-latest
    permissions:
      issues: read
      repository-projects: write
```

### 5. `steps:` — 処理の手順
```yaml
steps:
  - name: Assign Sprint based on due date
    uses: actions/github-script@v7
    with:
      github-token: ${{ secrets.SHOGOWORKS_TASKMANAGEMENT_PROJECT_PAT }}
```
- `actions/github-script@v7` = JavaScript でGitHub APIを直接呼べる公式アクション
- `secrets.SHOGOWORKS_TASKMANAGEMENT_PROJECT_PAT` = リポジトリのSecretsに登録したClassic PAT

### 6. `script:` — メインロジック

処理の流れ：
```
① GraphQL APIでプロジェクトの全データを取得
   - Sprint フィールドの全イテレーション一覧
   - 全Issueの「期限」と「現在のSprint」

② 各Issueをループ処理
   ├─ 期限なし → スキップ
   ├─ 期限の日付が含まれるSprintを検索
   │   └─ 該当なし → 警告を出してスキップ
   ├─ 既に正しいSprintが割り当て済み → スキップ
   └─ Sprint を割り当てる（GraphQL mutation）
```

---

## 実行方法

### 自動実行
Issue を作成・編集・ラベル付けすると自動で発火

### 手動実行
1. リポジトリ → Actions タブ → 「Auto-assign Sprint from Due Date」
2. 「Run workflow」ボタン
3. Issue番号を入力（空欄なら全Issue処理）

### 実行結果の確認
- Actions タブ → 該当のワークフロー実行をクリック → ログで割り当て結果が見える

---

## 現在の制約・改善余地
- Projects上で「期限」フィールドだけを変更した場合は `issues: edited` で検知されないことがある → 手動実行で対応
- `projects_v2_item` イベントを使えば Projects のフィールド変更を直接検知できるが、PAT の権限設定が複雑になる
