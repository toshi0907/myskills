<!--
  対象リポジトリの CLAUDE.md に追記するスニペット。
  同期処理そのもの(submodule更新)はフック側の責務なので、
  ここには「使う際の判断ルール」だけを書く。
-->

## 利用可能なスキル(myskills 由来)

このリポジトリの `.claude/skills/` は、[myskills](https://github.com/toshi0907/myskills)
リポジトリを submodule として取り込んだ `.myskills/claude-skills/` への symlink です。
Claude Code on the web のセッション開始時に自動で最新化されます
(`.claude/hooks/myskills-submodule-sync.sh`)。

実装作業を始める前に、`.claude/skills/` にあるスキルの一覧と各 `SKILL.md` の
description を確認し、該当するものがあれば優先的に使ってください。
