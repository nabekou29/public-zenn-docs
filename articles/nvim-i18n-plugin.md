---
title: "Neovim で React の多言語対応の開発を効率化するプラグイン 「js-i18n.nvim」の紹介"
emoji: "🌐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - 'vim'
  - 'neovim'
  - 'react'
  - 'i18n'
  - 'lua'
published: true
---

:::message
[Unipos Advent Calendar 2024](https://qiita.com/advent-calendar/2024/unipos) 22日目の記事です
:::

## はじめに

以前は React の開発には VSCode を利用していたのですが、今年から本格的に Neovim を利用して、Web フロントエンドの開発をするようになりました。  
いろんなプラグインを導入しながら楽しく開発していたのですが、大きな悩みとして**多言語対応のやりづらさ**がありました。

VSCode では [i18n Ally](https://marketplace.visualstudio.com/items?itemName=lokalise.i18n-ally) という有名なプラグインがあり、私自身これを利用していました。
しかし、Neovim にはそのような多機能かつメジャーなプラグインがなかったため、いっそのこと勉強がてら **自分でプラグインを作ってみることにしました**。

今年の春頃から少しずつ作っていき、ある程度機能が揃ってきたので紹介したいと思います。

プラグインの作成の基本的な流れは以前の記事で書いているので、今回は機能実現のための工夫した部分などの誰得な情報をおまけとして書いておきます。

https://zenn.dev/nabekou29/articles/neovim-lazy-nvim-development

Neovim で Web フロントの開発をしていて、多言語対応をしているような方はレアな気はしますが、誰かの役に立てば幸いです。

## js-i18n.nvim について

Neovim のプラグインで、多言語対応の開発効率化を目的としたプラグインになっています。
現状は [`i18next`](https://www.i18next.com/), [`next-intl`](https://next-intl.dev/) の２つのライブラリに対応していて、モノレポ環境でも動作するようになっています。

（おそらく i18n Ally はモノレポ環境に対応しておらず、モノレポ環境での動作は１つのこだわりポイントになっています）

インストール方法などは下記のリポジトリを参照してください。

https://github.com/nabekou29/js-i18n.nvim

## 主な機能

スクショなどを撮るにあたり、以下のリポジトリを利用しました。

https://github.com/i18next/next-i18next/tree/master/examples/simple

### 1. キーに対応する文言の情報の表示

基本的なインライン表示に加えて、文言の定義が漏れていれば警告が表示されます。
これにより文言の指定のミスや漏れを防ぐことができます。

小さいですが全体感はこんな感じです。

![](/images/nvim-i18n-plugin/translation-info-basic.png)
*通常の `t(...)` に加えて `<Trans>` にも対応。また長すぎるテキストは省略表示できる。表示言語の切り替えも可能。*

![](/images/nvim-i18n-plugin/translation-info-diagnostics.png)
*文言の定義が漏れている場合は警告が表示される。*

![](/images/nvim-i18n-plugin/translation-info-hover.png)
*言語を切り替えずともホバーで全言語の文言を確認できる。*

#### 細かい挙動

これ以降に紹介する機能も含めて、以下のようなケースでも正しく文言を取得できるようになっています。

- `useTranslation` でネームスペースやプレフィックスを指定する場合への対応
- 同じスコープに `useTranslation` が複数ある場合の対応

![](/images/nvim-i18n-plugin/translation-info-multiple.png)

### 2. 文言の追加・編集

効率化という点では**この機能が最も役に立っている**と思います。

コマンドかコードアクションによって、文言の追加・編集ができます。**個人的にはコードアクションの方が使いやすい**と思っています。
コードアクションで追加・編集する場合は、新たにキーバインドを設定する必要がないのがメリットです。

https://youtu.be/61UoLjFBEHs

### 3. ソースの定義・参照ジャンプ

既存コードの修正をする際には、**画面の文言から実装を探したいことがあります。**
しかし、文言を JSON ファイルに分離している場合、文言の定義元を探すのが面倒です。そういった時には参照ジャンプが便利です。

1. 修正したいUIの文言でコード内を検索する
1. JSON ファイルを開く
1. 参照している TSX ファイルを開いて修正する

といった手順をよく踏むのですが、この流れがスムーズに行えます。

Language Server として実装しているので、こちらも新しくキーバインドを覚える必要は無いです。自然にジャンプできると思います。

https://youtu.be/laC00Xfwb4w

## 実装の工夫など（おまけ）

### Language Server として実装する

Language Server として実装することで、慣れた操作でプラグインを利用できるようになります。
主にコードアクションや定義ジャンプまわりですね。

Language Server といえば基本的には外部のコマンドを呼び出すもので、プラグインとは別になっているものがほとんどです。
しかし、最初のうちはなるべくシンプルに実装したかったのと、Lua での実装にも慣れたかったのでプラグイン内で完結できる実装方法にしました。
外部コマンド以外での実装方法は調べても全然わからなかったので、ちょっと大変でした。

```lua
-- 外部コマンドパターン
configs.foo_lsp = {
  default_config = {
    cmd = {'/home/neovim/lua-language-server/run.sh'}, -- こんな感じで外部コマンドを呼び出すのが普通
    filetypes = {'lua'},
  },
}

-- プラグイン内で完結パターン
configs.i18n_lsp = {
  cmd = function(dispatchers) -- 実はこういう書き方もできる
    return require("js-i18n.lsp").create_rpc(...)
  end,
}
```

LSP の仕様の理解などには以下の記事や本が参考になりました。めちゃくちゃ助かりました。

https://zenn.dev/takl/books/0fe11c6e177223
https://zenn.dev/mtshiba/books/language_server_protocol

### Tree-sitter の利用

複数ライブラリに対応するに当たって、Tree-sitter を利用して解析することで、ライブラリごとの関数のシグネチャの違いなどを吸収することができました。

Tree-sitter の知識がないとわかりづらいと思いますが、`@i18n.xxx` となっている箇所でマッチした部分に名前をつけています。
名前を揃えておけばクエリから取得できる情報の構造が揃うため、それ以降の処理は共通化できるようになります。
また、`react-i18next` では `t(...)` に加えて `<Trans>` や `<Translation>`というタグを使う場合があるのですが、それらもクエリを追加するだけで対応できるようになりました。

```scm:react-18n.scm
;; useTranslation 関数呼び出し
(variable_declarator
  name: (object_pattern
    [
      (pair_pattern
        key: (property_identifier) @use_translation_t (#eq? @use_translation_t "t")
        value: (identifier) @i18n.t_func_name
      )
      (shorthand_property_identifier_pattern) @i18n.t_func_name
    ]
    )
  value:
    (call_expression
      function: (identifier) @use_translation (#eq? @use_translation "useTranslation")
      arguments: (arguments
        [
          (string (string_fragment) @i18n.namespace)
          (array)
          (undefined)
        ]?
        (object
          (pair
            key: (property_identifier) @key_prefix_key (#eq? @key_prefix_key "keyPrefix")
            value: (string (string_fragment) @i18n.key_prefix)
          )?
        )?
      )
    )
) @i18n.get_t
```

```scm:next-intl.scm
;; useTranslations 関数呼び出し
(variable_declarator
  name: (identifier) @i18n.t_func_name
  value:
    (call_expression
      function: (identifier) @use_translations (#eq? @use_translations "useTranslations")
      arguments: (arguments
        [
          (string (string_fragment) @i18n.key_prefix)
          (undefined)
        ]?
      )
    )
) @i18n.get_t
```

複数ライブラリの対応のために導入していたのですが、`t` 以外の変数名への対応をするにあたっても、Tree-sitter を使っていたことで、比較的簡単に対応できました。

## おわりに

今回は、Neovim で React の多言語対応の開発を効率化するプラグイン「js-i18n.nvim」を紹介しました。

プラグインの開発によって、普段の開発が効率化されたのはもちろん、Neovim のプラグインについての理解が深まり他のプラグインの実装も読めるようになって盆栽が捗ったりと良いことが多かったです。
何よりも**自分のプラグインを使う→改善するのサイクルが何よりも楽しかったです。**

まだいくつか追加で対応したいものがあったり、Lua の勉強にはある程度満足したので Language Server 部分を外部コマンドに切り出してみたりなど、やってみたいことがいくつかあります。（外部コマンド化したら VSCode でも使えるようにしたり...）

来年も Neovim での開発を楽しんでいこうと思います。
