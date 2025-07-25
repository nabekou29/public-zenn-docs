---
title: "Unipos の Elm から React (Next.js) へのリアーキテクチャプロジェクトを振り返る"
emoji: "🌳"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Elm", "React", "Nextjs", "リアーキテクチャ"]
published: true
publication_name: "unipos"
---

**Unipos では、約2年間にわたり Elm から React (Next.js) へのリアーキテクチャプロジェクトを実施しました。**
そして、2025年3月 をもって完全に置き換えが終わりました 🎉

今回の記事では、

- リアーキテクチャをする前の **開発者としての個人的な違和感やモチベーション**
- プロジェクトを通しての **よかった取り組みや学び**

について書いていきます。

現状の開発基盤に不満を持っている方や、リアーキテクチャに興味がある方にとって、何かしらの参考になれば幸いです。

:::message
文章中で「リアーキテクチャ」と書いている部分は、基本的に今回のプロジェクトのことを指しています。
ただし、今回のプロジェクトでは内部的な変更に留まらず、機能やデザインの整理もしたため、一般的な「リアーキテクチャ」とはズレるかもしれません。
:::

## リアーキテクチャプロジェクトの概要

プロジェクトの全体像です。

- 📅 **期間**
  - **2023/01~2025/03 の約2年間**。（技術検証などの期間を含む）
- 📏 **規模**
  - 実装における Elm のコードは LOC で **200,000行** ほど。
  - アプリのページ数は、ページ用のファイルが 100以上。(細かいページも含むので参考程度に)
- 🛠️ **プロジェクト体制**
  - 基本はエンジニア **2名**。私ともう1人。
  - 職能問わず、多くのメンバーの大小さまざまな協力
- 🏗️ **アーキテクチャの移行**
  - 旧アーキテクチャ: **Elm + The Elm Architecture**
  - 新アーキテクチャ: **TypeScript + Next.js** (Pages Router)
- 📦 **大まかな内容**
  - Next.js を利用した新しい開発基盤・アーキテクチャの構築
  - 既存のページを Next.js のものにリプレース
- 🔄 **置き換え方法**
  - **ストラングラーフィグパターン。** 1〜数ページ単位で置き換える。

## ✅ リアーキテクチャをする決断・なぜ必要だったか

### 🌱 個人的な違和感・モチベーション

私は、リアーキテクチャプロジェクト以前は、機能開発チームに所属し既存の改修や新機能の開発をしていました。

開発にも慣れて、いくつかの機能の開発を経た頃に、**「エンジニアの工数が原因で要件落としたりすること多いな…」** と思うようになりました。

実際に機能開発チームの中のコミュニケーションの中では以下のようなやり取りは珍しくなかったです。

機能開発の場:

> _Eng「今の仕様だとXX日かかってしまいそうです」
> Des「それならこの機能は今回のスコープから落とそうか」_

社内や顧客からのフィードバックを経て:

> _XXX「ここの機能を修正してほしい」
> Eng 「けっこう時間かかりそうで、あまり費用対効果良くないですね」
> 　　　（あるいは）「実装が複雑になるので、できれば避けたいです」_

開発速度の低下が、結果的に以下のような状態を招いてしまっていたと思います。

- **作るべき機能の優先順位が「実装しやすさ」ベースで決まってしまう**
- **組織全体がプロダクトに対する大きな変更をためらってしまう**。（これは個人の感覚ですが、そういう空気がなかったとも思わない）
- 細かい機能改善がなされない

このような状況が**プロダクトを通して成し遂げたいビジョン実現の妨げになっているなと感じていました。**

少なくとも、この時点で **開発速度を上げる改善** が必要だと強く思うようになりました。

## 💚 リアーキテクチャの目的を達成するためにやって良かったこと

### 🛠️ ただつくりなおすことはしない

一般的なリプレイスは、現行の仕様や機能、デザインをそのまま新しいシステムに引き継ぐようなイメージがあります。
しかし、今回のプロジェクトでは **仕様・デザイン・機能の整理** も同時に進めました。

---

「どうせ作り直すなら…」という風にいろいろ盛り込むのはスケジュール遅延などの要因になるため、基本はアンチパターンになると思います。

そのため、基本は現行を引き継いだ置き換えとしつつ、以下のようなものは対応することに決めました。

- 不必要なまでに複雑になっている仕様
- ほぼ使われていないが残っている機能
- 一貫性のない仕様・UI

これらは、削除・改善することで、スケジュールの短縮ができたり、実装もシンプルにすることもできました。

### ⚡ 難しいところから攻める

ここでの「難しいところ」とは、技術的負債が最も溜まっている箇所です。

- **実装が絡み合っている**
- **影響箇所が大きい**
- **不明な仕様が多い**

攻める理由は2つ：

1. **新アーキテクチャでの開発を早期に進められるようにするため**
2. **スケジュールの見通しを早期に立てるため**

とはいえ、それらは最初にリリースするには不安も大きい箇所なので、実際には別の箇所を先にリプレイスしました。

ただし、最初のページを選ぶ際には、万が一の影響が少ないことは前提に、**難しいページとの共通項が多いものを選択**しました。

そして、プロジェクト開始から**1年ちょっとで目指していたページの置き換えが完了**しました。

結果的には以下のような良い影響がありました。

- 1年の時点で新機能開発はほぼ新しいアーキテクチャで実装できるようになり、恩恵を受けられた。
- リアーキテクチャ後の資産が十分にない管理画面などは、機能開発チームを巻き込んで新機能開発ついでに置き換えられた。
- 後半の1年は、ある程度スケジュールの見通しが立っていたので、付加価値を付けられるような動きができた。

今でこそプロジェクトのすべてが完了していますが、なんらかの要因で途中でリプレイスが停止してしまうようなことも頭の中では想定していました。

そのようなことを考えても、**置き換えた際に価値のある場所から置き換える**というのは非常に良い選択だったと思います。

## 📝 反省

### 見積もりとスケジュールについて

初期の初期の段階では以下のような見積もりを立てていました。

（あくまでイメージです）

![初期のスケジュールイメージ](/images/unipos-rearchitecture/schedule-expected.png)
_必要最低限の基盤を作ったあとは、一定のペースでリプレイスできると考えていた_

必要な基盤をコンパクトに作って、ほかは実際のページを置き換えながらいろいろ整えられるだろうと雑に考えていました。

---

しかしながら実際には、以下のようなスケジュールになりました。

![初期のスケジュールイメージ](/images/unipos-rearchitecture/schedule-actual.png)
_実際には9割のページは後半の1年で置き換えられた_

まず、最初のリリースまでに半年以上かかっています。

また、難しいところから攻めたことも相まって、折り返しの1年時点では**全体のページ数でみれば全然終わっていないように見えてしまって**いました。

開発基盤やデザインシステム、全画面共通の機能などの規模感の想定が甘かったです。
またそれらは影響範囲が大きいため、より丁寧に作る必要があり時間がかかる要因になりました。

- 学び
  - 目に見えるアウトプットやプロダクトの変更点としてページ起点でコミュニケーションとるのはOK
  - ただし、スケジュールやエンジニアに対しての提供価値としては、実装基盤や共通機能も含めて考える必要がある

### リアーキテクチャをもっと早くやればよかった

リアーキテクチャプロジェクト以前、私が機能開発チームに居た頃から、開発効率アップのための改善もいくつかやっていました。

しかし、結果として十分満足できるほどの効果はなかったですし、もう大きな効果がある改善をするのは無理だという判断でリアーキテクチャを決行しています。

全てを置き換えた今となっては、「あの改善は無駄だったんじゃないか」と思うものもあります。

今の体制や環境でできることだけを考えていて、[個人的な違和感・モチベーション](#🌱-個人的な違和感・モチベーション)で書いたような問題を解決するためにはどうすればよいかを考えられていませんでした。

リアーキテクチャのような大きな改善も選択肢に含めた上で、どのような改善をやるのか/やらないかを考えられると良かったなと思います。

![改善アイテムを整理したチャート](/images/unipos-rearchitecture/value-effort.png)
_改善アイテムを費用対効果で整理した際に、大掛かりなものは候補に乗らなかった_

## 🎯 リアーキテクチャの結果

エンジニア目線では、以下の点で **顕著な効果** を感じられるようになりました：

- 🚀 **実装スピードの向上**
- 🔧 **メンテナンス性の向上**

![エンジニアからのフィードバック](/images/unipos-rearchitecture/engineer-fb.png)
_開発メンバーからの嬉しいフィードバック。
プロジェクト開始して１年ちょいでの社内発表資料より抜粋。_

---

また、エンジニア以外からも以下のようなフィードバックが得られました：

- 💬 **「できることが増えて、これがベストだと思う価値を諦めることがなくなった」**
- 💬 **「フロントエンドの改修の腰の重さが減った」**

開発スピードの向上を通して、**開発組織全体の意識や文化が変わりつつある** ことを感じています。  
個人的に目指していた方向性だったので、やれて良かったと思っています。

## おわりに

### 💭 個人的に Elm をやめたことについて思うこと

Elm をやめたことは、開発組織のための前向きな決断で実際良かったとも思っているのですが、一方で少し寂しさもあります。

Elmのような挑戦的な技術をプロダクトで使っていたことは、組織としての価値観やカルチャーの表れであり、ある種のアイデンティティでもあったと思っているからです。

言語的な機能で見たときも、React x TypeScript の組み合わせは全体的にみれば満足度は高いのですが、細かく見るとツラい部分もあります。

`undefined/null`、例外、破壊的な関数、暗黙的な型変換、どれも Elm にはないものです。

実際、これらによって問題が起きた記憶はそこまでないですが、実装やレビューのたびに精神はすり減っていきます。

`Result/Maybe` 型、パターンマッチ、シンプルで厳格な型システムが恋しくなることも多いです。

そうに感じるのは、Elm 時代に培った「型への意識」が残っているからだと思います。

これは自分だけに限らず、少なからず今の開発メンバーにも、良い文化として残っていると感じます。
その文化を損なわないまま、Reactの柔軟性やスピード感のある開発ができていけば良いなと思っています。
