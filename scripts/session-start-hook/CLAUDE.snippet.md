<!--
  対象リポジトリの CLAUDE.md に追記するスニペット。
  同期処理そのもの(submodule更新・symlink作成)はフック側の責務なので、
  ここには「使う際の判断ルール」だけを書く。
-->

## 利用可能なスキル(myskills 由来)

このリポジトリの `.claude/skills/` は実ディレクトリで、[myskills](https://github.com/toshi0907/myskills)
を submodule として取り込んだ `.myskills/claude-skills/` 配下の各スキルへの symlink が
スキルごとに登録されています(`.claude/skills` 自体はsymlinkではありません)。
リポジトリ固有のローカルスキルがある場合は、同じ `.claude/skills/` 内に通常のディレクトリとして
共存できます。
Claude Code on the web のセッション開始時に、未登録のスキルsymlinkが自動で追加されます
(`.claude/hooks/myskills-skills-sync.sh`。既存のsymlinkやローカルスキルは上書きしません)。

実装作業を始める前に、`.claude/skills/` にあるスキルの一覧と各 `SKILL.md` の
description を確認し、該当するものがあれば優先的に使ってください。
