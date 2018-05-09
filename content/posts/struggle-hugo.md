---
title: "GitHub Pages + Hugo で躓いた話"
date: 2018-05-09T00:32:25+09:00
tags: [ "hugo" ]
categories: [ "悪戦苦闘記" ]
draft: false
---

## うまくいかなかった話をしたい
[前回エントリ(link)]({{< ref "start-hugo.md" >}})で *GitHub Pages + Hugo* による HP
開設の話をしました． 

私は OSS を自分で設定したり動かしたりすることは好きなのですが，ドキュメント内の目的箇所
を特定するまでに時間がかかったり，大前提な部分を読み飛ばしたりなどのうっかりをやりがちです．

> このエントリは，私が GitHub Pages と Hugo を使って HP を開設するまでに躓いたことたちを供養するためのものです．

私は<b>*できるひとたちの途中でうまくいかなかった話*</b>
みたいなものに興味があるので，自分もなるべくそれらを残すようにしようと思っています．どんな人もしょうもないことで詰まったりするんだって話が聞けるとなんだかホッとするじゃないですか．俺ツエー話とは別で失敗エントリがあってもいいと思うんです．

### 1. function "default" not defined
```
 function "default" not defined
```
ローカル環境でサイトを生成して手元で確認するときに `hugo -wD server` を実行しますが，その際に Hugo が吐く go のエラーです．
[ドキュメント](https://gohugo.io/functions/default/)を読むとわかるんですが，Hugo には `default` という function が用意されています．
用意されているのに未定義だと怒られるということはおそらく当時パッケージマネージャで入れた Hugo のバージョンが低かったのでしょう．GitHub から Hugo の最新版をクローンして使ってみたら解決されました．
####  A1. 最新版の Hugo を使用する
- - - 
`default` の使い方は以下のように，`Params.seo_title` に値が設定されていない場合はデフォルトである `.Title` が title タグに使われるというふうです．
``` golang
<title>{{ .Params.seo_title | default .Title }}</title>
```
他のサイトジェネレータでも同じことだと思いますが，Hugo はサイトやページのタイトルやタグなどを `Value` として設定します．
ユーザが設定するサイトの baseurl やタイトルなどは `config.toml` に設定します．

また，テーマ側の layout 以下とトップディレクトリの `layout/` 以下に同名ファイル（e.g., `header.html`）が存在する場合はトップディレクトリの `layout/` のファイルが優先（オーバーライド）されます．これを利用して既存テーマを改造することができます．

### 2. 公開ページ用のブランチを `gh-pages` にしたい（できない！）
研究室の HP でも Hugo を使っています．HP のソース（Hugo 的には `public/` 以下）は `gh-pages` というブランチで，そのソースを生成するためのファイル群は `master` ブランチで，同一のレポジトリで管理しています．記事を書いて `master` に push すると CircleCI によってサイトが生成され，それを `gh-pages` に push してくれてページが更新されます．

同様に，自分のサイトでもこのようなブランチの分離をしたくなりました．

私の作業の流れは Hugo Document の Quick Start と Host on GitHub に基づいています．

* [Quick Start | Hugo](https://gohugo.io/getting-started/quick-start/)
* [Host on Github | Hugo](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

ユーザに紐づく GitHub Pages のリポジトリに関して重要なのは以下の2点です．

1. リポジトリ名は `<USERNAME>.github.io`
1. <mark>公開ページ用のブランチは `master`</mark>

2 でハマりました．ドキュメントにしっかり書いてあるのに読み飛ばしたのでしょう．
研究室 HP と同様に `gh-pages` ブランチを公開ページ用に使用できると**勝手に思い込んで**いました．

つまり，`master` に生成元のファイル群を push して公開ページ用のブランチを変更しようとしました．
実際に公開ページ用のブランチを変更しようとするとグレーアウトされて変更できません．
{{< figure src="/img/github-pages.png" title="図1． master branch から変更不可" class="center"  >}}

#### A2. ページ公開用には `<username>.github.io` の <mark>`master` ブランチ</mark>を使う
- - - 
この仕様をきちんと把握していなかったのでした．Hugo で記事を公開するより前に
ページのレイアウトを改造するとき，よくある手順は以下のとおりです．
```shell
$ hugo new site ./site-name # サイトを作成するファイル群の生成
$ cd ./site-name            
$ ls
archetypes/  config.toml  content/  data/  layouts/  static/  themes/
$ git init				
$ git add . &&  git commit -m "initial commit" # master に commit
```
サイトの設定はこれ以降 commit していけば保存されますが，ブランチを新たに作成していないので，`master` ブランチに `config.toml` のような設定ファイルを追加してしまいます．
この状態で「レイアウトができた，記事書いて公開するぞ〜」と意気込んで push すると，**めでたく図１のようになります**．

これを回避するには，なんでもいいので開発用ブランチでも切っておけば良かったわけです．
手順は以下の通り．
```shell
$ hugo new site ./site-name # サイトを作成するファイル群の生成
$ cd ./site-name            
$ ls
archetypes/  config.toml  content/  data/  layouts/  static/  themes/
$ git init				
$ git checkout -b dev		# 開発用ブランチにチェックアウト
$ git add . && git commit -m "initial commit" # dev に commit
```

### 3. 同一リポジトリ上で `public/` 以下だけ `master` ブランチで扱う
Hugo を使う上で，`config.toml` などの設定ファイル群の commit 先として考えられるのは以下の 2 つです．

1. <mark>ページ公開用と同一リポジトリの `master` 以外のブランチ</mark>
1. ページ公開用と別のリポジトリ

Hugo 公式では１を採用しており，Git の機能である `git submodule` で実現しています．
私も Hugo 公式のやり方に従いました．

#### A3. Git Submodule で public/ 以下を別プロジェクトとして管理できる
- - - 
#### git submodule とは
一旦話が Git submodule に逸れます．  
使用目的は公式ドキュメントから引用します．（公式最強）

> あるプロジェクトで作業をしているときに、プロジェクト内で別のプロジェクトを使わなければならなくなることがよくあります。サードパーティが開発しているライブラリや、自身が別途開発していて複数の親プロジェクトから利用しているライブラリなどがそれにあたります。こういったときに出てくるのが**「ふたつのプロジェクトはそれぞれ別のものとして管理したい。だけど、一方を他方の一部としても使いたい」という問題**です。

[Git - サブモジュール](https://git-scm.com/book/ja/v1/Git-のさまざまなツール-サブモジュール) より

Hugo + GitHub Pages の場合を考えると，

1. 設定ファイルや記事の元となる md ファイル
1. `punlic/` 以下
1. `themes/` 以下

をそれぞれ別のプロジェクトとして管理したいわけです．実際，ほとんどの Hugo のテーマが GitHub で管理されているので，自サイトのテーマは submodle で管理するのが自然です．

submodule の詳細については先の公式ドキュメントを，実際の操作や簡単な使い方はこちらの Qiita エントリがわかりやすいです．

* [Git submodule の基礎 - Qiita](https://qiita.com/sotarok/items/0d525e568a6088f6f6bb)
* [Git submoduleの押さえておきたい理解ポイントのまとめ - Qiita](https://qiita.com/kinpira/items/3309eb2e5a9a422199e9)

- - - 
今回のような Hugo + GitHub Pages の場合，`public/` 以下をプロジェクトとして `<username>/<username>.github.io` の `master` ブランチに commit していけば良いわけです．

### 結論
公式ドキュメントが最強なので思慮深く読むべし．
個人ユーザの場合，ページ公開用には `master` ブランチしか許可されてないことをきちんと把握しておくべきでした．
