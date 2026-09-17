# myskills

汎用的に使えるスキル・スクリプト・ノウハウをまとめて管理するリポジトリ。

## 構成

- `claude-skills/` : Claude Skills形式(SKILL.md準拠)のスキル集。Claude / Claude Codeにそのまま読み込ませられる
- `scripts/` : 汎用スクリプト・テンプレート類
- `docs/` : スキル化しないノウハウ・チェックリスト

## スキル一覧

| スキル名 | 概要 | パス |
|---|---|---|
| github-dev-flow | GitHubリポジトリでの実装依頼を進める際、着手前に自動化の範囲(調査のみ〜マージまで5段階)を確定させ、依頼→作業→PR作成→CI対応→セルフレビュー→マージのフローを一貫して実行する | `claude-skills/github-dev-flow/` |
| task-completion-check | タスクを依頼された際に使う共通Skill。タスク完了(GitHub作業ならPRマージ+マージ先CI完了)を判定し、完了時にセッションをアーカイブするかユーザーに確認する | `claude-skills/task-completion-check/` |

## 使い方(環境別)

- **claude.aiのプロジェクト**: 該当するSKILL.mdの中身を「プロジェクトの知識」に貼り付ける
- **他リポジトリのClaude Code(ローカル/web共通)**: myskillsをsubmoduleとして取り込み、`.claude/skills` をsymlinkにする。Claude Code on the webなど使い捨て環境ではSessionStart hookで自動更新する。手順は [`docs/skill-integration-submodule.md`](docs/skill-integration-submodule.md) を参照
- **新しい端末**: `myskills` をcloneするだけで全スキルにアクセス可能

## 更新ルール

新しいスキルを追加したら、上記の表に1行追記すること。