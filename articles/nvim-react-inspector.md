---
title: "React Inspector (Chrome 拡張) を Neovim で利用する"
emoji: "⚛️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# React Inspector とは

開発者の方の記事がありますので、こちらを参照してください。

@[card](https://zenn.dev/hand_dot/articles/a1523d8b0449ad)

ブラウザから選択した要素のコンポーネントの実装ファイルに一発で飛ぶことができて便利です。
特に、開発から時間がたった画面とか他の人が作った画面を見る際に、重宝しています。

# Neovim で React Inspector を利用する

VSCode と JetBrains IDE については、それぞれの IDE の機能とそれに応じたプリセットが拡張機能にあるため、React Inspector を容易に利用できます。

![react-inspector-options](/images/nvim-react-inspector/react-inspector-option.png)

IntelliJ では HTTP 通信によって、ファイルを開けるため、Neovim でも同様の機能を実現できそうです。

## HTTP 通信で開けるようにする

おそらく調べた限りでは、Neovim 標準の機能や既存のプラグインでは、HTTP 通信でファイルを開くことはできないようでした。（検索不足の可能性はある）
大きな処理ではないですが、今回はプラグインを作成することにしました。

作成したプラグインはこちらです。

https://github.com/nabekou29/open-by-http.nvim

Neovim のドキュメントに簡単なサーバーを立てる方法が載っていたので、それを参考にしました。

https://neovim.io/doc/user/lua.html#lua-loop
