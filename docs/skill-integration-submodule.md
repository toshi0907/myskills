# myskills を他リポジトリに組み込む(submodule + symlink方式)

myskills のスキルを、他のリポジトリのClaude Codeから使えるようにするための
汎用的な組み込み方法。ローカルで使う場合・Claude Code on the web(cloud)で
使う場合のどちらにも共通する土台として、submodule + symlink 方式を使う。

## 方式: submodule + symlink(スキルごと)

- 対象リポジトリに myskills を **submodule として `.myskills` に配置**する。
- `.claude/skills` は**実ディレクトリのまま**にし、その配下に
  `.myskills/claude-skills/<スキル名>` への **symlinkをスキルごとに**登録する。
  (`.claude/skills` 自体をsymlinkにはしない)
- symlinkは通常のgit管理ファイルとして対象リポジトリにコミットされるため、
  一度設定すれば作り直す必要はない。
- myskills側の更新を取り込みたいときは `git submodule update --init --remote`
  を実行する(このコマンド自体はどの環境でも共通)。myskills側で新しいスキルが
  追加された場合は、それに対応するsymlinkを追加する処理も併せて必要になる
  (詳細は後述の同期スクリプトを参照)。

この構成にしておくと、「スキルをコピーして個別に同期する」独自ロジックが不要になり、
myskills側でスキルを追加した場合もsymlink越しに素直に反映される。また
`.claude/skills` が実ディレクトリのままなので、対象リポジトリ固有のローカルスキルを
通常のディレクトリとして共存させることもできる。

### ルール: 既存ファイルは上書きしない

`.claude/skills/<スキル名>` に**同名のファイル/ディレクトリ/symlinkが既に存在する場合は
上書きしない**。対象リポジトリ側で意図的に用意したローカルスキルや、myskills側の
スキルを個別にカスタマイズしたい場合を優先する。

## 設置手順(対象リポジトリ側、初回のみ)

1. myskills を submodule として追加する。

   ```bash
   git submodule add -b main https://github.com/toshi0907/myskills.git .myskills
   ```

2. `.claude/skills` 配下に、myskillsの各スキルへのsymlinkを(未作成のものだけ)張る。
   同期スクリプト([`../scripts/session-start-hook/myskills-skills-sync.sh`](../scripts/session-start-hook/myskills-skills-sync.sh))
   を使うと、この処理(と後述のsubmodule更新)をまとめて実行できる。

   ```bash
   mkdir -p .claude/skills
   for d in .myskills/claude-skills/*/; do
     name="$(basename "$d")"
     [ -e ".claude/skills/$name" ] || [ -L ".claude/skills/$name" ] || \
       ln -s "../../.myskills/claude-skills/$name" ".claude/skills/$name"
   done
   ```

3. 変更をコミットする(`.gitmodules`, `.myskills`, `.claude/skills/` 配下の各symlink)。

これでどの環境のClaude Codeからも、対象リポジトリをcloneして
submoduleを初期化すれば myskills のスキルが `.claude/skills/` 配下にスキルごとの
symlinkとして見える状態になる。

## 更新の取り込み方(環境によって異なる)

更新には「submoduleを最新化する」ことに加えて、「myskills側で新しく
追加されたスキルのsymlinkを `.claude/skills/` に追加する」ことの2つが必要になる。
後者は [`../scripts/session-start-hook/myskills-skills-sync.sh`](../scripts/session-start-hook/myskills-skills-sync.sh)
にまとめてあり、既存のsymlinkやローカルスキルを上書きしないため何度実行しても安全。

「いつ・誰が実行するか」は環境によって変わる。

### ローカル(永続環境)の場合

作業ディレクトリが消えないので、必要なときに手動で実行すればよい。

```bash
git submodule update --init --remote -- .myskills
bash .myskills/scripts/session-start-hook/myskills-skills-sync.sh
```

頻繁に更新を取り込みたい場合は、シェルのエイリアスや `direnv` 等で
リポジトリに入るたびに実行する運用でも十分。

### Claude Code on the web(使い捨て環境)の場合

セッションごとにコンテナがまっさらな状態から始まるため、手動実行に頼れない。
SessionStart hookで毎回自動的に submodule更新 + スキルsymlink追加を
実行させることで、都度最新化する。具体的なフックスクリプトと設置手順は
[`../scripts/session-start-hook/`](../scripts/session-start-hook/) を参照。

## 使う際のルール(CLAUDE.md)

同期処理そのものはフックやコマンドの責務なので、`CLAUDE.md` には
「同期されたスキルをどう使うか」という判断ルールだけを書く。
文面例は [`../scripts/session-start-hook/CLAUDE.snippet.md`](../scripts/session-start-hook/CLAUDE.snippet.md) を参照。

## 既知の制約・今後の検討事項

- myskills側でスキルの名前を変更・削除した場合、対象リポジトリに残った古いsymlinkは
  自動では消えない(既存ファイルを上書きしない方針のため、削除は現状手動対応)。
- claude.ai のプロジェクト(ブラウザ版)は本方式の対象外。SKILL.md本文を
  「プロジェクトの知識」に手動で貼る運用が別途必要。
