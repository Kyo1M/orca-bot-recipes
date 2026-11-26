# orca-bot-recipes

Orca の bot の汎用化した役割と仕事のしかた(Git の整理・議事録・skill の整理・タスク管理)

AI のエージェントで複数のプロジェクトを回していると、片付け・議事録・指示の整理・タスクの見渡しのような、毎週くり返す細かい仕事が溜まります。このリポジトリは、そうした仕事を Orca の bot に任せている事例を、自分の Orca で同じ bot を作れる形にまとめたものです。bot ごとに、役割(SOUL.md)・定期実行の例・前提と作り方を置いています。

## bot の一覧

| bot | 困りごと | 楽になること |
|---|---|---|
| [Git整理](bots/git-cleanup/) | エージェントの作業ごとに worktree・ブランチ・PR が溜まり、消してよいかの確認が後回しになる | 毎週、消してよいものは自動で消え、判断が要るものだけが選択式の質問で届く |
| [議事録作成](bots/meeting-minutes/) | 文字起こしを議事録にする前に、どのプロジェクトの会議かを見分けてリポジトリを開く手間がかかる | 文字起こしを渡すだけで、振り分けから議事録・台帳の更新まで済み、作り忘れも毎週知らせてもらえる |
| [skills・mods・bot整理](bots/skills-curator/) | skill・メモリー・mod・bot がプロジェクトごとに増え、同じ指示が散らばる | 毎週の整理案から採用するものを選ぶだけで、共通の skill と原則を一か所で育てられる |
| [タスク管理](bots/task-manager/) | 台帳がリポジトリごとに分かれ、今日なにから手を付けるかを決めるたびに見比べている | 毎朝、全プロジェクトの 1 週間の段取りが届き、台帳の状態も番号で答えるだけで直る |

## Orca の bot とは

[Orca](https://orca.dev)([GitHub](https://github.com/stablyai/orca))は、複数の AI のエージェントを worktree ごとに動かす開発環境です。bot は Orca の中に住む常駐のエージェントで、役割を書いた `SOUL.md` と自分のメモリーを持ち、チャットでの依頼と定期実行(automation)で仕事をします。

## 使い方

1. `bots/` から使いたい bot を選び、その `README.md` の前提(要るリポジトリ・skill・接続)をそろえます。
2. Orca で新しい bot を作ります。bot のフォルダは `~/.orca/bots/` の下にできます(`orca bot list` で場所を確かめられます)。
3. `SOUL.md` のパス(`~/Developer/<your-repo>` のような書き方のところ)を自分の環境に直し、bot のフォルダの `SOUL.md` を置き換えます。bot のチャットに貼って「SOUL.md をこの内容にして」と頼んでもかまいません。
4. `automations.md` の定期実行を足します。bot のチャットでプロンプトを渡して作ってもらうか、`orca automations create --bot <bot の名前>` で作ります。初回は `orca automations run` で 1 回動かし、結果を確かめてから定期に任せます。

SOUL.md は出発点です。使いながら、bot に「SOUL.md のここをこう直して」と頼んで自分の仕事に合わせていきます。

## ファイルの構成

```text
bots/
  git-cleanup/        Git整理
  meeting-minutes/    議事録作成
  skills-curator/     skills・mods・bot整理
  task-manager/       タスク管理
    hub-example/      タスクのハブのリポジトリの例(projects.yaml・priorities.md・task-ledger skill)
```

各 bot のフォルダには次の 3 つがあります。

| ファイル | 中身 |
|---|---|
| `SOUL.md` | bot の役割と仕事のしかた。bot のフォルダの `SOUL.md` に貼る |
| `automations.md` | 定期実行の例(いつ・何をする・結果をどこに出すか・プロンプト) |
| `README.md` | 前提と、Orca での作り方 |

## 関連する公開リポジトリ

| リポジトリ | 中身 | 使う bot |
|---|---|---|
| [Kyo1M/skills](https://github.com/Kyo1M/skills) | Claude Code と Codex で使う skill(議事録の meeting-minutes など) | 議事録作成、skills・mods・bot整理 |
| [Kyo1M/claude-mods](https://github.com/Kyo1M/claude-mods) | Claude Code の mod(hooks プラグイン) | skills・mods・bot整理 |
| [Kyo1M/project-context-template](https://github.com/Kyo1M/project-context-template) | 議事録・決定・タスク台帳をリポジトリに集めるためのテンプレート | 議事録作成、タスク管理 |
| [Kyo1M/claude-worklog](https://github.com/Kyo1M/claude-worklog) | Claude Code と Codex のログから、プロジェクトごとの稼働時間と作業内容の素材を出す CLI | タスク管理 |

## ライセンス

[MIT License](LICENSE) です。
