# myskills を他リポジトリに組み込む(submodule + symlink方式)

myskills のスキルを、他のリポジトリのClaude Codeから使えるようにするための
汎用的な組み込み方法。ローカルで使う場合・Claude Code on the web(cloud)で
使う場合のどちらにも共通する土台として、submodule + symlink 方式を使う。

## 方式: submodule + symlink

- 対象リポジトリに myskills を **submodule として `.myskills` に配置**する。
- `.claude/skills` を、その中の `claude-skills/` への **symlink** にする。
- symlinkは通常のgit管理ファイルとして対象リポジトリにコミットされるため、
  一度設定すれば作り直す必要はない。
- myskills側の更新を取り込みたいときは `git submodule update --init --remote`
  を実行する(このコマンド自体はどの環境でも共通)。

この構成にしておくと、「スキルをコピーして個別に同期する」独自ロジックが不要になり、
myskills側でスキルを削除・追加した場合もsymlink越しに素直に反映される。

### 割り切っている点

`.claude/skills` を丸ごとsymlinkにするため、このディレクトリは実質
100% myskills由来のスキルになる。対象リポジトリ固有のローカルスキルを
`.claude/skills` に混在させることはできない(必要になった場合は、
個別スキルディレクトリ単位のsymlinkに切り替えるなど別方式を検討する)。

## 設置手順(対象リポジトリ側、初回のみ)

1. myskills を submodule として追加する。

   ```bash
   git submodule add -b main https://github.com/toshi0907/myskills.git .myskills
   ```

2. `.claude/skills` を `.myskills/claude-skills` への symlink にする。
   既存の `.claude/skills` が実ディレクトリの場合は退避してから置き換えること。

   ```bash
   mkdir -p .claude
   rm -rf .claude/skills   # 既存の中身がある場合は退避してから
   ln -s ../.myskills/claude-skills .claude/skills
   ```

3. 変更をコミットする(`.gitmodules`, `.myskills`, `.claude/skills` のsymlink)。

これでどの環境のClaude Codeからも、対象リポジトリをcloneして
submoduleを初期化すれば myskills のスキルが `.claude/skills/` に見える状態になる。

## 更新の取り込み方(環境によって異なる)

submoduleの更新自体は共通コマンドだが、「いつ・誰が実行するか」は環境によって変わる。

### ローカル(永続環境)の場合

作業ディレクトリが消えないので、必要なときに手動で実行すればよい。

```bash
git submodule update --init --remote -- .myskills
```

頻繁に更新を取り込みたい場合は、シェルのエイリアスや `direnv` 等で
リポジトリに入るたびに実行する運用でも十分。

### Claude Code on the web(使い捨て環境)の場合

セッションごとにコンテナがまっさらな状態から始まるため、手動実行に頼れない。
SessionStart hookで毎回自動的に `git submodule update --init --remote` を
実行させることで、都度最新化する。具体的なフックスクリプトと設置手順は
[`../scripts/session-start-hook/`](../scripts/session-start-hook/) を参照。

## 使う際のルール(CLAUDE.md)

同期処理そのものはフックやコマンドの責務なので、`CLAUDE.md` には
「同期されたスキルをどう使うか」という判断ルールだけを書く。
文面例は [`../scripts/session-start-hook/CLAUDE.snippet.md`](../scripts/session-start-hook/CLAUDE.snippet.md) を参照。

## 既知の制約・今後の検討事項

- `.claude/skills` を丸ごとsymlinkにする都合上、対象リポジトリ固有のローカルスキルとは
  共存できない(前述)。
- claude.ai のプロジェクト(ブラウザ版)は本方式の対象外。SKILL.md本文を
  「プロジェクトの知識」に手動で貼る運用が別途必要。
