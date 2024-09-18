---
title: "React Inspector (Chrome 拡張) を Neovim で利用する"
emoji: "⚛️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "react"]
published: true
---

# React Inspector とは

開発者の方の記事がありますので、こちらを参照してください。

@[card](https://zenn.dev/hand_dot/articles/a1523d8b0449ad)

ブラウザから選択した要素のコンポーネントの実装ファイルに一発で飛ぶことができます。
開発中はコンポーネント名が頭に入っているのですが、時間が経つと忘れてしまいます。
そんなときに素早く目的のコンポーネントに辿り着けるのは非常に便利だなと思っています。

以前は VSCode で開発していて React Inspector を使えていたのですが、最近は Neovim で開発をするようになったため使えずにいました。
そこで、今回は Neovim でも React Inspector を使う方法を模索してみました。

# Neovim で React Inspector を利用する

VSCode と JetBrains IDE については、それぞれの IDE の機能とそれに応じたプリセットが拡張機能にあるため、React Inspector を容易に利用できます。

![react-inspector-options](/images/nvim-react-inspector/react-inspector-option.png)

IntelliJ では HTTP 通信によってファイルを開けるため、Neovim でも同様の機能を実現できそうです。

## HTTP 通信で開けるようにする

調べた限りでは、Neovim 標準の機能や既存のプラグインでは、HTTP 通信でファイルを開くことはできないようでした。（検索不足の可能性はある）
大した実装ではないですが、今回はプラグインを作成することにしました。

作成したプラグインはこちらです。

https://github.com/nabekou29/open-by-http.nvim

Neovim のドキュメントに簡単なサーバーを立てる方法が載っていたので、それを参考にしました。

https://neovim.io/doc/user/lua.html#lua-loop

このプラグインを入れることで、以下のような方法でファイルを開くことができるようになります。

```sh
curl http://localhost:8682/api/file/src/index.ts:10:5
```

あとは、React Inspector を以下のように設定すれば動きます。

- Open in Editor URL: `http://localhost:8682/api/file/{path}:{line}:{column}`
- Method: `Call with fetch`

## 完成

これで、React Inspector から Neovim でファイルを開くことができるようになりました。

https://youtu.be/AnYWP4PfU6k

# あとがき

こちらの記事からインスピレーションを受けて今回の調査をするに至りました。

https://zenn.dev/vim_jp/articles/5a245a6c278929

フロントエンドの開発は VSCode が主流だと思いますし、比較すると Neovim だと不便を感じることもたまにあります。
ですが、今回のようにちょっとした知識や工夫で解決できることもあるなと思いました。
フロントエンド開発界隈でも Neovim が盛り上がるように微力ながら貢献できればと思います。
