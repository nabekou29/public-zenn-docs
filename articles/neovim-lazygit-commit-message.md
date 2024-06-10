---
title: "Neovim ×  lazygit でコミットメッセージを楽に書く"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "lazygit"]
published: true
---

タイトルは NeoVim × lazygit としましたが、今回紹介する方法は lazygit に限らずターミナルでコミットメッセージを書く場合に使える方法です。
Copilot を利用してコミットメッセージを楽に書く方法を紹介します。

- [lazygit](https://github.com/jesseduffield/lazygit)

# はじめに

私は lazygit を非常に気に入って愛用しています。lazygit を使ううちにターミナル上で作業を完結させるべく Neovim を使うようにもなりました。
しかし少し不満を感じる点として、VSCode や IntelliJ などの IDE に統合された git 操作ではないため、Copilot Chat などの恩恵を受けられないことがあります。
そこで今回は lazygit でコミットメッセージを書く際に Copilot を活用する方法を紹介します。

私はコミットメッセージを書くのが苦手で、特に英語で書くのが苦手です。なので Copilot の力を借りてコミットメッセージを楽に書きたいと思いました。

Neovim から Copilot Chat を利用するプラグイン ([CopilotChat.nvim](https://github.com/CopilotC-Nvim/CopilotChat.nvim)) もあり、こちらでもコミットメッセージを生成するができます。
`:CopilotChatCommitStaged` というコマンドがあり、これを使うことでコミットメッセージを生成することができます。
今回のやり方でも一部プロンプトを参考にさせていただいています。

# やり方

## 1. lazygit の設定

以下のようにカスタムコマンドを追加します。
`C` は元々 `git commit` に割り当てられているキーですが、今回はこれを上書きする形で設定します。

```toml:~/Library/Application Support/lazygit/config.yml
customCommands:
  - key: "C"
    context: "files"
    command: "git commit -v -t ~/.config/git/commit_template_with_prompt.txt"
    description: "Commit changes using git editor"
    subprocess: true
```

- `-v`: コミットメッセージのテンプレートの下に差分の内容を表示するオプションです
- `-t ~/.config/git/commit_template_with_prompt.txt`: Copilot を活用するためのテンプレートファイルを指定します。次のステップで作成します。

## 2. テンプレートファイルの作成

以下のようなテンプレートファイルを作成します。
ファイルの置き場所や中身もお好みで。

```txt:~/.config/git/commit_template_with_prompt.txt
# Write the commit message according to the commit convention. The title should be a maximum of 50 characters and the message should wrap at 72 characters. Wrap the entire message in a code block in the gitcommit language.
# Format should follow Conventional Commits.
# Describe changes at the bottom of this file.

### commit message from here
### subject


### description


### commit message until here


### 日本語 ver
### commit message from here
### タイトル


### 説明


### commit message until here
```

テンプレートファイルにコミットメッセージを Copilot にいい感じに生成してもらうためのプロンプトを記述します。
`### subject` の下の行で生成が動くように調整しています。

リポジトリごとに日本語か英語を使い分けることがあるため、入力場所によって補完される言語を変えられるようにしています。
まだ活用したばかりなので改善の余地はかなりあると思いますが、とりあえずこれで使っています。

## 3. コミットメッセージの入力で Copilot が動くように

git のエディタが nvim になっていない場合 `git config --global core.editor nvim` で設定してください。

[zbirenbaum/copilot.lua](https://github.com/zbirenbaum/copilot.lua) を利用している方であれば以下のように設定するとコミットのエディタでも動くようになります。
lazy.nvim での設定例です。

```lua
{
"zbirenbaum/copilot.lua",
cmd = { "Copilot" },
event = { "InsertEnter" },
opts = {
  filetypes = {
    gitcommit = true,
  },
},
```

# 動作例

以下のようにコミットメッセージを入力すると Copilot が補完してくれます。

![demo.gif](/images/neovim-lazygit-commit-message/demo.gif)

Copilot Chat で生成するとプレフィックスが思っていたのと違ったりすることがあるので、自分でプレフィックスは入力してメッセージは Copilot に生成してもらうのが個人的には好みなのでこの手法は気に入っています。

# おわりに

途中で気づきましたが近いことをやっている先駆者の方がいらっしゃいました。
https://zenn.dev/hasundue/articles/copilot-vim-commit-messages

今回紹介した手法も先日思い付いたものでまだ活用しきれていないので、しばらく使ってみてまた改善したり別の方法を考えてみたいと思います。
