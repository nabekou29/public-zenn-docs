---
title: "Neovim のプラグイン開発をした学びを作りながら振り返る"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "lua", "lazynvim"]
published: true
---

Neovim を本格的に利用しはじめてから半年ほど経つのですが、とうとう自分でプラグインを作ることになりました。今回は、そのプラグイン開発において学んだことをつらつらと書き留めておこうと思います。

自分への備忘録としつつ、これからプラグイン開発を始める方にも参考になればと思います。

ちなみに作成したプラグインは[こちら](https://github.com/nabekou29/js-i18n.nvim)です。この子についてもそのうち記事を書くかもしれません。

https://github.com/nabekou29/js-i18n.nvim

# 前提

- **lua によるプラグイン開発**

  denops などを利用すると TypeScript で書くこともできるらしい。今回は初めてのプラグイン開発で、せっかくなので lua で書くことにしています。
  結果的には他のプラグインの実装を参考にできることが多かったので lua で良かったとも思う。

- **プラグインマネージャーは lazy.nvim を利用**

  プラグインマネージャーによって読み込み方などは変わるので、参考にする方がいたら適宜読み替えてください。

- **筆者は lua を体系的に学んではいない**

  プラグインの設定をいじったりする程度の知識でスタートして、結局最後までちゃんと学ぶことはなかったです。

# 簡単なプラグインを作成する

簡単なプラグインを作成しながら、プラグイン開発の流れを確認していきます。
今回は `example.nvim` という名前でプラグインを作成します。

## プラグインに必要なファイルを作成

```sh
example.nvim
├── lua
│   └── example
│       └── init.lua
├── stylua.toml
└── .luarc.json
```

## ビルトインのAPIの補完が効くようにする (`.luarc.json`)

`.luarc.json` を作成して、lua の lsp での補完がされるようにします。lua-language-server が有効になっている環境が前提です。
プラグインを作成しない場合でも `~/.config/nvim/.luarc.json` を設定しておくと、Neovim の設定をする際に補完が効くので設定して損はないと思います。

:::details ~/.config/nvim/.luarc.json の例
```json
{
  "$schema": "https://raw.githubusercontent.com/LuaLS/vscode-lua/master/setting/schema.json",
  "runtime": {
    "version": "LuaJIT"
  },
  "workspace": {
    "library": [
      "lua",
      "$VIMRUNTIME",
      "${3rd}/luv/library",
      "${3rd}/busted/library",
      "$HOME/.local/share/nvim/lazy"
    ],
    "checkThirdParty": false
  }
}
```

`"$HOME/.local/share/nvim/lazy"` は lazy.nvim の場合の設定です。
プラグインの数が多いと、lua_ls の起動が重くなるケースもありそうですが、Config の型を使えることもあるので、
個人的にはあった方が良いと思います。

:::


```json:.luarc.json
{
  "$schema": "https://raw.githubusercontent.com/LuaLS/vscode-lua/master/setting/schema.json",
  "runtime": {
    "version": "LuaJIT"
  },
  "workspace": {
    "library": ["lua", "$VIMRUNTIME", "${3rd}/luv/library"]
  }
}
```

:::message
`workspace.library` は最低限 `lua` と `$VIMRUNTIME` を指定しておくと良いです。
私の環境では `vim.uv.xxx` などの補完が動くためには `${3rd}/luv/library` を指定する必要がありました。
他にも必要なものがあるかもしれません。
:::

## プラグインの実装 (`lua/example/init.lua`)

プラグイン名が `example.nvim` なのに対して、ここではディレクトリ名は `example` としています。
`init.lua` がエントリーポイントとなります。

ひとまず以下のようにしてみます。

```lua:lua/example/init.lua
local example = {}

example.setup = function(opts)
  vim.print(opts)
end

return example
```

プラグインの大まかな作成の仕方は以上です。

# プラグインを読み込む

## 個人的におすすめの読み込ませ方

開発中の動作確認にも便利なので、最小構成の設定を作成します。

```lua:minimal-init.lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup({
  {
    "nabekou29/example.nvim",
    dir = "~/ghq/github.com/nabekou29/example.nvim",
    opts = {
      message = "Hello, world !",
    },
  },
})
```

`dir` を指定することで、開発中のローカルのプラグインを読み込むことができます。

開発中のプラグインを読み込む方法はいくつかありますが、特に**複数マシンで Neovim を利用している**場合は、次のような設定にするのがおすすめです。

```lua
local opts = {
  dev = {
    path = "~/ghq/github.com/nabekou29",
    -- path = vim.fn.systemlist("ghq root")[1] .. "/github.com/nabekou29",
    -- ↑ このようにしても良いが、nvim の起動速度に影響が出るため、可能なら全マシンで固定のパスを指定するのがよい。
    patterns = { "nabekou29" },
    fallback = true,
  },
}

require("lazy").setup({
  {
    "nabekou29/example.nvim",
    opts = {
      message = "Hello, world !",
    },
  },
}, opts)
```

このような設定でも開発中のプラグインを読み込むことができます。
この方法のメリットは以下の通りです。

- プラグインの設定部分はローカルとリモートのどちらから取得する場合も同じになる
- `fallback = true` を設定することで、ローカルにリポジトリがある場合はローカルを読み込み、ない場合は GitHub から取得するようになる

## プラグインを読み込んで起動

この時点で nvim を起動すると、起動後に以下の画像のようなメッセージが表示されるはずです。

```sh
$ nvim -u minimal-init.lua
```

![minimal-init.gif](/images/neovim-lazy-nvim-development/minimal-init.gif)

# 開発の進め方

基本的にここまでできたら、あとはひたすらプラグインの実装を進めていくことになります。
作りたいプラグインによって必要な情報も大きく変わるので、具体的な情報よりは開発中に気をつけていたことを書いていきます。

私は以下のように進めていました。

## 実装に参考になる情報を集める

### 参考になりそうなプラグインの実装を読んでみる

作りたいものが、コマンドなのか、UIなのか、作りたいものに合わせて、自分がよく使っているプラグインの実装を読んでみると良いと思います。  
ディレクトリ構造だったり、参考になるものが多いです。1つだけでなく多くのプラグインの実装を読んでみると、実装のイメージが掴めると思います。

### ChatGPT に聞いてみる

何すればいいかわからないって詰まった時には、ChatGPT に聞くようにしていました。  
Neovim のプラグインに関する日本語の情報は少ないのか、ググってもなかなか情報が見つからないことが多かったので、ChatGPT はかなり頼りになりました。

### `help` を読む

多分一番重要かつ頼りになるが、サボりがち。  
参考にした実装が古かったり、ChatGPT も最適解を出してくれないことがあるので、最終的に `help` を読むことが大事だと思います。

telescpoe を入れている場合は、`:Telescope help_tags` で検索すると便利です。

`vim.fn.xxx`, `vim.tbl_xxx` など、組み込みの関数や API が意外と便利だったりします。
既存のプラグインの実装だと自前で実装していることもあったりするのですが、割と組み込みの関数で代替できることも多いです。

## 型アノテーションを書く

個人の方針もあると思いますが、がっつりプラグインを作る場合は型アノテーションを書いておくと良いと思います。

参考: https://github.com/LuaLS/lua-language-server/wiki/Annotations

## プラグイン向けのプラグインを使う

lua では LuaRocks というパッケージマネージャーがあるのですが、使わないのが一般的なようです。
ただ、プラグインとして提供されているパッケージのようなものはあります。

よく見るものだと、[plaenary.nvim](https://github.com/nvim-lua/plenary.nvim) があります。

こういったものはそこまで多くないように思いますが、探してみる価値はあると思います。

## 開発中の動作確認・デバッグ方法

### プラグインの再読み込み

- lazy.nvim の機能で再読み込み

  `:Lazy reload {plugin_name}` で再読み込みできます。

  ただし、プラグインの実装次第では理想的な再読み込みができないことがあるかもしれません。基本的にはうまく動くと思います。

- Neovim を再起動

  一番確実な方法。セッションを活用すればバッファの状態を保持できるのでそれほど不便ではなく、基本はこの方法を利用していました。
  普段はセッションはあまり使わないって人は、以下の記事が参考になるかもしれません。（私は試していません 🙇）

  @[card](https://zenn.dev/nazo6/articles/neovim-restart-command)

## プラグインの公開

基本は github にあげるだけですが、追加で [dotfyle.com](https://dotfyle.com/) に登録すると、他の方にも使ってもらいやすくなるかもしれません。

# 終わりに

まとまりのない内容になってしまいましたが、Neovim のプラグイン開発において学んだことを書き留めておきました。
読んだ人に何か一つでも参考になることがあれば幸いです。
