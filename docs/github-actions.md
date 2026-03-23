# GitHub Actions ワークフロー設定ドキュメント

## 概要

本リポジトリでは、`.github/workflows/` ディレクトリに以下のGitHub Actionsワークフローが定義されています。

---

## ワークフロー一覧

### 1. Claude Code (`claude.yml`)

#### 目的
IssueやPull Requestのコメントで `@claude` をメンションすることで、ClaudeによるAI支援を受けられるワークフローです。

#### トリガー条件

| イベント | 条件 |
|----------|------|
| `issue_comment` (created) | コメント本文に `@claude` を含む場合 |
| `pull_request_review_comment` (created) | レビューコメント本文に `@claude` を含む場合 |
| `pull_request_review` (submitted) | レビュー本文に `@claude` を含む場合 |
| `issues` (opened / assigned) | Issue本文またはタイトルに `@claude` を含む場合 |

#### 権限

| 権限 | レベル |
|------|--------|
| `contents` | read |
| `pull-requests` | read |
| `issues` | read |
| `id-token` | write |
| `actions` | read（CI結果の読み取り用） |

#### 使用アクション
- `actions/checkout@v4`
- `anthropics/claude-code-action@v1`

#### 必要なシークレット
- `CLAUDE_CODE_OAUTH_TOKEN`: Claude Code OAuth認証トークン

#### 使い方
IssueまたはPRのコメントに `@claude` を含めてメンションするだけで、Claudeが自動的にタスクを実行します。

---

### 2. Claude Code Review (`claude-code-review.yml`)

#### 目的
Pull Requestが作成・更新された際に、ClaudeによるAI自動コードレビューを実行するワークフローです。

#### トリガー条件

| イベント | 条件 |
|----------|------|
| `pull_request` (opened) | PRが新規作成された時 |
| `pull_request` (synchronize) | PRにコミットが追加された時 |
| `pull_request` (ready_for_review) | ドラフトPRがレビュー可能になった時 |
| `pull_request` (reopened) | クローズされたPRが再オープンされた時 |

#### 権限

| 権限 | レベル |
|------|--------|
| `contents` | read |
| `pull-requests` | read |
| `issues` | read |
| `id-token` | write |

#### 使用アクション
- `actions/checkout@v4`
- `anthropics/claude-code-action@v1`（`code-review` プラグイン使用）

#### 使用プラグイン
- プラグインマーケットプレイス: `https://github.com/anthropics/claude-code.git`
- プラグイン: `code-review@claude-code-plugins`

#### 必要なシークレット
- `CLAUDE_CODE_OAUTH_TOKEN`: Claude Code OAuth認証トークン

#### 備考
- 特定の著者のPRのみに実行対象を絞りたい場合は、ジョブの `if` 条件にフィルターを追加できます（ファイル内コメント参照）
- 特定ファイルパスの変更のみを対象にしたい場合は、`paths` フィルターを有効化できます（ファイル内コメント参照）

---

### 3. Hello World (`hello.yml`)

#### 目的
動作確認・サンプル用のシンプルなワークフローです。

#### トリガー条件

| イベント | 条件 |
|----------|------|
| `push` | `main` ブランチへのプッシュ時 |

#### 処理内容
1. リポジトリをチェックアウト
2. `echo "Hello, World!"` を実行

#### 使用アクション
- `actions/checkout@v5`

#### 備考
このワークフローは学習・テスト目的のサンプルです。実際の開発には影響しません。

---

## シークレット設定

ワークフローを正常に動作させるには、以下のシークレットをリポジトリに設定する必要があります。

| シークレット名 | 用途 | 設定場所 |
|----------------|------|----------|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code APIへの認証 | GitHub リポジトリ Settings > Secrets and variables > Actions |

### シークレットの設定方法
1. GitHubリポジトリの **Settings** タブを開く
2. 左メニューの **Secrets and variables** > **Actions** を選択
3. **New repository secret** をクリック
4. Name に `CLAUDE_CODE_OAUTH_TOKEN`、Value にトークンを入力して保存

---

## ワークフロー間の関係

```mermaid
graph TD
    A[開発者] -->|Issue/PRにコメント| B[claude.yml]
    A -->|PRを作成・更新| C[claude-code-review.yml]
    A -->|mainブランチにプッシュ| D[hello.yml]

    B -->|@claudeメンション検知| E[Claude AIが応答・タスク実行]
    C -->|PR自動検知| F[Claude AIがコードレビュー実施]
    D -->|プッシュ検知| G[Hello World出力]
```
