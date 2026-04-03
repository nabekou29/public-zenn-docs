---
title: "i18n 開発体験を LSP で底上げした話、Neovim でも VSCode でも"
emoji: "🌐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "neovim"
  - "vscode"
  - "i18n"
  - "rust"
published: true
publication_name: "unipos"
---

i18next などの JS 系の i18n ライブラリを使ったコードを読み書きしやすくするための LSP サーバーを作ったのでその紹介と、作るに至った経緯など書いていきます。

最近では AI で開発するのがメインで1日に数回しかエディタは開かないから LSP なんて興味ないという人もいるかもですが、今回は考えないものとします。
コードを読むときには便利だと思うので、もうしばらく存在意義はあると思います。

今回紹介するものは、以前にNeovim 用に作成した i18n のプラグインを大幅に改良し VSCode にも対応させたものになります。

以前の記事はこちらです。（若干重複する内容があります）

https://zenn.dev/nabekou29/articles/nvim-i18n-plugin#js-i18n.nvim

この記事では `LSP` という単語を `LSP サーバー` という意味で書いてある場所がありますが、ご了承ください。

## 紹介

**作ったものはこちらです！**

https://marketplace.visualstudio.com/items?itemName=nabekou29.js-i18n

- VSCode: https://github.com/nabekou29/vscode-js-i18n
- Neovim: https://github.com/nabekou29/js-i18n.nvim
- LSP サーバー: https://github.com/nabekou29/js-i18n-language-server

デモ動画も撮りました。

https://youtu.be/HdP_PnHlzsY

動画撮影はなるべくスクリプト化してるのですが、マウス操作を見せづらいのが困りどころです。

### 代表的な機能

i18next / next-intl / vue-i18n / svelte-i18n をサポートしています。

- 定義へのジャンプ （翻訳キー → JSONファイル）
- **翻訳キーの参照一覧** (JSONファイル → 翻訳キー利用箇所)
- 翻訳キーの補完
- 翻訳キーのリネーム
- 診断
  - 翻訳漏れの検出
  - **未使用キーの検出**
- 翻訳のインライン表示
- 翻訳内容の編集

### こだわりポイント

#### LSP サーバーとして作る

LSP サーバーとして作ることで、特定のエディタに縛られずに同じ体験を提供できるようになります。

慣れたキーバインドで操作ができるのも良いところだと思います。i18n 用の拡張機能はニッチな部類だと思うので、わざわざ新しいことを覚えなくていいというのは大事だと思います。

#### 未使用キーの検出

![JSONで管理している翻訳ファイルの中で使われていないキーに対する診断が表示されている様子](https://raw.githubusercontent.com/nabekou29/vscode-js-i18n/refs/heads/main/docs/images/key-prefix.png)

未使用キーの検出、参照リストは特に実装的に大変だった機能です。
ほかの機能は基本的にキー (ソースコード) → JSON の参照になるのですが、未使用キーを検出するためにはソースコードを全て読み込んでおく必要があるためです。

lua をやめて1から作り直そうと思ったのは、これをやりたかったのが大きいです。

また、なるべく正確に未使用キーを検出できるように、各ライブラリの namespace や key-prefix などの機能に対応するようにもしています。

#### なるべく Zero Config で動くように

JS系の i18n ライブラリは書き味が結構似てはいるのですが、機能やインターフェースに若干の違いはあります。

同じライブラリを使っていてもどのように翻訳ファイルを置くかなどはプロジェクトによって様々です。
例えば、`./locals/en/translation.json` のようなパターンもあれば、 `./messages/en.json` のようなパターンもあります。
このようなどちらのパターンでも LSP が正しく言語を特定できるようにしたりしています。

## 作った理由

### Neovim プラグインを作った理由

作ったのは1年以上前になりますが、当時 Neovim を使い始めたばかりだったころの困りの1つが i18n 開発まわりでした。
VSCode には優れた拡張機能がある中で、満足のいく Neovim のプラグインは見つからず、

???「全員が全員 VSCode だと思うなよ」

という怒りの気持ちで作りました。

実際自分以外にも困っている人はいたみたいで、少なくとも2桁人くらいは使ってくれていそうです。感謝。

### VSCode 版を作った理由・ i18n ally への不満

- **Neovim 版が思いのほか良いものになったので、チームメンバーを含め VSCode ユーザーにも使ってもらいたいと思った。**
- 機能を増やしていくときにあたって LSP サーバーの作り直しが必要だったため、ついでに VSCode 版も作ることにした。

というのが主な理由です。

しかし Neovim と違い、VSCode では [i18n Ally](https://marketplace.visualstudio.com/items?itemName=lokalise.i18n-ally) [^1]という拡張機能が有名です。

[^1]: i18n Ally 現在は開発が止まっている？みたいで、フォークして開発を続けている人もいるみたいです。https://github.com/lokalise/i18n-ally/discussions/1377

実際、i18n Ally は Unipos の開発でも使われていますが、気になる点もいくつかありました。

- 未使用キーが自動で検出されない
- モノレポ未対応
- Key Prefix 未対応
- i18next の Plural 未対応

など。
正直、重箱の隅的な感じもしますが、**i18n 開発の体験を向上させるために、これらの点を改善したいと思いました。**
特にメインで使っている i18next への対応は厚めにやろうと思って、多くの構文をサポートしてます。

ただ、妥協した部分もあり、例えば [Nesting](https://www.i18next.com/translation-function/nesting#basic) は対応してないです。
今気づきましたが、 [Context](https://www.i18next.com/translation-function/context#basic) も対応してないです。なぁにこれぇ。

## おわりに

今回、動作確認は主に Neovim でやりました。
Neovim の方が LSP の設定がしやすいし、テストもしやすいんじゃないかと思います。Neovim 最高！

LSP サーバーは Rust で作りました。基盤と元々 Lua で作っていた機能の移植はある程度の人力も交えながらやって、vue / svelte のライブラリ対応は Claude Code にやらせました。
最終的に rust-analyzer などでも使われている salsa を採用したのですが、最初は使わずに作っていてなかなかカオスなことになっていました。
改めて土台作りの大切さと土台がしっかりしている状態での AI の生産性の高さを実感しました。
