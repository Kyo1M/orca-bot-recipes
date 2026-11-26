# 議事録作成

いくつものプロジェクトの会議に出ていて、文字起こしを議事録にする前に、どのプロジェクトの会議かを見分けてリポジトリを開く手間がかかっている人向けの bot です。文字起こしをチャットに渡すだけで、プロジェクトの振り分けから議事録の作成、台帳・wiki の更新までを引き受けます。議事録になっていない文字起こしも毎週知らせてもらえるので、作り忘れが残りません。

議事録の作り方そのものは、公開の [Kyo1M/skills](https://github.com/Kyo1M/skills) の [meeting-minutes](https://github.com/Kyo1M/skills/tree/main/meeting-minutes) skill に任せます。この bot が受け持つのは、どのリポジトリで meeting-minutes を動かすかの判断と、作り忘れの見張りです。

## 前提

- **meeting-minutes skill**: [Kyo1M/skills](https://github.com/Kyo1M/skills) を手元に clone し、`meeting-minutes/` を Claude Code の `~/.claude/skills/meeting-minutes` に置きます(リポジトリの README の symlink の手順を使えます)。
- **各プロジェクトのリポジトリ**: 議事録の置き場・命名・型を CLAUDE.md / AGENTS.md か運用ガイドに書いてあること。meeting-minutes はそこから決まりを読みます。[project-context-template](https://github.com/Kyo1M/project-context-template) から作ったリポジトリなら、議事録の置き場とタスク台帳の決まりが最初から入っています。
- **文字起こしの置き場**: 定期チェックは `~/Downloads` の `.txt` と Teams の会議名の `.docx` を見ます。別の置き場や形式なら、SOUL.md と定期実行のプロンプトを書き換えます。
- **対応表 `projects.md`**: bot のフォルダに、プロジェクトとリポジトリの対応表を置きます(自分用。リポジトリには入れません)。

```markdown
# プロジェクトの対応表

| プロジェクト | リポジトリ | 議事録の置き場 | 見分ける手がかり |
|---|---|---|---|
| Web ショップの改修 | ~/Developer/webshop-renewal | docs/minutes/ | 取引先の会社名、「決済」「商品ページ」 |
| 調査の受託 | ~/Developer/research-team | docs/minutes/ | 定例の会議名、「アンケート」「集計」 |
```

## Orca での作り方

1. **bot を作る**: Orca で新しい bot を作り、名前を「議事録作成」にします。bot のフォルダは `~/.orca/bots/` の下にでき、`orca bot list` で場所を確かめられます。
2. **SOUL.md を貼る**: [SOUL.md](SOUL.md) で bot のフォルダの `SOUL.md` を置き換えます。bot のチャットに貼って「SOUL.md をこの内容にして」と頼んでもかまいません。続けて、上の型で `projects.md` を bot のフォルダに作ります。
3. **定期実行を足す**: [automations.md](automations.md) の「週1回の未処理の文字起こしチェック」を足します。

## 使い方

- 会議のあと、文字起こしのファイルをチャットに渡すか「Downloads の今日の文字起こしを議事録にして」と頼みます。
- bot がプロジェクトを決め、そのリポジトリで meeting-minutes の流れを動かします。保存前の確認はチャットで聞かれます。
- 終わると、作ったファイルの絶対パスと更新した範囲がチャットに返ります。
