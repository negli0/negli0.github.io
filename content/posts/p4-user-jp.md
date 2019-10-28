+++
title = "日本 P4 ユーザ会 2019 参加記（まとめや感想など）"
date = 2019-10-28T09:23:48+09:00
draft = false
tags = ["p4","protocol"]
categories = ["engineering","event"]
+++

## 日本 P4 ユーザ会とは
ホームページによれば，以下のとおりです．

- [日本 P4 ユーザ会](https://p4users.org)

> [P4 Lang](https://p4.org) について日本語で語るグループ．
P4 関連のセミナー情報やカンファレンス情報及び技術情報を共有する．

インタラクティブな議論は Slack 上でされています．


10/11(Fri) に「[日本 P4 ユーザ会 2019](https://p4users.org/2019/07/16/event2019/)」
が開催され，私にしては珍しく聴講のみのイベント参加をしてまいりました．
一緒に行った友人との議論もあって知見が溜まった気がするので吐き出します．

##### ※本エントリでは P4 についての詳細は省きます

### P4 と私
存在自体は論文[^p4-ccr] を読んで知ってから久しいものですが，具体的に動かしたことはないです．
Software Defined Network（SDN）の文脈で言われる，いわゆる "Data-Plane(D-Plane) Programmability"
を実現するための Domain Specific Language（DSL）です．

- [論文PDF](https://www.sigcomm.org/sites/default/files/ccr/papers/2014/July/0000000-0000004.pdf)

D-Plane Programmability といえば，私は 2017 年に某所で 
OpenFlow[^openflow]（Controller は Ryu）， Docker， Open vSwitch を組み合わせて軽量な 
NFV box を作ってデプロイして面倒を見るという作業に参加していました．手前味噌
ですが，運用成果をまとめた論文[^clnfv] を研究会に出してます．


OpenFlow[^openflow] ではヘッダフィールドを規定してパケットプロセッシングする一方，
P4 はビットストリームとしてパケットプロセッシングするのだな，ぐらいのイメージを
持っていました．加えて P4 は技術的な特徴からエコシステムまで含めて，OpenFlow 
でうまく行かなかった部分を克服しようとしているように思っておりました．

 
ここから当日得た内容を私なりにまとめてみます．資料は HP 上で公開されています．
P4 の基本的な仕組みについて触れた資料もあるため全部読めば知識は手に入ります．

- [【資料公開】日本 P4 ユーザ会 2019 開催のお知らせ](https://p4users.org/2019/07/16/event2019/)


##### ※ なお発表内容すべてには触れませんが特に意図はありません

---


感想は以下のようにまとめられます．

1. Stratum で P4 プログラムを扱いやすく
2. 'fully-programmable' のつらさと品質保証
3. INT 大流行
5. 研究の観点
6. P4 に限らずエコシステムを回すことが大切

順番に書いていきます．

### Stratum で P4 プログラムを扱いやすく
実際に P4 を使ったある開発経験者から聞くところによると，Barefoot の SDK を
直接使うのはけっこう辛いのだそう．加えて私のように手元でちょっとした実験をしたい
人からするとわざわざ P4 対応の ASIC を購入するのはハードルが高いです．


Stratum は ONF や Google からオープンソースとして9月に公開された，いわゆる NOS です．
P4 対応 ASIC だけでなく，FPGA や NPU，CPU も P4 の実行プラットフォーム
として扱うことができます．P4 ネイティブ非対応のプラットフォームにも P4 プログラム
をデプロイして使えるのは非常にありがたいです．

- [Stratum - Enabling the era of next-generation SDN](https://github.com/stratum/stratum)

各プラットフォームごとの主なターゲットは以下のとおりです．
#### ※ ebiken さんのスライド参照: [PDF](https://p4users.files.wordpress.com/2019/10/p4ug-japan-toyota-ebisawa-20191011.pdf)

|プラットフォーム|ターゲット|処理性能|コンパイラ|
|:-:|:-:|:-:|:-:|
|ASIC| Barefoot | xx Tbps | P4 Studio |
|FPGA| Xilinx, Intel | xxx Gbps | P4-SDNet, Netcope NP4 |
|NPU| Netronome | xx Gbps | Agilio P4C SDK |
|CPU| BMv2, eBPF | x Gbps | p4lang/p4c |

P4 を手元でサクッと試すなら Stratum + BMv2 が良さそうです．
"PoC 程度なら" 同じ P4 ファイルをマルチプラットフォームで使い回せます．
それがP4の意義のひとつですからね．

ASIC は高スループットですが P4 で表現可能なことの他にできることが限られます．
例えば最新の P4\_16 の仕様では P4 には除算命令が無いらしいです．
FPGA では，P4 で表現できないロジックは HDL でどうとでもなりますが，ロジックを変更
するたびに合成が必要なので規模に寄ってはアジリティは低いのかもしれません．
NPU ですが，私は使ったことがありません．おそらくだいぶソフトウェア寄りな表現が
可能になると思います．価格もそこまで高くないそう(7万円とか)です．


Stratum とは別に，
最近だと P4 で D-Plane ロジックを書いてコンパイルすると，それ専用の CLI も一緒に
生成されるらしい(要出典)ので，C-Plane が CLI 
で良ければコントローラの開発コストも下がりそうですね．P4 の API を提供する 
P4Runtime は gRPC を採用しています．gRPC のインターフェイスに即していれば自由に 
C-Plane アプリ（コントローラ）を作れますし，NETCONF/YANG （xmlRPC）
のポートも生やせそうですね．


### ‘fully-programmable’ のつらさと品質保証
P4 を使えば完全に D-Plane をフルスクラッチ可能です．つまり，IP routing や
ACL や forwarding といったよく使う機能も自分で作る必要があります．もちろん
サンプル P4 ファイルがいくつか提供されることもありますが．
あと，P4 がキャリアなどで商用利用されることを想定すると品質保証が大切だと考えられます．

例えば最近では OSS のソフトウェアルータによる DC Networking が注目されていますが，
オープンであるがゆえに開発と品質保証を両立するのは大変です．だからこそ 
Cisco や Juniper といったプラットフォームベンダは頑張っているわけですよね．
プラットフォームベンダの方々は一番要件が厳しいところ（おそらくキャリア）向け
を想定して開発をしているため，こうした方々の品質とユーザによる 'fully-programmable' な
P4 部分をどう両立するかが大事なわけです．


佐藤さん(Cisco)の資料の pp.18-21 に重要な図が載っています．
##### ※ 佐藤さんのスライド参照: [PDF](https://p4users.files.wordpress.com/2019/10/cisco.pdf)

かいつまんで説明すると，品質保証がされた基本的な P4 ロジックをプラットフォームベンダ
（Cisco）が提供し，ユーザは独自で P4 ロジックを用意します．
加えて，プラットフォームベンダが開発環境として Constraint Checker も提供し，両者に機能重複が
無いかどうかチェックします．


これなら IOS のように品質保証された基本的なロジックと P4 のプログラマビリティ
の両立が目指せます．この Checker が何をどこまでやってくれるかに期待がかかります．


Cisco がチップを Barefoot のようなチップベンダから購入する
（＝自社製品向けにチップを内製しない）のも然り，ユーザ拡張のための
Checker も提供してくれるというのも然り，本気で P4 を商用利用しに行こうという
気合を感じます．Net One Systems（インテグレータ）からも会場で
ブースが出されていたことも同様のことを感じさせます．


### INT 大流行
P4 ロジックだからこそのユースケースですが，In‐band Network Telemetry (INT) 
を挙げる方がほとんどでした．その目的で面白かったのが，キャリアにおける SLA の
可視化です．例えば，5G ネットワークを提供するテレコムキャリアが，5G で謳われる
超低遅延，広帯域，同時多数接続といったサービスが本当に実現できているのか確認する
のに期待されています．

##### 武井さん(NTT研究所)のスライド参照: [PDF](https://p4users.files.wordpress.com/2019/10/p4e383a6e383bce382b6e4bc9ae799bae8a1a8e8b387e69699ntte6ada6e4ba95.pdf)

INT では，INT のドメインに入るときにネットワーク機器が
メタデータ（タイムスタンプとか）をパケットに追加し，
ドメインから出るときに機器がメタデータを取り外してテレメトリサーバに計算結果
（パケットがドメイン内にいた時間とか）を送信します．これにより，パケット単位の
SLA 可視化が理論上は可能です．


課題は時刻同期にあります．IEEE 1588 で定義される PTP はサブマイクロ秒のオーダー
で時刻同期ができますが，キャリアが持つような広域ネットワークへの適用は現状困難です．
この規模だと NTP が候補になりますが，時刻同期がミリ秒オーダーにまで荒くなります．
SLA と一口に言っても色々あるので，NTP でも保証ができるレベルであれば十分に実用が
期待できる手法だと思います．


INT 以外のユースケースは後述の IRTF COIN RG でも議論がされています．

##### 研究の観点
研究と言っても P4 の応用研究です．P4 自体の研究は私はちっとも詳しくありません．


IETF がインターネット周辺の技術の標準化を担う団体である一方，IRTF もう少し研究
寄りの団体です．IRTF には Computing in the Network (COIN) という Research 
Group (RG）があります．

- [Proposed IRTF Research Group: Computing in the Network (COIN)](https://trac.ietf.org/trac/irtf/wiki/coin)

今回の発表では特に触れられていませんが，COIN においても P4 の活用が議論の対象
になっています．
焦点がよく当たるのは，一般的なアプリケーション処理をネットワーク内
にどのように透過的に落とし込むかという点です．
よく混同されますが，”In-Network Computing” と ”Network Computing” は別の概念です．
というか私も人に言われるまで特に気にしていませんでした．

{{< tweet 1173781090193920000 >}}


In-Network Computing はデータをよりビットストリームとして扱っている一方で，
Network Computing はデータをミドルウェアやプロトコルをベースに扱っていると考えて
良いでしょう．


私が取り組んでる研究のひとつに，一般的なアプリケーション処理を地理的に
分散した計算機にオフロードするというものがあります．当研究では
5Gネットワークを想定して Network Computing 寄りの考え方を採用しています．
国内研究会には論文[^afc] を出している（手前味噌v2）ので良かったら読んでみてください．
国際会議は目下取り組み中です（はよ出せ）．



仮に In-Network Computing な手法をアプリケーション処理に適用する場合は NPU ぐらいの
柔軟性を設けたほうが良いでしょうね．使い方も D-Plane Programmability というよりは
ヘテロジーニアスコンピューティングやオフローディングという文脈でしょうか．
現状 P4 をアプリケーション処理一般に適用する試みはあまり見ないですね．


あと何でもかんでも Hardware Offloading するのも良くないですよね，ソフトウェアと
違って刺さっていればデータが来なくても多少なりとも電力消費するので，ピークが
立つときには有効な手段ですがアベレージが低いと HW でやる意味が薄いですよね．
特にデータセンタは低消費電力に努めたいわけですし．


### P4 に限らずエコシステムを回すことが大切
今回のイベントで，関わる人がみなで盛り上げるということの大切さを感じました．
そもそもインターネットの世界はマルチステークホルダーで，それぞれの立場のひとが
同じ方向を向けるというのは結構難しいのです．ネットワーク工学の研究をしていると
特に感じます．パワーゲームに勝てば良いのかも
しれませんが，私が目指すのは P4 のように多くの人が嬉しい技術です．


## まとめ
とても濃厚な時間でした．台風接近中だと言うのに 150人近い参加者だったそうで，
そこからも P4 の注目度が伺えます．
あと一緒に参加した友人x2 が P4 に詳しすぎたのでお昼ご飯食べてるときも私の稚拙な
解釈を聞いてくれました．おかげで何となく P4 を取り巻く世界（主に産業寄り）が
見えました．私のようにネットワーク工学を専攻する学生は産業界の話に積極的に
参加したほうが良いと思いました．Slack にも参加したのできちんと議論に参加して
いきたい．

Stratum+BMv2 くらいさっさと動かしたいので APRESIA の桑田さんの資料を参考に
がんばります．あと研究室に Netronome NPU があるのでそれも Stratum から
触れるようにしたいですね．（研究とは直接関係ないのだけどね）

##### 桑田さんの資料参照: [PDF](https://p4users.files.wordpress.com/2019/10/e697a5e69cacp4e383a6e383bce382b6e4bc9a2019_apresia_v.0.4.pdf)


[^p4-ccr]: Pat Bosshart, Dan Daly, Glen Gibb, Martin Izzard, Nick McKeown, Jennifer Rexford, Cole Schlesinger, Dan Talayco, Amin Vahdat, George Varghese, and David Walker. 2014. P4: programming protocol-independent packet processors. SIGCOMM Comput. Commun. Rev. 44, 3 (July 2014), 87-95. DOI: https://doi.org/10.1145/2656877.2656890
[^clnfv]: 早川 侑太朗, 渡邊 大記, 岡田 和也, "clnfv：コンテナを用いた軽量なNFVアーキテクチャ," 信学技報, vol. 117, no. 303, NS2017-111, pp. 1-6, 2017年11月. [研究会ページ](https://www.ieice.org/ken/paper/20171116tbzw/)
[^openflow]: Nick McKeown, Tom Anderson, Hari Balakrishnan, Guru Parulkar, Larry Peterson, Jennifer Rexford, Scott Shenker, and Jonathan Turner. 2008. OpenFlow: enabling innovation in campus networks. SIGCOMM Comput. Commun. Rev. 38, 2 (March 2008), 69-74. DOI=http://dx.doi.org/10.1145/1355734.1355746
[^afc]: 佐藤 友範, 渡邊 大記, 林 和輝, 近藤 賢郎, 寺岡 文男, “5Gコアネットワーク向けアプリケーション処理連接基盤”, 研究報告マルチメディア通信と分散処理 (DPS), Vol. 2019-DPS-180, no. 18, pp. 1 - 8, 2019 年 9 月. [論文リンク](https://ipsj.ixsq.nii.ac.jp/ej/?action=pages_view_main&active_action=repository_view_main_item_detail&item_id=199311&item_no=1&page_id=13&block_id=8)
