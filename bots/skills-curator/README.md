# skills・mods・bot整理

複数のプロジェクトで AI のエージェントを使っていて、skill・メモリー・mod・bot がプロジェクトごとに増え、同じ指示があちこちに散らばって直すたびに探して回っている人向けの bot です。毎週の点検で、どのプロジェクトでも効くものを共通の置き場に集める整理案が届きます。採用する提案を選ぶだけで、共通の skill と原則を一か所で育てられます。

## 前提

- **集める先のリポジトリ**: skill と mod の正本を、それぞれ 1 つのリポジトリにまとめていること。この bot は公開の次の 2 つを例にしています。自分のリポジトリに置き換えて使います。
  - [Kyo1M/skills](https://github.com/Kyo1M/skills): skill の正本。`~/.claude/skills/` と `~/.agents/skills/` に symlink で配り、`scripts/link-skills.sh --check` で壊れたリンクと同名の別実体を点検します。
  - [Kyo1M/claude-mods](https://github.com/Kyo1M/claude-mods): Claude Code の mod(hooks プラグイン)の正本。リポジトリ全体がプラグインのマーケットプレイスです。
- **bot の写しのリポジトリ(任意)**: 別の Mac で同じ bot を使いたいときに、bot のフォルダの中身を写す非公開のリポジトリです。MEMORY.md や memory/ も写すので、必ず非公開にします。手元の bot を書き出す手順(スクリプトなど)は、そのリポジトリに用意します。使わないなら、SOUL.md と定期実行のプロンプトから写しの記述を消します。
- **グローバルの CLAUDE.md**: どのプロジェクトでも効く指示を集める先です。
- **Orca の item と worker**: 提案は item の選択式の質問で届き、採用した提案は bot が worker に渡して PR を作らせます。worker が作業できるよう、skill と mod の正本(と bot の写し)のリポジトリを `orca repo add --path <パス>` で Orca に登録しておきます。

## Orca での作り方

1. **bot を作る**: Orca で新しい bot を作り、名前を「skills・mods・bot整理」にします。bot のフォルダは `~/.orca/bots/` の下にできます。フォルダの名前は bot の名前と同じとは限らないので、`grep -l '"name": "skills・mods・bot整理"' ~/.orca/bots/*/bot.json` で探します。
2. **SOUL.md を貼る**: [SOUL.md](SOUL.md) の「集める先」の表を自分の置き場に書き換え、bot のフォルダの `SOUL.md` を置き換えます。bot のチャットに貼って「SOUL.md をこの内容にして」と頼んでもかまいません。
3. **定期実行を足す**: [automations.md](automations.md) の「skill・メモリー・mod・bot 週次点検」を、置き場を書き換えてから足します。

## この bot がしないこと

- 自分で skill やメモリーを書き換えること。中身の整理は、採用された提案だけを worker が PR にします。
- PR のマージ。マージはオーナーが行います。
- PR・ブランチ・worktree の片付け。こちらは [Git整理](../git-cleanup/) の bot が受け持ちます。
