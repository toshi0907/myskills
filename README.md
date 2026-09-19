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
| android-mobile-only-dev | Android Studio/PCを使わず、スマホ+Claude CodeだけでAndroidアプリを開発する際の開発フロー・体制に関するSkill。ローカルでビルドが完結しない前提、CIへのビルド委譲、デバッグ署名鍵の固定、ビルド番号を使ったアプリ内アップデート配布、実機確認までの開発ループを扱う | `claude-skills/android-mobile-only-dev/` |
| android-app-dev-foundations | Kotlin + Jetpack Compose + GradleでAndroidアプリを開発する際の汎用的な技術知見。Compose Compilerプラグイン要件、RoomのMigration運用、WorkManagerの短間隔実行パターン、並行処理の直列化・キャンセル安全性、位置情報(Geofencing/継続的追跡)利用時の注意点を扱う | `claude-skills/android-app-dev-foundations/` |

## 使い方(環境別)

- **claude.aiのプロジェクト**: 該当するSKILL.mdの中身を「プロジェクトの知識」に貼り付ける
- **他リポジトリのClaude Code(ローカル/web共通)**: myskillsをsubmoduleとして取り込み、`.claude/skills` 配下にスキルごとのsymlinkを登録する(既存の同名ファイルは上書きしないため、対象リポジトリ固有のローカルスキルとも共存できる)。Claude Code on the webなど使い捨て環境ではSessionStart hookで自動更新する。手順は [`docs/skill-integration-submodule.md`](docs/skill-integration-submodule.md) を参照
- **myskills自身のClaude Code**: submoduleを介す必要がないため、`.claude/skills` を直接 `claude-skills/` へのsymlinkにしてある(このリポジトリ内で作業するとき自動的にスキルが読み込まれる)
- **新しい端末**: `myskills` をcloneするだけで全スキルにアクセス可能

## 更新ルール

新しいスキルを追加したら、上記の表に1行追記すること。