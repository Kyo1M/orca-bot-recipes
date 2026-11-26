# Git整理の定期実行

| 定期実行 | いつ | 何をする | 結果の出し先 |
|---|---|---|---|
| 毎週金曜の Git 整理 | 金曜 8:00 | worktree・ブランチ・PR を棚卸しし、消してよいものを消し、判断が要るものを質問にする | 新しいチャットで実行し、件数を Home チャットへ。質問はその週の item で聞く |

- 時刻はホストの時刻で動く。別のタイムゾーンで動かしたいときは `--timezone` を付ける。
- 週の終わりに片付けておくと、翌週の作業を少ない worktree で始められる。曜日は好みで変えてよい。

## 作り方

bot のチャットで「次のプロンプトで、毎週金曜 8:00 の定期実行を作って」と頼むと、bot が作る。自分で作るときは、下のプロンプトを置き換えてからファイル(例 `weekly-git.txt`)に保存し、次のコマンドを使う。

```sh
orca automations create --bot "Git整理" --name "毎週金曜の Git 整理" \
  --trigger "FREQ=WEEKLY;BYDAY=FR;BYHOUR=8;BYMINUTE=0" --runs-in new --results-to chat \
  --missed-run-grace-minutes 720 \
  --prompt "$(cat weekly-git.txt)"
```

## 毎週金曜の Git 整理

- いつ: `FREQ=WEEKLY;BYDAY=FR;BYHOUR=8;BYMINUTE=0`(金曜 8:00)
- GitHub のアカウントが 1 つなら、手順 1 の 2 つ目の検索を消す。2 つ以上なら、`<second-account>` をそのアカウント名にし、アカウントの数だけ並べる。

```text
毎週の Git 整理。SOUL.md の「How I work」に従う。

1. 対象範囲(SOUL.md の「対象範囲」):
   - ブランチと worktree: ~/Developer/ 配下(サブフォルダにまとめたリポジトリを含み、もう使わないフォルダ ~/Developer/_archive/ は除く)の全リポジトリと、その git worktree list に出る worktree すべて(~/.cursor・~/.codex・.claude/worktrees など ~/Developer の外にあるものを含む)。リポジトリを移したことで .git ファイルの指す先が壊れている worktree は、先に `git -C <リポジトリ> worktree repair <worktree のパス>` で直してから判定する。`orca worktree ps --since 7d` で 7 日以内に動いている worktree と、作業者が使っている Orca の作業用のフォルダ(例 ~/orca/workspaces)配下の worktree は消さない。
   - PR: GitHub 全体の自分の open PR を探す(`gh search prs --author @me --state open`。別のアカウントも使っているなら `GH_TOKEN=$(gh auth token --user <second-account>) gh search prs --author <second-account> --state open` も)。手元に無いリポジトリの PR も、閉じる・まとめるの質問の対象にする。ただし ~/Developer/_archive/ のフォルダの origin が指すリポジトリの PR は、もう使っていないので数えず、質問にもしない。
   棚卸しする: worktree(git worktree list、prunable)、ローカルブランチ(main にマージ済み・[gone]・未プッシュのコミット)、未コミットの変更、gh pr list の open PR(自分の PR とほかの人の PR を分ける)。worktree のブランチは gh pr list --head <branch> --state all で PR の状態を見る。gh は upstream など origin 以外のリモートがあるとそちらを見てしまうので、必ず -R <origin の owner/repo> を付ける(origin の URL から owner/repo を取る)。別のアカウントのリポジトリは、gh の前に GH_TOKEN=$(gh auth token --user <second-account>) を付けて読む。
2. 確認なしで消してよいもの(SOUL.md の定義)だけを消す: prunable な worktree の prune、PR がマージ済み、または中身がすべて origin/main に入っている(`git cherry origin/main <HEAD>` に + が無い)うえに未コミットの変更が無い worktree の git worktree remove(--force は使わない)、main にマージ済みのローカルブランチの git branch -d、[gone] で未プッシュのコミットが無いブランチの git branch -d。-D・force push・rm -rf は使わない。消したものはリポジトリごとに件数と名前を控える。
3. 判断が要るものは、選択式で答えられるようにする。今週分の item を 1 つ作り(`orca bot item add --title "Git整理 YYYY-MM-DD の確認" --done-means "すべての質問に答えが出て、その通りに整理した"`)、`orca bot item set <n> --status needs-you --comment "<今週消したものの件数と、確認が要るものの概要。5行以内>"` に、判断ごとに --ask と --option を付ける。
   - 1 つの --ask は 1 つの判断。対象を名指し、140字未満の1行で ? で終える(例「<リポジトリ> の open PR #3〜#12(9本)を1本の PR にまとめますか?」)。
   - 選択肢は 2〜4 個(例「まとめる」「このまま残す」「来週また聞く」、削除なら「消す」「残す」)。おすすめを --recommended で示す。
   - 同じ種類の候補が多いときは、リポジトリ単位にまとめて 1 問にする(「<リポジトリ> の PR の無い worktree 7個を消しますか?」)。細かい一覧はコメントか添付ファイルに書く。
   - 質問は重要な順に最大 5 問で、1 回の item set にまとめて付ける。残りは来週に回す(答えの無い質問は、次の item set で新しい質問に置き換わるため、2 回に分けて set しない)。
   - 前の週の item でまだ答えの無い質問があれば、新しい item に引き継いで古い item は cancel する。
   - ほかの人の PR は質問にしない(件数だけコメントに書く)。
4. 答えが item mail で来たら、その答えの通りに実行する(PR の一本化はワーカーに渡す。マージは答えで明示されたときだけ)。全問に答えが出て実行したら item を done にする。
5. チャットへの最後の返事は 3 行以内: 今週消した件数、確認待ちの質問数と item 番号。消したものも質問も無ければ NO_REPLY で終える。
```
