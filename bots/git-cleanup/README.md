# Git整理

AI のエージェントに作業を任せるたびに worktree・ブランチ・PR が増え、どれを消してよいかを確かめる手間で片付けが後回しになっている人向けの bot です。毎週、消してよいものは確認なしで消え、判断が要るものだけが選択式の質問で届きます。質問に答えるだけで、溜まった PR・ブランチ・worktree を片付けられます。

## 前提

- **リポジトリの置き場**: 手元のリポジトリが 1 つのフォルダの下(例 `~/Developer/`)にまとまっていること。別の置き場を使うときは、SOUL.md と定期実行のプロンプトの `~/Developer/` を書き換えます。もう使わないリポジトリを `~/Developer/_archive/` のようなフォルダに分けておくと、そこを対象から外せます。
- **GitHub CLI(`gh`)**: 自分の PR を探すのに使います。`gh auth login` を済ませておきます。仕事用と個人用など複数のアカウントを使うときは、両方を `gh auth login` しておき、定期実行のプロンプトの `<second-account>` を書き換えます。
- **Orca の item**: 判断が要るものは、その週の item の選択式の質問(Needs you)で届きます。

## Orca での作り方

1. **bot を作る**: Orca で新しい bot を作り、名前を「Git整理」にします。bot のフォルダは `~/.orca/bots/` の下にでき、`orca bot list` で場所を確かめられます。
2. **SOUL.md を貼る**: [SOUL.md](SOUL.md) の置き場(`~/Developer/`・`_archive/`)を自分の環境に合わせて直し、bot のフォルダの `SOUL.md` を置き換えます。bot のチャットに貼って「SOUL.md をこの内容にして」と頼んでもかまいません。
3. **定期実行を足す**: [automations.md](automations.md) の「毎週金曜の Git 整理」を足します。初回は `orca automations run` で動かし、消したものと質問の中身を確かめてから毎週に任せると安心です。

## 消すもの・聞くもの

| 扱い | 対象 |
|---|---|
| 確認なしで消す | prunable な worktree、PR がマージ済みか中身がすべて main に入っていて未コミットの変更が無い worktree、main にマージ済みのローカルブランチ、リモートが削除済みで未プッシュのコミットが無いブランチ |
| 聞いてから行う | 未コミットの変更・未プッシュのコミットが残るもの、PR が無い・open のブランチや worktree、PR の一本化・クローズ・マージ |
| 触らない | 7 日以内に動いている worktree、作業中の `~/orca/workspaces` 配下の worktree、他の人の PR |

消すときも `git branch -D`・`git worktree remove --force`・force push・`rm -rf` は使いません。
