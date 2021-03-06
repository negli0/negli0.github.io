+++
title = "KLab Expert Camp にチューターとして参加した"
date = 2019-09-02T14:30:09+09:00
draft = false
tags = ["network", "protocol"]
categories = ["engineering"]
+++

## 概要
2019 年 8/26 ~ 8/29 に開催された 第1回 KLab Expert Camp にチューターとして参加
させていただきました．テーマは「TCP/IPプロトコルスタック自作開発」です．
KLab Expert Camp についてはこちらを御覧ください．

- [技術系インターン特別版「KLab Expert Camp」を初開催！: KLab広報ブログ](http://pr.blog.klab.jp/archives/51712424.html)

## 参加までの経緯
6月上旬に主催者である山本さんに Twitter で「めっちゃいいですね」とお伝えしたら
「チューターお待ちしています」とのお返事をいただきました．(ほぼこれだけで決まっちゃった)

{{<tweet 1137027047589421058 >}}

この KLab Expert Camp の告知は山本さんのツイートだけだし，
それにテーマもニッチだし，まあ 4,5 人くらいは集まるでしょと思っていました．


後日打ち合わせで 15 人程度いると伝えられてマジかとなりました．（どこに生息してるんだ）


## Expert Camp の内容
「TCP/IP プロトコルスタック自作開発」という大きなテーマがあり，申込時の希望に
合わせて以下のコースに分かれて 4 日間ひたすら開発を進めます．

- (1) 基本コース：受講者全員が，用意された教材に沿って学習を進めていく
- (2) 発展コース：個人毎に，発展的な課題にチャレンジする

### 基本コース
基本コースでは，山本さんの開発している microps (マイクロピーエス) を題材にし，
山本さんがプロトコルスタックとはなんぞやという話から，各 OS の実装，
microps ではどう実装しているのか，という話を講義形式で進めます．ある程度
まで講義が進んだら手を動かして自分で実装したり動作検証したりします．

microps はこちら．非常にきれいな実装です．

- [microps](https://github.com/pandax381/microps)

### 発展コース
発展コースでは，参加者の作りたいものを作ります．
今回は 4 名からの希望で，各自以下の内容に取り組んでいらっしゃいました．

- L3 まで microps の写経をしながら改善 + L4 を再実装する
- Rust で TCP のメカニズムと Socket-like な API を実装する
- microps 上にルーティングプロトコルを乗せる
- Rust で L2 ~ L4 をフルスクラッチする

みなさん偶然異なるテーマになって面白い．ちなみに私は主に発展コースの
参加者からの質問に答えたり，質問に答えるための調べ物（あとハッシュタグつきで
Tweet！）
をしていました．

## お役目 (本題)
基本的に私のお役目は発展コースの方の質問に答えることなのですが，私は研究者
の立場から参加者の方々になにか伝えられたらいいなと思っておりました．
4日間みなさんが作っていくものがどういう研究につながっているのかとか，今の
インターネットを取り巻く情勢（クラウド事業者や ISP）の話とか，なぜ IoT が進んで
行かないのかとか，5G って何が変わるの？とか，クラウドに対するエッジとは？とか．


プロトコルスタックは1台のマシン上における比較的ミクロな世界なわけですが，
それが大規模につながるとこんな世界が広がっているんだよ〜ということを伝えようと
懇親会でたくさん喋っていました．参加者用 Slack の雑談チャネルにおもしろ論文を
ブンブン投げたりもしました．将来有望な学生たちに種を蒔きまくっていました．
全然時間足りなかったけど．


私がいる意味ってこういう話を楽しくして，研究おもすれ〜とか，インターネットの世界
すげ〜とか思ってもらうことだと思ってたんですよね．なのでもし今回私の話が
コンピュータネットワークやインターネットを専業にするぞというきっかけに（ほんの
ちょっとでも）なってもらえたら嬉しいです．もしかしたら一緒にお仕事をするかも
しれない．というか，別に専業にしなくてもいつかどこかで「あ〜 nelio
ってやつがなんか言ってたなぁ」ぐらいに思ってもらえれば私はお役目を果たせたと思います．
遅効性だね．

## 感想
Twitter でもつぶやきましたが，参加者のみなさんの集中力が本当にすごかった．
基本コースも発展コースもみんな黙〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜って
ひたすら黒い画面と向き合ってプロトコルスタックかりかり作ってるんですね．

あと，みなさん言語化能力と現状把握能力に長けていて，質問が的確でした．
とても質問に答えやすかったです．私が知らないことや盲点だったこと
に気付かされることもしばしばあり，非常に勉強になりました．


今回の様子が気になる方は Twitter で 
[#KLabExpertCamp](https://twitter.com/hashtag/KLabExpertCamp?src=hashtag_click&f=live)
[#プロトコルスタック自作](https://twitter.com/hashtag/プロトコルスタック自作?src=hashtag_click)
のハッシュタグを追っていただけると良いと思います．


もし次回があればまた参加したいなと思えるほど素晴らしいイベントでした．

山本さんをはじめとする関係者の皆様，ありがとうございました．
