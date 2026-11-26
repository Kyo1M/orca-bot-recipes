# skills・mods・bot整理の定期実行

| 定期実行 | いつ | 何をする | 結果の出し先 |
|---|---|---|---|
| skill・メモリー・mod・bot 週次点検 | 月曜 9:00 | skill・メモリー・mod・bot を点検し、整理の提案を最大 5 件出す。bot の写しの差分があれば PR を作る | bot の Home チャットで続けて実行し、要約をそこへ。提案はその週の item で聞く |

- 点検は 1 台の Mac だけで動かす。複数の Mac で同じ bot を使っていても、定期実行は 1 台にだけ作る。
- Home チャットで続けて実行する(`--runs-in chat`)と、前の週の提案と答えを踏まえて点検できる。
- 時刻はホストの時刻で動く。別のタイムゾーンで動かしたいときは `--timezone` を付ける。
- プロンプトの置き場(`~/Developer/Skills`・`~/Developer/claude-mods`・`<your-bots-repo>`・`<bot のフォルダ>`)は、SOUL.md の「集める先」と同じものに置き換える。bot の写しを使わないなら、「bot の写しの PR」の節と、見る場所の写しの行を消す。

## 作り方

bot のチャットで「次のプロンプトで、毎週月曜 9:00 の定期実行をこのチャットで作って」と頼むと、bot が作る。自分で作るときは次のコマンドを使う。

```sh
orca automations create --bot "skills・mods・bot整理" --name "skill・メモリー・mod・bot 週次点検" \
  --trigger "FREQ=WEEKLY;BYDAY=MO;BYHOUR=9;BYMINUTE=0" --runs-in chat --results-to chat \
  --missed-run-grace-minutes 720 \
  --prompt "$(cat weekly-review.txt)"
```

## skill・メモリー・mod・bot 週次点検

- いつ: `FREQ=WEEKLY;BYDAY=MO;BYHOUR=9;BYMINUTE=0`(月曜 9:00)

```text
週次の skill・メモリー・mod・bot の点検。オーナーが複数のプロジェクトで使う skill・メモリー・mod・bot の現状を確かめ、整理と改善の提案を出す。ゴールは、どのプロジェクトでも効くものを skill の正本(~/Developer/Skills、公開)・mod の正本(~/Developer/claude-mods、公開)・bot の写し(~/Developer/<your-bots-repo>、非公開)とグローバル CLAUDE.md に集め、プロジェクトにはそのプロジェクトでしか成り立たないものだけを残すこと。中身の整理は自分では直さず、オーナーの判断を待つ。bot の写しの更新(下の「bot の写しの PR」)だけは判断を待たずに PR まで作る。

見る場所:
- skill
  - ~/Developer/Skills/:README の管理表、各 SKILL.md、書き方の原則のファイル、symlink の点検(scripts/link-skills.sh --check)の結果、ブランチ(main か)、open な PR、直近 7 日の git log
  - ~/Developer/ 配下(もう使わないフォルダと各リポジトリの archive/ を除く)のプロジェクトの skill:.agents/skills/、.claude/skills/ の実体、.claude/commands/ など SKILL.md があるところすべて
- メモリー
  - ~/.claude/projects/*/memory/(プロジェクトごと。リポジトリを移す前の古いパスの名前のものも含む)
  - ~/.claude/CLAUDE.md とそこが読み込むファイル、各リポジトリの CLAUDE.md・AGENTS.md
  - ~/.codex/AGENTS.md と ~/.codex/memories/
- mod
  - ~/Developer/claude-mods/:手元で入れている plugin(~/.claude/plugins/installed_plugins.json)・README の一覧・.claude-plugin/marketplace.json の一致、mod どうしの重なり、open な PR
- bot
  - ~/.orca/bots/ の各 bot の SOUL.md・MEMORY.md・memory/・skills/:bot どうしの役割の重なり、bot の memory と skill・CLAUDE.md の重複、どの bot にもつながっていないフォルダ(bot.json の id が orca bot list に無いもの)
  - ~/Developer/<your-bots-repo>/:手元の bot との差分
- 使われ方:~/.claude/projects の直近 7 日の会話ログで、skill が呼ばれた場面と、呼ばれず手作業になった場面(読める範囲で)

確かめること:
- 同じ役割の skill がプロジェクトと skill の正本に別実装で重複していないか。共通に寄せる場合、違いを AGENTS.md・references のどこに移すか
- 複数のプロジェクトのメモリーに同じ指示が重複していないか。1 つのプロジェクトにしか無いが汎用な feedback はどれか。振り分けは A(重複していて共通に移す)・B(1 つにしか無いが共通の skill に入れる)・C(プロジェクトに残す)
- 指示どうしの食い違い(メモリーどうし、メモリーと原則、メモリーと skill)、既に共通側に入っていて消してよいメモリー
- 壊れたリンク・同名の別実体・README の管理表との食い違い・SKILL.md への原則の書き写し・古いパスや決定と矛盾する記述・長く止まっている skill や PR
- 書き方の原則を正本と要約に分けて持っているなら、その反映漏れ
- 減らす整理:内容が足され続けて膨らむので、毎回、重複している記述・同じことを別の言い方で書いた原則・1 つの上位の原則にまとめられる個別の原則・使われていない手順や skill を探し、統合・抽象化・削除で文書を減らす案を出す。提案のうち少なくとも 1 件は減らす案にする。足す提案には、代わりに減らせる記述を添える
- 規模の推移:skill ごとの SKILL.md と references の行数・文字数、書き方の原則の件数と正本の文字数を測り、~/.orca/bots/<bot のフォルダ>/sizes.md に日付つきで 1 行ずつ足す。前回から増えたものを現状に書く
- 公開の置き場(skill と mod の正本)に案件名・人名・案件固有の語が入っていないか。共通に移す提案では、それらを外して一般化する
- 前回の点検 item で見送られた提案は、状況が変わらない限り出し直さない。採用済みで未反映のものは進み具合を書く

bot の写しの PR:
- 点検の最初に、その週の点検 item(下の「出力」の item)を作っておき、写しのリポジトリの worker(orca orchestration worker-start --item <その item> --worktree new-child --repo name:<your-bots-repo> --agent claude)を 1 つ始める。自分では写しのリポジトリのファイルを書かない。
- worker に頼むこと:main を pull → 写しのリポジトリの書き出しの手順(スクリプトなど)で手元の bot を書き出す → 差分が無ければ何も push せず「差分なし」と報告。差分があれば秘密(トークン・鍵・パスワード・API キー)を走査し、ブランチ bot-sync で commit・push して PR「bot の写しの更新」を作る。同じブランチの PR が開いていれば新しく作らずそこへ push して更新する。秘密が見つかったら写さず場所だけ報告する。マージはしない。
- worker の結果は点検 item に載せ、PR があればリンクを現状に書く。

出力:
- 詳しい点検結果(現状、メモリーの A・B・C の振り分け表、提案)を md にまとめ、`orca bot item add` で「skills・メモリー点検 YYYY-MM-DD」を 1 件作り、その md をコメントに付ける。--status needs-you で提案ごとに --ask と --option(採用 / 見送り / 修正して採用)を付ける。提案は重要な順に最大 5 件で、残りは「そのほか」に畳む。
- 最後のメッセージはこのチャット向けに、現状 3 行と提案の一覧(各 1 行)、bot の写しの PR があればそのリンクだけ書く。問題も提案もなければ NO_REPLY。
オーナーが採用した提案は、skill の正本のリポジトリ(またはメモリーやプロジェクトの側)を直す worker(orca orchestration worker-start --item <n> --worktree new-child --repo <対象> --agent claude)に渡して PR を作らせる。メモリーの削除・移動は、移した先が PR でマージされてから行う。
```
