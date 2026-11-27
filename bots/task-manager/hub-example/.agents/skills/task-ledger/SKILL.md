---
name: task-ledger
description: >-
  ハブのリポジトリの projects.yaml に登録した各リポジトリのタスク台帳(tasks/index.md)を読み、自分の担当のタスクを優先度と期限の順に 1 つの表にして会話に出す。優先度(高・中・低)の相談と priorities.md の書き換え・片付け、期限切れのタスクの棚卸し(write: ledger のプロジェクトは台帳を直し、write: handoff のプロジェクトは引き継ぎ文を出す)も行う。ハブのリポジトリで「今週やることは？」「タスク一覧を出して」「優先度を見直したい」「webshop:T-12 を高にして」「片付けて」「期限切れを整理したい」「棚卸しして」「webshop:T-3 を完了にして」「/task-ledger」などのリクエスト時に使う。
---

# task-ledger

状態の正本は各リポジトリの台帳で、ハブのリポジトリは `projects.yaml`(登録簿)と `priorities.md`(優先度)だけを持つ。一覧は毎回台帳から組み立てて会話に出し、ファイルには残さない。各リポジトリの台帳を書き換えるのは、4 の棚卸しで `write: ledger` のプロジェクトに書くときだけ(タスクの追加とタスク名の変更はしない。見送りの「（見送り）」だけは付ける)。

台帳は [project-context-template](https://github.com/Kyo1M/project-context-template) の `tasks/index.md` と同じ形を前提にする。「進行中」と「完了」の 2 つの節があり、列は次のとおり。

- 進行中: `| ID | タスク | 担当 | 期限 | 親 | 出典 | 詳細 | 状態 |`
- 完了: `| ID | タスク | 担当 | 完了日 | 親 | 出典 | 詳細 |`

## projects.yaml の書き方

```yaml
owner: 山田            # 台帳の「担当」の列に書く自分の名前。この名前が入っている行を対象にする
projects:
  - key: webshop       # 参照(webshop:T-12)に使う短い名前
    repo: ~/Developer/webshop-renewal
    ledger: tasks/index.md
    read: worktree     # 手元の作業ツリーの台帳を読む
    write: ledger      # 棚卸しで台帳に書く
  - key: research
    repo: ~/Developer/research-team
    ledger: tasks/index.md
    read: origin/main  # fetch してから origin/main の台帳を読む(台帳を PR で直す共同のリポジトリ向け)
    write: handoff     # 台帳に書かず、そのリポジトリのセッションへの引き継ぎ文を出す
```

- `read` は `worktree` か `origin/main`。共同で使うリポジトリで、手元の作業ツリーが main と離れやすいときは `origin/main` にする。
- `write` は `ledger` か `handoff`。台帳を PR で直す決まりのリポジトリや、Issue から台帳に写す決まりのリポジトリは `handoff` にする。書いていなければ `handoff` として扱う。

## 1. 一覧

「今週やることは？」「タスク一覧を出して」で行う。一覧ではファイルを書き換えない。

### 1-1. 台帳を読む

1. `projects.yaml` を読む。`repo` の `~` はホームに展開する。
2. `read: origin/main` のプロジェクトは、最新を取ってから一時ファイルに書き出す。

```bash
git -C <repo> fetch origin
rm -f "${TMPDIR:-/tmp}/task-ledger-<key>.md"
git -C <repo> show origin/main:<ledger> > "${TMPDIR:-/tmp}/task-ledger-<key>.md" || rm -f "${TMPDIR:-/tmp}/task-ledger-<key>.md"
```

- `fetch` に失敗したときは、手元にある `origin/main` をそのまま読む。最後に取得した日時を `stat -f '%Sm' -t '%Y-%m-%d %H:%M' "$(git -C <repo> rev-parse --absolute-git-dir)/FETCH_HEAD"`(macOS の書き方)で調べ、一覧の冒頭に添える。
- リポジトリのパスが無い・`git show` に失敗したときも、そのプロジェクトを次の読み取りに渡す(一時ファイルが無いので `SKIP` になる)。飛ばした理由には git のエラーを書く。

3. `read: worktree` のプロジェクトは `<repo>/<ledger>` をそのまま読む。
4. 先頭に `projects.yaml` の `owner` を置き、続けて全プロジェクトについて次の 3 つを登録簿の順に並べ、下の読み取りを 1 回で実行する(例は上の `projects.yaml` のとき)。
   - key
   - 台帳のパス(`read: origin/main` は手順 2 の一時ファイル)
   - リンクの基準: `read: worktree` は台帳のあるディレクトリの絶対パス。`read: origin/main` は `git -C <repo> remote get-url origin` の `<owner>/<name>` から作る `https://github.com/<owner>/<name>/blob/main/<台帳のディレクトリ>`

台帳は表として列を分けて読む(`awk` で `\|` を区切りにすると macOS では列を正しく取れない)。

```bash
python3 - 山田 \
  webshop ~/Developer/webshop-renewal/tasks/index.md ~/Developer/webshop-renewal/tasks \
  research "${TMPDIR:-/tmp}/task-ledger-research.md" https://github.com/<github-owner>/research-team/blob/main/tasks <<'EOF'
import datetime, os, posixpath, re, sys
EXPECT = ['ID', 'タスク', '担当', '期限', '親', '出典', '詳細', '状態']
LINK = re.compile(r'\]\(([^)\s]+)\)')

def resolve(text, base):
    def fix(m):
        url = m.group(1)
        if re.match(r'(https?:|mailto:|#|/)', url):
            return m.group(0)
        if base.startswith('http'):
            head, sep, rest = base.partition('/blob/main/')
            return '](' + head + sep + posixpath.normpath(posixpath.join(rest, url)) + ')'
        return '](' + posixpath.normpath(posixpath.join(base, url)) + ')'
    return LINK.sub(fix, text)

owner, args = sys.argv[1], sys.argv[2:]
today = datetime.date.today()
monday = today - datetime.timedelta(days=today.weekday())
print('TODAY', today, 'WEEK', monday, monday + datetime.timedelta(days=6), sep='\t')
for key, path, base in zip(args[0::3], args[1::3], args[2::3]):
    if not os.path.isfile(path) or os.path.getsize(path) == 0:
        print('SKIP', key, '台帳が見つからないか空: ' + path, sep='\t'); continue
    try:
        lines = open(path, encoding='utf-8').read().splitlines()
    except (OSError, UnicodeDecodeError) as e:
        print('SKIP', key, '台帳を読めない: ' + type(e).__name__, sep='\t'); continue
    sec, hdr, rows, undecided, open_rows, done_ids, bad = None, {}, [], 0, [], [], None
    for line in lines:
        if line.startswith('## '):
            sec = line[3:].strip(); continue
        s = line.strip()
        if sec not in ('進行中', '完了') or not s.startswith('|') or set(s) <= set('|-: '):
            continue
        cells = [c.strip() for c in re.split(r'(?<!\\)\|', s)[1:-1]]
        if sec not in hdr:
            hdr[sec] = cells; continue
        if sec == '完了':
            done_ids.append(cells[0]); continue
        if len(cells) != len(EXPECT):
            bad = bad or cells[0]; continue
        open_rows.append(cells)
        if owner in [n.strip() for n in cells[2].split('・')]:
            rows.append(cells)
        elif cells[2] == '未定':
            undecided += 1
    if hdr.get('進行中') != EXPECT:
        print('SKIP', key, '「進行中」の節が無いか、表の列が想定と異なる', sep='\t'); continue
    if bad:
        print('SKIP', key, '列の数が異なる行がある: ' + bad, sep='\t'); continue
    for c in rows:
        print('ROW', key, c[0], resolve(c[1], base), c[2], c[3], c[4], resolve(c[5], base), c[7], resolve(c[6], base), sep='\t')
    print('UNDECIDED', key, undecided, sep='\t')
    for c in open_rows:
        print('OPEN', key, c[0], c[1], c[2], c[4], sep='\t')
    if hdr.get('完了') != ['ID', 'タスク', '担当', '完了日', '親', '出典', '詳細']:
        print('NODONE', key, '「完了」の節が無いか、表の列が想定と異なる', sep='\t')
    else:
        print('DONE', key, *done_ids, sep='\t')
EOF
```

出力の行の種類:

| 先頭 | 中身 |
|---|---|
| `TODAY` | 今日の日付と、今週の月曜・日曜 |
| `ROW` | 「進行中」の節で、担当を `・` で区切った中に `owner` がある行(共同担当を含む)。列は key・ID・タスク・担当・期限・親・出典・状態・詳細。相対リンクはリンクの基準からのパスに直してある |
| `UNDECIDED` | 担当が「未定」の行の件数 |
| `OPEN` | 「進行中」の節の全行の key・ID・タスク・担当・親。片付けの候補・個別の指定・棚卸しで子のタスクを探すときに使う |
| `DONE` | 「完了」の節の ID の全件。片付けの候補のときに使う |
| `NODONE` | 「完了」の節が読めない。そのプロジェクトの片付けの候補は出さない |
| `SKIP` | 飛ばしたプロジェクトと理由 |

### 1-2. 優先度を付けて並べる

1. `priorities.md` を読み、参照(`<key>:<ID>`)で `ROW` と突き合わせる。行の無いタスクは「未設定」。
2. 優先度(高 → 中 → 低 → 未設定)の順に並べ、同じ優先度の中は期限の早い順にする。期限の無いものは各優先度の最後に置く。優先度と期限が同じものは、登録簿の順(`projects.yaml` の並び)、台帳の行の順に置く。
3. 期限の列は自由記述なので、次のように読む。表には台帳の書き方のまま出す。

| 書き方 | 期限とみなす日 |
|---|---|
| `2026-12-15` | その日付 |
| `2026-12 末`・`10 月前半`・`10 月中旬`・`11 月頭`・`10 月第 1 週`・`今週〜来週頭(12/1〜12/3 頃)` のような期間 | 期間の最後の日。`前半` は 15 日、`中旬` は 20 日、`頭` は 5 日、`第 n 週` は 7×n 日とみなし、括弧で日付が添えてあればその最後の日を使う |
| `未定`・空欄・`T-2 の後` のような他のタスク待ち・括弧の日付が無い `来週頭` のような相対的な書き方 | 期限の無いものとして扱う |

- 年が無いときは出典の日付の年を補い、それで出典の日付より前になるときは翌年とみなす。出典に日付が無ければ今日の年を使う。

4. 優先度に関係なく、期限の列の先頭に印を付ける。両方に当たるときは `**期限切れ・今週**` とする。
   - `**期限切れ**`: 期限(期間なら最後の日)が今日より前。
   - `**今週**`: 期限が `TODAY` の行の月曜から日曜の間。期間のときは、最後の日がこの間に入るときだけ付ける。

### 1-3. 表にして出す

次の列の表を 1 つ、会話に出す。

| 優先度 | 参照 | タスク | 期限 | 状態 | 詳細 |
|---|---|---|---|---|---|

- 参照は `<key>:<ID>`。
- 状態は台帳のまま出す(Issue の状態までは読みにいかない)。
- 詳細は `ROW` の詳細の列をそのまま出す。
- 共同担当の行は、タスクの末尾に「(共同: <自分以外の担当>)」を添える。

表の下に次を書く。

1. プロジェクトごとの件数。表に出した行の数で、`webshop n件・research n件` の形にする。飛ばしたプロジェクトは「飛ばした」と書く。
2. 担当が「未定」の行の、プロジェクトごとの件数(`担当が未定: webshop n件・research n件`)。
3. 片付けの候補(1-5)。候補があれば、片付けるかを尋ねる。無ければ書かない。

### 1-4. うまく読めないとき

- `SKIP` が出たプロジェクトは表に入れず、一覧の冒頭に「research を飛ばした: 台帳が見つからない(パス)」のように、飛ばしたことと理由を書く。
- `read: origin/main` のプロジェクトで `fetch` に失敗したときは、冒頭に「research は fetch に失敗したため、手元の origin/main(最後の取得 YYYY-MM-DD HH:MM)を読んだ」のように書く。
- `NODONE` が出たプロジェクトは、冒頭に「webshop は完了の節を読めないため、片付けの候補を確かめていない」と書く。

### 1-5. 片付けの候補

`priorities.md` の参照のうち、次のものを候補にする。候補を出すだけで、`priorities.md` は書き換えない。

- `DONE` にある ID: 「完了済み」
- `OPEN` にも `DONE` にも無い ID: 「台帳に見当たらない」。`priorities.md` の「タスク(写し)」と同じか近いタスク名の行が `OPEN` にあれば、「ID が変わった可能性: research:#12」として付け替えを候補にする(Issue を起票すると台帳の ID が Issue 番号に変わる運用のリポジトリがある)。見つからなければ、台帳の archive に移った可能性もあると添える。
- `projects.yaml` に無い key: 「登録簿に無いプロジェクト」

`SKIP` または `NODONE` のプロジェクトの参照は、確かめられないので候補にしない。

## 2. 優先度の相談

- 「優先度を見直したい」で起動したときは、1 の手順で一覧を作り、未設定のタスクと今の優先度を並べる。期限・親タスク・メモをもとに高・中・低の案を理由と一緒に出す。確認をもらってから `priorities.md` に書く。
- 「webshop:T-12 を高にして」のように個別に指定されたときは、1-1 の手順で台帳を読み、確認を挟まずに反映する。参照が `OPEN` に無いときは書かずに知らせる。
- 書き換えるときの決まり:
  - 「タスク(写し)」は、`OPEN` にあるその時点の台帳のタスク名に直す(書き換える行以外も、台帳と食い違っていれば直す)。
  - 状態・期限・担当は書かない。メモは利用者が言ったことだけを書く。
  - 行は高 → 中 → 低の順にまとめて置く。
- 書き換えたらコミットして push する(例: `git commit -m "priorities: webshop:T-12 を高にする"`)。

## 3. 片付け

- 「片付けて」と言われたとき、または一覧で片付けの候補を出して了承をもらったときに行う。
- 1-5 の候補を挙げ、確認をもらってから `priorities.md` の該当行を消す(付け替えの候補は参照を新しい ID に直す)。コミットして push する。

## 4. 期限切れの棚卸し

「期限切れを整理したい」「棚卸しして」で行う。`projects.yaml` で `write` を書いていないプロジェクトは `write: handoff` として扱う。

「webshop:T-3 を完了にして」「webshop:T-8 の期限を 12/5 にして」のような個別の指定も同じ手順で書く。そのときは次のようにする。
- 対象は期限に関係なく `OPEN` から探す。`OPEN` に無い ID は書かずに知らせる。
- 決め方がすでに示されているので 4-1 の手順 2 を飛ばし、手順 3・4 だけを行う。
- 担当が自分以外の行は、確認のときに担当を添えて知らせる。

### 4-1. 決めてもらう

1. 1-1 の手順で台帳を読み、`ROW` のうち 1-2 の読み方で期限切れのものを、プロジェクトごと(登録簿の順)・期限の古い順に並べる。
2. `| # | 参照 | タスク | 期限 | 状態 |` の表にして(通し番号は全プロジェクトで 1 本。`write: ledger` のプロジェクトは 1 つの表にまとめる)、1 件ずつ次のどれにするかを聞く。共同担当の行は 1-3 と同じくタスクの末尾に「(共同: <自分以外の担当>)」を添える。「1 完了、2 12/5、3 見送り、ほかはそのまま」のようにまとめて答えられるようにする。
   - 完了(完了日。言われなければ今日)
   - 期限を引き直す(新しい期限。年が無ければ今日の年とし、それで今日より前になるときは翌年)
   - 見送り(見送りにした日。言われなければ今日)
   - そのまま

   `write: handoff` のプロジェクトは表を分けて後ろに置き、表の上に次を添える。
   - Issue と台帳を併用するリポジトリでは、Issue 側ではもう閉じているか進んでいて、台帳の写しが遅れているだけの行がある
   - このプロジェクトは台帳をここから直さず、引き継ぎ文を出す(4-3)
   - 「台帳を Issue に合わせ直してから決める」とも答えられる

3. `write: ledger` のプロジェクトごとに、書く前の確かめをする。まだ何も書き換えない。
   - そのリポジトリの `AGENTS.md` の Git の決まりと、そこが指す運用ガイドの台帳の節(型の正本)を読む。この skill と食い違うときは、そのリポジトリの決まりに従う。
   - `git -C <repo> status -sb` で、ブランチ・未コミットの変更・未追跡のファイルを見る。push する決まりのプロジェクトでは、origin より先行しているコミットも見る(push しない決まりなら見ない)。決まりで作業を始めるブランチ(多くは main)以外にいるとき、または追跡しているファイルに未コミットの変更があるときは、そのプロジェクトは書かずに知らせる。
   - 決まりに台帳を直す前の手順(`git pull --ff-only` など)があれば行い、失敗したら書かずに知らせる。そのあと 1-1 の手順で読み直し、対象の行が手順 1 から変わっていれば、変わった行を確認に挙げる。
4. 書く内容を 1 回で確かめる。
   - プロジェクトごとの変更の一覧(ID・タスク名・担当・変更前 → 変更後)
   - 完了・見送りにするタスクの子(`OPEN` で親の列がそのタスクの行)が進行中に残るときは、その子(担当も添える)
   - `write: ledger` のプロジェクトの Git の流れ(ブランチ・lint・コミットメッセージの案・push・マージ)。コミットメッセージはそのリポジトリの `git log` の書き方(接頭辞・言語)に合わせる。
   - push するプロジェクトで origin より先行しているコミットがあるときは「push すると次の n 件も出る」とその一覧。未追跡のファイルがあるときは、コミットに含めないこと
   - 詳細がタスクファイル(`tasks/yyyymmdd_<slug>.md` など)を指すタスクの frontmatter の `status` を直すか(完了は `done`、見送りは `cancelled`。値はそのリポジトリの AGENTS.md の決まりに合わせる。子が進行中に残っていれば聞かない)
   - 完了にするタスクの詳細の列に、そのリポジトリの運用ガイドの決まり(例: 分析を終えたらレポートへのリンクを書く)でリンクを書き足すか。既存のリンクは残し、その台帳で複数のリンクをつないでいる記号(`・` や `、`)でつなぐ
   - 完了・見送りにするタスクに `priorities.md` の行があれば、その行を消すか(`write: ledger` の分だけ。`write: handoff` の分は反映されたあとの片付けで消す)

### 4-2. 台帳に書く(`write: ledger`)

確認をもらってから、プロジェクトごとに次の順で行う。

1. 書く直前に台帳を読み直し、対象の行が 4-1 で確かめた内容と同じかを見る。変わっていれば、その行は書かずに知らせる。
2. 決まりどおりにブランチを切る(決まりが main への直接のコミットなら切らない)。
3. 台帳を直す。行は ID で探し、その 1 行だけを書き換える。
   - 完了: 進行中の節から行を消し、完了の節の表の先頭(区切り行の直後)に足す(複数の行を移すときは完了日の新しいものを上にする)。完了の節の列は `| ID | タスク | 担当 | 完了日 | 親 | 出典 | 詳細 |` で、元の行から期限と状態の列を除き、4 列目に完了日を入れる。ほかの列は元の行のまま写す。
   - 見送り: 完了と同じく完了の節へ移し、タスク名の末尾に「（見送り）」を付け、4 列目に見送りにした日を入れる。
   - 期限を引き直す: 期限の列だけを `YYYY-MM-DD` で書き直す。
   - タスクファイルの `status` と詳細の列は、4-1 で直すと答えたものだけ直す。
4. コミットの前に確かめる。どれかに当たれば、コミットせずに知らせる。
   - `git -C <repo> diff` で、変わったのが対象の行(とタスクファイルの `status`)だけか
   - 1-1 の読み取りで、そのプロジェクトが `SKIP`・`NODONE` にならず、完了・見送りの ID が `DONE` にあり `OPEN` に無いこと、引き直した期限が `ROW` に出ていること
   - 決まりにある確かめ(lint など)が通ること
5. `git add` は台帳と直したタスクファイルだけを並べて指定する。そのリポジトリの決まりに `git add -A` とあっても、`-A`・`.` は使わない(関係の無いファイルを棚卸しのコミットに入れないため。この点だけはリポジトリの決まりより優先する)。4-1 で確かめたメッセージでコミットし、決まりどおりに push・マージする(push しない・ローカルで main にマージするなどの決まりもそのまま守る)。
6. 報告に、プロジェクトごとのコミット(`git show --stat`)と、台帳の変わった行を添える。

### 4-3. 引き継ぎ文を出す(`write: handoff`)

`write: handoff` のプロジェクトは、台帳・Issue をここから書き換えない。台帳を PR で直す、Issue の値を台帳に写すなど、そのリポジトリの流れで直してもらうため、次の 2 つを出す。

1. **自分で Issue の画面ですること**(Issue と台帳を併用するリポジトリだけ): Issue 番号の付いた行の一覧(ID・タスク名・決めたこと)。完了は close、見送りは not planned で close、期限は Issue やプロジェクトボードの期限を直す。
2. **そのリポジトリのセッションに貼る引き継ぎ文**: 下の型で、当てはまる行が無い手順を消して番号を振り直す。`<!-- -->` のコメントは出す前に消す。「台帳を Issue に合わせ直してから決める」と答えられたときは、目的を「台帳を Issue の今の値に合わせ直す」にし、決まったことの表を除く。

```markdown
## 目的
タスク台帳(tasks/index.md)の期限切れのタスクを、ハブでの棚卸しで決めた内容に合わせて直す。

## 入力ファイル
- tasks/index.md
- AGENTS.md の Git の決まりと、そこが指す運用ガイドの台帳の節

## 決まったこと
<!-- 「そのまま」以外にした行について、棚卸しのときの台帳の値と、決めたことを並べる -->
| ID | タスク | 棚卸しのときの台帳 | 決めたこと |
|---|---|---|---|
| #12 | 調査の結果を共有する | 期限 2026-11-28・進行中 | 期限を 2026-12-05 に引き直す(Issue の画面で直す) |
| T-7 | 集計の手順を見直す | 期限 未定・未着手 | 見送り(台帳で直す) |

## 開いている論点
- Issue の今の値が「棚卸しのときの台帳」と同じなら、Issue の画面での変更がまだなので、その行は台帳を直さずに残す。「決めたこと」と同じなら写す。どちらとも異なるときは、直す前に相談する。

## 最初にやること
1. AGENTS.md の決まりのとおり main を最新にして作業用のブランチを切る。
2. Issue と台帳を併用しているなら、Issue の今の値を確かめ、台帳の担当・期限・状態を写す(閉じた Issue の行は完了の節へ移す)。
3. Issue にしていない行は、決まったことのとおりに台帳を直す。
4. 完了・見送りにした行の詳細がタスクファイルを指していれば、そのファイルの `status` を運用ガイドの決まりどおり直すかを確かめる。
5. 差分を見せてコミットし、PR を出す(マージは人が行う)。ハブの一覧には、マージされてから反映される。
```

### 4-4. priorities.md を直す

4-1 で消すと答えた `priorities.md` の行のうち、4-2 の手順 4 の確かめを通って台帳に書けたものだけを消し、ハブのリポジトリでコミットして push する(例: `git commit -m "priorities: 完了した webshop:T-3 を外す"`)。

## 既知の制約

- `read: origin/main` のプロジェクトは、まだマージしていない自分のブランチでの台帳の変更が、マージされるまで一覧に出ない。
- 担当は表記で判定する。`owner` と異なる表記(敬称付き・ローマ字など)の行は対象にならない。
- 台帳の列や節の名前を各リポジトリで変えると読めなくなる(`SKIP` になる)。`登録先` のような列を足すときは、読み取りの `EXPECT` もあわせて直す。セルの中にエスケープしない `|` を書いた行が 1 行あるだけでも、そのプロジェクトを飛ばす。
