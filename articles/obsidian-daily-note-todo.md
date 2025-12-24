---
title: "デイリーノートとがんばらないTODO管理【Obsidian】"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["obsidian", "タスク管理", "templater"]
published: false
publication_name: "unipos"
---

この記事は [Unipos Advent Calendar 2025](https://qiita.com/advent-calendar/2025/unipos) 18日目の記事です。

## TODO管理、続かない問題

TODO管理は悩みのタネになりがちです。「TODO管理 続かない」で検索すれば山ほど記事が出てきますし、ツールを変えても方法論を学んでも、結局続かない人は多いのではないでしょうか。

私も、チームのメインタスクは別で管理していて困ることはないのですが、**個人の細かいタスク管理が続かない**のが悩みでした。

- 余裕ある時に見たい issue
- アンケートの回答
- ちょっとした調べ物
- 後で読みたいドキュメント

こういったものを TODO アプリで管理しようとしても、いちいちアプリを開いて追加したりが面倒で、結局続きません。

## どうしたか

**Obsidian のデイリーノートに Markdown 形式で TODO を書いて管理するようにしました。**

半年ほど続いていて、個人の細かいタスク管理にはこれで十分だと感じています。

以下のようにチェックリスト形式で TODO を書いていて、メモや子の項目もインデントをつけて書いています。  
メモは後から追加したり、ちょっとだけ作業したときに追記したりすることが多いです。

```markdown
# 12月17日のノート

## TODO

- [ ] issue #123 を見る
      ×× を ○○ したら上手くいきそう
- [x] アンケート回答 https://example.com/survey
- [ ] ○○さんの記事を読む
```

12月18日のノートを作ると、前日の未完了 TODO が自動で引き継がれるようにしています。

```markdown
# 12月18日のノート

## TODO

- [ ] issue #123 を見る
      ×× を ○○ したら上手くいきそう
- [ ] ○○さんの記事を読む
```

実際の動作はこんな感じです。仕組みは後述します。

![デイリーノートで未完了TODOが引き継がれる様子のGIF](/images/obsidian-daily-note-todo/demo.gif)

## なぜデイリーノートか

デイリーノートを使っている理由は以下の通りです。

- **プレーンなテキストでシンプル**
- **ちょうどいいタイミングで目に入る**
- **完了した記録も残る**

### プレーンなテキストでシンプル

**シンプル故に追加や編集が楽で、かつ全体感を把握しやすいです。**
好きなエディタで編集すると使い慣れたインターフェースで編集できてより快適です。

手書きのホワイトボードなども一定の支持がある手法だと思いますが、**後から追記しやすい点でデジタルの方が便利。**  
また、在宅とオフィスの両方で仕事するので、物理的なホワイトボードや付箋などは使いにくいです。

### ちょうどいいタイミングで目に入る

もともとデイリーノート自体もがっつり運用しているわけではないのですが、やったことや気づいたことなどをメモするのには使っていました。

**作業の区切りなどで開くことが多く、ちょうどいいタイミングで目に入る**のが良いです。

### 完了した記録も残る

一時期は、単一のファイルで TODO 管理をしていたこともあるのですが、完了したタスクが消えてしまうことに少しもったいなさを感じていました。

「振り返りでも確認したいから一週間は残しておこう」とか考えると、消すタイミングにも悩みます。

デイリーノートにすることで、1日ごとにクリーンになるのと、必要なら遡って確認できるので管理コストが減っていい感じです。

## 仕組み

別に手動でも運用はできるのですが、面倒なので**デイリーノート作成時に前日の未完了 TODO を自動で引き継ぐ仕組み**を作っています。

この仕組みは [Templater](https://github.com/SilentVoid13/Templater) プラグインを使って実現しています。

スクリプトは AI に書かせたので、真似するにしても自分のスタイルに合わせて作り直すのが良いかもしれません。
どんなことができるのかの参考にしてもらえればと思います。

Templater 自体はファイルの生成を監視しているっぽいので、Obsidian 上でファイルを生成せずとも、外部でファイルを作成しても動作します。
そのため、私は Neovim 上で生成・編集をしています。

### デイリーノートのテンプレート

```markdown
---
id: <%- tp.date.now("YYYY-MM-DD") %>
aliases:
  - <%- tp.date.now("YYYY年MM月DD日") %>
tags:
  - daily-notes
---

## TODO

<% await tp.user.getPreviousTodos(tp) -%>

## Memo
```

`<% await tp.user.getPreviousTodos(tp) -%>` の部分が前日の未完了TODOを取得して展開してくれます。

### 未完了TODOを取得するスクリプト

Templater のユーザースクリプト機能を使います。以下のスクリプトを `_scripts/templater/getPreviousTodos.js` として保存します。

```javascript
async function getPreviousTodos(tp) {
  const today = tp.date.now("YYYY-MM-DD");

  // 日付からファイルパスを生成
  function generateFilePath(dateString) {
    const year = dateString.substring(0, 4);
    const month = dateString.substring(5, 7);
    // ファイルパスは環境に合わせて変更してください
    return `daily/${year}/${month}/${dateString}.md`;
  }

  // 前の日付を取得
  function getPreviousDate(dateString) {
    const date = new Date(dateString);
    date.setDate(date.getDate() - 1);
    return date.toISOString().split("T")[0];
  }

  // 最大30日前まで遡る
  let currentDate = getPreviousDate(today);
  let attempts = 0;
  const maxAttempts = 30;

  while (attempts < maxAttempts) {
    const filePath = generateFilePath(currentDate);
    const file = app.vault.getAbstractFileByPath(filePath);

    if (file && file.extension === "md") {
      const content = await app.vault.read(file);

      // TODOセクションを抽出
      const todoSectionMatch = content.match(/## TODO\n([\s\S]*?)(?=\n## |$)/);

      if (todoSectionMatch) {
        const todoSection = todoSectionMatch[1];
        const lines = todoSection.split("\n");

        // 未完了のTODO項目とその子項目を抽出
        const incompleteTodoBlocks = [];
        let i = 0;

        while (i < lines.length) {
          const line = lines[i];

          if (line.trim().startsWith("- [ ]") && line.trim() !== "- [ ]") {
            const todoBlock = [line];
            const todoIndentLevel = line.length - line.trimStart().length;
            i++;

            // 子項目も取得
            while (i < lines.length) {
              const nextLine = lines[i];
              const nextLineIndent =
                nextLine.length - nextLine.trimStart().length;

              if (nextLine.trim() === "") {
                i++;
                continue;
              }

              if (
                nextLineIndent <= todoIndentLevel &&
                (nextLine.trim().startsWith("- [ ]") ||
                  nextLine.trim().startsWith("- [x]"))
              ) {
                break;
              }

              if (nextLineIndent > todoIndentLevel) {
                todoBlock.push(nextLine);
              } else {
                break;
              }
              i++;
            }

            incompleteTodoBlocks.push(todoBlock.join("\n"));
          } else {
            i++;
          }
        }

        if (incompleteTodoBlocks.length > 0) {
          return incompleteTodoBlocks.join("\n") + "\n";
        }
      }
    }

    currentDate = getPreviousDate(currentDate);
    attempts++;
  }

  return "";
}

module.exports = getPreviousTodos;
```

### スクリプトの動作

1. 前日のデイリーノートを探す
2. `## TODO` セクションから未完了タスク（`- [ ]`）を抽出
3. 子項目（インデントされた行）も一緒に取得

### Templater の設定

1. Templater の設定を開く
2. 「Script files folder location」にスクリプトのフォルダを指定（例: `_scripts/templater`）
3. デイリーノート作成時にテンプレートが適用されるよう設定

## おわりに

Obsidian のデイリーノートを使ったシンプルな TODO 管理の方法の紹介でした。

デイリーノートを使わずとも、プレーンテキストでの管理自体もおすすめなので、TODO 管理に悩んでいる方はぜひ試してみてください。
