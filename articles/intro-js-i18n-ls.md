---
title: "特定のエディタに縛られない i18n 開発体験を LSP で実現した【Neovim/VSCode】"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

以前、Neovim 用に作成した i18n のプラグインを VSCode にも対応させたので、紹介です。

以前の記事はこちらです。（若干重複する内容があります）

https://zenn.dev/nabekou29/articles/nvim-i18n-plugin#js-i18n.nvim

この記事では `LSP` という単語を `LSP サーバー` という意味で書いてある場所が多々あります。ご了承ください。

## 紹介

**作ったものはこちらです！**

https://marketplace.visualstudio.com/items?itemName=nabekou29.js-i18n

- VSCode: https://github.com/nabekou29/vscode-js-i18n
- Neovim: https://github.com/nabekou29/js-i18n.nvim
- LSP サーバー: https://github.com/nabekou29/js-i18n-language-server

雰囲気だけのデモ動画がこちら

https://youtu.be/HdP_PnHlzsY

### 代表的な機能

i18next系 / next-intl をサポートしています。(今後増やしていきたい)

- 定義へのジャンプ （翻訳キー → JSONファイル）
- 翻訳キーの参照表示
- 翻訳キーの補完
- 翻訳キーのリネーム
- 診断
  - 翻訳漏れの検出
  - **未使用キーの検出**
- 翻訳のインライン表示
- 翻訳内容の編集

❌ 翻訳ツールとの統合などは作っていません。

### こだわりポイント

#### LSP サーバーとして作る

#### なるべく Zero Config で動くように

####

## 作った理由

### Neovim プラグインを作った理由

作ったのは1年以上前になりますが、当時 Neovim に入門したてで目立った困りの1つが i18n 開発まわりでした。  
VSCode には優れた拡張機能がある中で、満足のいく Neovim のプラグインは見つからず、

???「全員が全員 VSCode だと思うなよ」

という怒りの気持ちで作りました。

実際自分以外にも困っている人はいたみたいで、少なくとも2桁人くらいは使ってくれていそうです。

### VSCode 版を作った理由・ i18n ally への不満

- **Neovim 版が思いのほか良いものになったので、チームメンバーを含め VSCode ユーザーにも使ってもらいたいと思った。**
- 機能を増やしていくときにあたって LSP サーバーの作り直しが必要だったため、ついでに VSCode 版も作ることにした。

というのが主な理由です。

とはいえ Neovim と違い、VSCode では [i18n Ally](https://marketplace.visualstudio.com/items?itemName=lokalise.i18n-ally) [^1]という拡張機能が有名です。

[^1]: i18n Ally 現在は開発が止まっている？みたいで、フォークして

実際、i18n Ally は Unipos の開発でも使われていますが、気になる点もいくつかありました。

- キーの参照の表示ができない
- 未使用キーがインタラクティブに検出されない
- モノレポ未対応
- Key Prefix 未対応
- i18next の Plural 未対応

など。  
正直、重箱の隅的な感じもしますが、**i18n 開発の体験を向上させるために、これらの点を改善したいと思いました。**

---

そもそも、エディタを開く機会が減っていくこの時代においては LSP の開発の優先度は落ちているような気もしますが、
週に1回でもエディタを開いているうちはあった方が嬉しいと思うので、考えないこととしました。

---

## Rust にした理由・してよかった理由

序盤は手で書いていたが、後半はほぼ Claude Code に実装させた。

speckit も軽く触ったりしているが、仕様以上に何のフレームワークを使うかとかデータモデル設計などがかなり重要だと思う。

結局のところ、Rust の習得にはいたらず、途中からはほぼコードを見なくなってしまった。

salsa に乗っかって作っただけ。

最初、salsa を使えという指示なしで作らせたところ、動かなかったり動作が非常に重くなったりと。
とらえずの素振り的に作ったものとはいえ。

## なるべく Zero Config で動くように

言語の特定。

## LSP サーバーとして作る

これは前にも

## いろーんなケースに対応

未使用キーの警告を出すにあたって、間違って消してしまうことが少ないように工夫した。

namespace, keyPrefix, useTrasnslation の別名。

妥協した部分もあって、例えば [Nesting](https://www.i18next.com/translation-function/nesting#basic) は対応してないです。
今気づきましたが、 [Context](https://www.i18next.com/translation-function/context#basic) も対応してないです。なぁにこれぇ。

## おわりに

今回、動作確認は主に Neovim でやりました。

Neovim の方が LSP の設定がしやすいし、テストもしやすいんじゃないかと思います。

需要があれば vue や svelte などへの対応もできるといいなと思っています。

CLI 版があっても良さそう。
