# PR コメントでデプロイを起動するワークフロー

`.github/workflows/pr-comment-deploy.yml` は、PR へのコメントをトリガーに任意リポジトリの `workflow_dispatch` デプロイを起動する再利用可能ワークフローです。デフォルトではステージング環境のみを受け付けます。

## 使い方
- PR コメントに `/deploy-staging` を投稿すると、対応する `workflow_dispatch` が起動します。
- デフォルトのガードと権限:
  - コマンド実行者の権限が `admin`/`maintain`/`write` 以外の場合は拒否。
  - フォークからの PR は拒否。
- 受理・拒否はいずれも PR コメントでフィードバックされます。

### 主要な入力
- `staging_command`: 受け付けるコマンドキーワード（デフォルト `/deploy-staging`）。
- `staging_env`: dispatch 時に `inputs.environment` として渡す値（デフォルト `dev`）。
- `dispatch_workflow_id`: 起動する `workflow_dispatch` ワークフローのファイル名または ID（デフォルト `deploy.yml`）。
- `allow_permissions`: 実行を許可する権限（カンマ区切り、デフォルト `admin,maintain,write`）。
- `acknowledge_reaction`: 受理したことを示すリアクション（デフォルト `eyes`）。

## 他リポジトリへの組み込み例
`workflow_dispatch` で走るデプロイワークフロー（例: `.github/workflows/deploy.yml`）があることを確認したうえで、以下のワークフローを追加します。

```yaml
name: Deploy via PR comment

on:
  issue_comment:
    types: [created]

jobs:
  pr-comment-deploy:
    if: github.event.issue.pull_request != '' &&
        startsWith(github.event.comment.body, '/deploy-staging') # `with.staging_command` と同じ値にすること
    uses: YOUR-ORG/.github/.github/workflows/pr-comment-deploy.yml@master
    secrets: inherit
    with:
      dispatch_workflow_id: deploy.yml         # 起動する workflow ファイル名
      staging_env: dev                         # dispatch に渡す環境名
      allow_permissions: admin,maintain,write  # 許可する権限
      acknowledge_reaction: eyes               # 受理したことを示すリアクション
```

### カスタマイズ例
- `staging_command` を別のキーワードにすれば、組織で統一したコマンドに変更できます。
- `staging_env` で渡す環境名を変更すれば、呼び出し先のワークフロー入力に合わせられます。

## 動作確認の手順メモ
1. ダミー PR（例: base: `master`）に `/deploy-staging` コメントを投稿し、指定した `dispatch_workflow_id` が PR の head ブランチで `createWorkflowDispatch` されることを確認する。
2. ステージング以外のコマンド（例: `/deploy-production`）をコメントしてもスキップされることを確認する（必要なら `staging_command` を別のキーワードに変更して利用）。
