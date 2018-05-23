---
title: "Intruducing Enhanced Communication Paths over Layer-4 (Multipath)"
date: 2018-05-21T01:24:15+09:00
tags: [ "layer-4", "multipath" ]
categories: [ "研究紹介" ]
description: "OSI参照モデルのL4以上における高機能通信路のモデル（今回はマルチパス）を紹介します"
draft: false
---

## 概要
インターネットが発展するにつれて，アプリケーションが多様化してきました．
多様化したアプリケーションはネットワークに**高機能性**を要求します．
本記事で述べる「高機能性」とは，マルチパス，耐遅延性ネットワーク，ミドルボックスを
用いた通信路のようなものを指します．<u>本記事の目的は，こうした高機能な通信路のモデル
を具体例を用いながら紹介し，通信路のモデルの整理をする</u>ことにあります．

研究紹介のカテゴリにしておきながら，今回はあまり踏み込んだ話はしません．
前提知識はあまり必要としませんが，TCP/IP の基本的な仕組みを知っていると尚良いと思います．

#### おことわり
本当はマルチパス以外も書きたかったのですが，長くなりすぎるので今回はマルチパスだけです．


## 高機能通信路
本記事では，

- 通常のTCP/UDPでは提供が困難な機能を持つ Layer-4（OSI参照モデル）以上の通信路

のことを高機能通信路（Enhanced Communication Paths / Highly Functional Communication Paths）[^1]
と呼ぶことにします．特に断りがない限り，階層はOSI参照モデルにおけるものを指します．つまり Layer-4（L4）
という記述は OSI 参照モデルのトランスポート層を指し，L7と言ったら OSI 参照モデルの
アプリケーション層を指します．

高機能通信路のモデルとして紹介するのは以下の3つです．

1. **マルチパス**
1. **耐遅延性ネットワーク（Delay/Disruption Tolerant Networking; DTN）**
1. **ミドルボックスを用いた通信路（ミドルボックス通信路）**

それぞれ通信路の使い手（多くの場合上位プロトコル，つまりL7）に以下のような高機能性を提供します．

| マルチパス | DTN | ミドルボックス通信路 |
|:----------:|:---:|:--------------------:|
| 耐障害性，帯域集約 | 対遅延性，対間欠性 | 中継地，ポリシ適用 |

##### 余談：プロトコルと階層構造における位置付け
ここで "OSI参照モデル" と明記している理由は，プロトコルとその位置付けに絶対は無いからです．
そのため，XXというプロトコルは OSI参照モデルでいうと第N層だよね，TCP/IP プロトコルスイート[^tcpip] でいうと第M層だよね，
というような言い方が個人的には適切かと思います．大抵の場合は開発者や文書の著者が第X層に位置すると言います，
IETFに関しては，以下のような記述が Wikipedia にありますね．（一次ソースが発見できませんでしたが）

> IETFは7層からなるOSI参照モデルに従うような試みはせず、また標準化過程 (Standards Track) にあるプロトコル仕様やその他の構造上の文書をOSI参照モデルに対して参照する事もしない。...（中略）...  IETFは再三にわたりインターネット・プロトコルと構造の開発はOSI参照モデルに準拠する事は意図しないという事を述べている。

<font size="2">[インターネット・プロトコル・スイート - Wikipedia](https://ja.wikipedia.org/wiki/インターネット・プロトコル・スイート) より（2018/05/21 時点で確認）</font>

階層構造を採用する理由のひとつとして，開発効率を上げて互換性や再利用性を高めるというものが挙げられます．
各プロトコルではあまり複雑なことはせず，抽象度の高いことは上位層に任せる方が理にかなっていると言えそうです．
- - - 

### マルチパス
一般的にマルチパス通信路は，使い手が複数の通信路を扱える通信路のことです．
上位層（L7）に提供されるのは帯域集約と耐障害性です．
```
耐障害性(Failt Tolerance)： 通信路のうちのひとつが遮断されても通信を継続する
帯域集約(Bandwidth Aggregation)： 複数の通信路を束ねてスループットを高める
```
L4 以上のマルチパス通信路として代表的なものに Stream Control Transmission Protocol（SCTP）[^sctp]
と Multipath TCP（MPTCP）[^mptcp]，最近では Multipath QUIC（MPQUIC）[^mpquic] があります．

#### Stream Control Transmission Protocol (SCTP)
高機能性の説明に入る前に，SCTP の基本的な考え方を説明します．図1にSCTPの通信の概念図を示します．
{{< figure src="/img/sctp-assoc.png" title="図1. Communication Concept of SCTP" class="center"  >}}

- Association：SCTP の通信路の単位．ひとつ以上の stream から構成される．
- stream：Association 内の独立した論理的なチャネル．

SCTP では，Association という単位で相手（Peer）と通信路を確立します．Association 確立時に，
この Association 内で使用したい stream の数と使用可能な IP アドレスのリストを交換し合います．
エンドホストはひとつ以上の IP アドレスを持っていればひとつのポート番号で SCTP Association を確立できます．
stream は Association に独立して属するだけで，実際に通信路を確立するわけではありません．

SCTP では，TCP とは異なりメッセージ単位でデータを扱います．
上位層は，用途に応じて stream を指定できます．例えば，制御メッセージを stream 0 に，
画像データを stream 1 に，それぞれ指定して送受信できます．
このように，ひとつの通信路の中に小さい通信路がいくつも存在するかのような通信をマルチストリームと呼びます．
SCTP のマルチストリーミングは Head-of-Line Blocking（HoL Blocking）が発生しない点でも優れています．

WebRTC では L4 の一部に SCTP を採用していますが，G社は将来的に QUIC に置き換えたいのでしょうね．

##### 耐障害性
SCTP は，エンドホストが複数のエンドポイント（ほとんどの場合 IP アドレス）
を持つマルチホーム環境をサポートします（図2）．
{{< figure src="/img/mh-sctp.png" title="図2. Multi-homed SCTP" class="center"  >}}
SCTP には <b>Primary/Secondary Path</b> という概念があります．ひとつの Association には，複数の
IP アドレスを紐付かせることができます．
両エンドホストは Association 確立時に使用可能な IP アドレスのリストを交換しているので，互いの
IP アドレスからの疎通性を確認できます．これにより，Association 内に IP 
アドレスと紐づく通信路（Path）を認識できます．標準的な SCTP では，複数の Path 
が存在する場合，ひとつを Primary Path としてデータ転送に使用し，残りを Secondary Path
として待機させます．
SCTP では，Path ごとに定期的に Heartbeat をやり取りして障害を検知します．

マルチホームな SCTP では，Primary Path に障害が発生した場合，上位層に透過的に Secondary
Path に切り替えて通信を継続します（failover といいます）．これが SCTP の耐障害性です．

##### 帯域集約
標準的な SCTP には帯域集約は存在しませんが，Concurrent 
Multipath Transfer（CMT）[^cmt-sctp] という拡張が存在します．CMT-SCTP では，複数の
Path が存在する状況で同時に（simultaneously）それらを使用します．

少し話が逸れますが，マルチパス通信，特に帯域集約を効率的に実施するのは意外と難しいです．
一般的に，パス間の輻輳制御や再送などがシングルパスのそれよりも複雑になります．

#### Multipath TCP (MPTCP)
MPTCP[^mptcp] は TCP コネクションを L4 内で多重化して資源利用率を高め，冗長性を高めることを目的とします．
{{< figure src="/img/mh-mptcp.png" title="図3. MPTCP with 4 subflows (fullmesh)" class="center"  >}}
図3に示すように MPTCP では最大で ||src IP|| × ||dst IP|| 個の異なる経路の Path を
保持します．この Path を subflow といいます．

{{< figure src="/img/mptcp.png" title="図4. Structure of MPTCP" class="center"  >}}
また図4に示すように，上位層からは通常のソケットAPIで TCP のように操作ができ，
下位層（L3）では subflow，つまり通常の TCP コネクションとして見えます．MPTCP は，
特別な操作を抜き[^conf-mptcp] に上位層に TCP を束ねた通信路を提供します．

TCP とアプリケーションの間に位置するので L5（L6？）のようにみえるかもしれませんが，
MPTCP は 通常の TCP の動作する領域を出ない範囲で機能していると見えます．このことから
MPTCP は L4 に位置するものとして解釈しています．

Linux MPTCP Project[^mptcp-official] のページによれば，MPTCP では以下の設定を変更することで 
Path の扱い方を決めることができます．今回は詳細を省きますが，詳しい人は設定値の名前からなんとなく動作が把握できると思います．

```
path-manager: default, fullmesh, ndiffports, binder から選択
scheduler: default, roundrobin, redundant から選択
```

##### 耐障害性
MPTCP では，Path は subflow という概念（実体は TCP コネクション）で存在します．
MPTCP コネクション内のすべての subflow が切断された場合に，MPTCPコネクションが切断されます．
また，scheduler を redundant に設定すると，すべての subflow が同じデータを運ぶ
（冗長）ようになります．いずれかの subflow からデータが届けば良いので，重複したデータは
MPTCP で破棄します．
これらが MPTCP の耐障害性です．


障害検知の仕組みがあまり詳しくわかりませんでしたが，実装上は TCP コネクションの切断を
MPTCP で検知，吸収しているのでしょうか？
RFC 6834[^mptcp] には，Failure Handling という項が Heuristics の節に含まれています．
TCP でも keepalive という拡張の仕組みで切断を検知できますが，デフォルトで向こうになっていることが多いです．
<mark>MPTCP の障害検知あたり，もし詳しい方がいらっしゃれば教えていただきたいです．</mark>

##### 帯域集約
MPTCP では，scheduler の設定に従って subflow を同時に使って帯域集約をします．
例えば roundrobin の場合は subflow を一回ずつ変えながらデータを送信します．
subflow ごとに輻輳制御が働くので，MPTCP 全体で効率的に輻輳制御をするのは複雑になります．
SCTP と異なり，標準の MPTCP は上位層からはシングルパス TCP として見えるので，subflow 
を指定した送信はできません．このため，HoL Blocking も発生します．

記事を書くためにいろいろ調べていたら，MPTCP 用にソケット API 
を拡張するといった論文[^enhanced-mptcp] を見つけました．この論文では subflow を扱える API を設計，実装しています．

MPTCP のパススケジューリングのアルゴリズムに関する研究はよく目にします．

### Multipath QUIC (MPQUIC)
MPQUIC は，2018年5月現在 IETF で標準化が進められている[^ietf-mpquic]比較的新しいプロトコルです．
2017年には，ネットワーク系の一流国際会議である ACM CoNEXT 2017 で MPQUIC 
に関する論文[^mpquic] が発表されました．ラストオーサーは先程紹介した MPTCP の拡張 
API に関する論文[^enhanced-mptcp] の方と同じですね．

#### QUIC (Quick UDP Internet Connections)
注目度の高さから，大変勉強になる記事が多いです，ありがとうございます．
以下の記事を参考に，かい摘んで説明します．

- [GoogleのQUICプロトコル：TCPからUDPへWebを移行する | POSTD](https://postd.cc/googles-quic-protocol-moving-web-tcp-udp/)

QUIC は元々 Google 社が考案したプロトコルで，HTTP のメッセージを高速に安全に転送することを目的としています．
2014 年以降，Chrome で試験的に使用されています．また，IETF でも標準化が進められており，
前者を gQUIC 後者を iQUIC と呼ぶこともあります．お互い要素を取り込み合ったりしているのですが，
細かい部分では仕様が異なるようです．
Google Chrome からこの URL（ [chrome://net-internals](chrome://net-internals)）を叩くと QUIC や HTTP/2 のセッション情報がモニタリングできます．

私はあまり標準化を追っていないので標準化に関する詳しい話はできません．
標準化に関しては，yuki さんの記事が大変参考になりました，ありがとうございます．

- [QUICの現状確認をしたい（2018/1）](https://qiita.com/flano_yuki/items/251a350b4f8a31de47f5)
- [QUICの現状確認をしたい 2018//2 (MTU, Migration, Packet Number Encryptionなど) - ASnoKaze blog](https://asnokaze.hatenablog.com/entry/2018/02/06/004539)

以下に QUIC の主な特徴を示します．

- **UDP の上位で動作**：通信路確立時間の短縮
- **TCP のような輻輳制御アルゴリズムを提供**：公平性の確保
- **セッション管理**：L3 ハンドオーバ時の遅延削減
- **ペイロードだけではなく制御情報もほとんど暗号化可能**：情報保護
- **前方誤り訂正（Forward Error Correction; FEC）付与**：再送抑制
- **ユーザ空間実装**：開発，デプロイの高速化

このように非常に高機能で，かえって層が複雑化しないのかが気になります．
論文に関しては，2017年，ネットワーク系のトップカンファレンスである ACM 
SIGCOMM で発表された論文[^sigcomm-quic] がおそらく初めてだと思われます． 

この論文，著者の順番がアルファベット順になっていることに何か意味を感じざるを得ない...

下記ツイートのように，QUIC のトラフィックシェアも上がってきているようで本当にすごいです．
こんな複雑なプロトコルが平気でインターネットスケールで動作していることに驚愕です．
{{< tweet 996378414754947073 >}}
{{< tweet 996384026540752896 >}}

<mark>ところでみなさん，DCCP って覚えていますか？</mark>

##### QUIC Architecture ？
私の研究分野がネットワークアーキテクチャであるので，この視点で見てしまいがちです．
図6 にプロトコルスタックにおける QUIC の位置付けを示します．
{{< figure src="/img/quic-stack.png" title="図6. Protocol Stack (QUIC)" class="center"  >}}
<font size="2"> https://datatracker.ietf.org/meeting/98/materials/slides-98-edu-sessf-quic-tutorial/ より引用</font>

一般に QUIC はトランスポート層プロトコルと言われています．はじめは UDP サブレイヤみたいな位置付けかと
思っていましたが，図6 のように UDP の上位で動作します．そして一部アプリケーション機能を提供します．

ここで私の良くない病気（階層モデル志向）が出ます．

アプリケーション（HTTP？）機能を一部提供するのにトランスポート層プロトコルと言われてしまうと，
（少なくともOSI参照モデルでは）第何層に位置するのか，私はよくわからなくなってしまうのです．
こんなことを気にしてもしょうがないとわかってはいるのですが...

TCP/IP プロトコルスイートの場合も同じくトランスポート層（第3層）に位置付けられそうですが，
やはりアプリケーション機能を提供する部分で頭が止まってしまいます．

最近では，（ほんとうに）勝手に QUIC Architecture という，HTTP 
に特化した新しいプロトコルスタックだと自分に言い聞かせています．

UDP が L4 で L7 が QUIC+HTTP という解釈はどうかな？？  
セッション管理機能もあるし L4=UDP，L5=QUIC，L7=HTTP とかどうかな？？？？  
この位置付けをすることの意味ってなんだっけ？？？？？？


病気終わり
- - - 

##### 耐障害性
MPQUIC では

##### 帯域集約
そもそも QUIC は UDP 上で動作するので TCP マルチストリーミングで発生する HoL
Blocking が発生しません．受信したら，それが送信された順番に関係なくすぐ上位層にデータを渡せます．
この点で非常にマルチパス通信との相性が良いです．

## 階層の複雑化と機能重複について
#### 複雑化
L4 以上で実現される代表的なマルチパスについて説明しました．
もちろん SCTP や MPTCP，MPQUIC 以外にも over L4 マルチパスに関する研究はありますが，
L4 で通信路を多重化するモデルは概ね共通していると思います．

とはいえ，正直階層が複雑だと思います．難しいことが実現できるのはすごいのですが，
もっとシンプルなやりかたで同等のことができたらいいのにな，とほそぼそと考えています．
実現したいことが高度になるほど，実装部分でアルゴリズムが難しくなるのは仕方がないかと思います．
ただ，階層構造を採る以上は，より複雑なことは上位層に任せる（場合によっては層を追加する）とプロトコルとしての汎用性が高まるのではないのかと考えます．
仮に /32 でパケットが運べるなら L3 ルーティングは不要と唱えるひともいますし，それが当時できなかったからこそ L3 
にシフトしたとも考えられます．

#### 機能重複
階層構造の性質上，下位層で高度なことが実現されるとそれよりも上位の層は意識すること無く高機能性を利用できます．
例えば，L3 で ID/Locator Split という機能が働けば，L4 以上は IP アドレスを意識しないで通信路を確立できます．
この場合，端末の IP アドレスが変化しても L4 通信路は切断されること無く通信を継続できます．では 
L4 以上でマルチパスを実現すればよいのか？という話になります．

紹介はしませんでしたが，L4 以上に限らず L2，L3 にもマルチパスは存在します．
例えば，L2 には Link Aggregation Control Protocol（LACP）が，L3 には Equal Cost Multi Path（ECMP）が存在します．

各階層でそれぞれ同等の高機能性が提供されるのは機能重複ではないか？
と考えられますが，各層で意味が異なるので良い意味で冗長だと思います．

[^tcpip]: R. T. Braden, "Requirements for Internet Hosts - Communication Layers," [RFC 1122](https://tools.ietf.org/html/rfc1122), Oct. 1989.
[^1]: こちらの意図を英訳すると Highly Functional Communication Paths になるのですが，直感的には Enhanced Communication Paths がわかりやすいです．
[^sctp]: R. R. Stewart, "Stream Control Transmission Protocol," [RFC 4960](https://tools.ietf.org/html/rfc4960), Sep. 2007.
[^mptcp]: A. Ford, C. Raiciu, M. J. Handley, and O. Bonaventure, "TCP Extensions for Multipath Operation with Multiple Addresses," [RFC 6838](https://www.rfc-editor.org/rfc/rfc6824.txt), Jan. 2013.
[^cmt-sctp]: Prof. P. D. Amer, M. Becke, T. Dreibholz, N. Ekiz, J. Iyengar, P. Natarajan, R. R. Stewart, M. Tüxen, "Load Sharing for the Stream Control Transmission Protocol (SCTP)," [draft-tuexen-tsvwg-sctp-multipath-15.txt](https://www.ietf.org/id/draft-tuexen-tsvwg-sctp-multipath-15.txt), Jan. 2018.
[^conf-mptcp]: src IP ごとにルーティングテーブルを設定したり，I/F を mptcp enabled にしたりする程度です．詳しくは [^mptcp-official]を参照．
[^mptcp-official]: [MultiPath TCP - Linux Kernel implementation](https://multipath-tcp.org/pmwiki.php)
[^mpquic]: Q. D. Coninck and O. Bonaventure, "Multipath QUIC: Design and Evaluation," In Proc. of the 13th International Conference on Emerging Networking EXperiments and Technologies (CoNEXT'17), pp. 160--166, Incheon, Republic of Korea, Dec. 12--15. 2017.
[^ietf-mpquic]: Q. D. Coninck and O. Bonaventure, "Multipath Extension for QUIC," [draft-deconinck-multipath-quic-00](https://tools.ietf.org/html/draft-deconinck-multipath-quic-00), Oct. 2017.
[^sigcomm-quic]: A. Langley, A. Riddoch, A. Wilk, A. Vicente, C. Krasic, D. Zhang, F. Yang, F. Kouranov, I. Swett, J. Iyengar, J. Bailey, J. Dorfman, J. Roskind, J. Kulik, P. Westin, R. Tenneti, R. Shade, R. Hamilton, V. Vasiliev, W. Chang, and Z. Shi, "The QUIC Transport Protocol: Design and Internet-Scale Deployment," In Proc. of the Conference of the ACM Special Interest Group on Data Communication (SIGCOMM '17), pp. 183--196, Los Angeles, CA, USA, Aug. 21--25. 2017.
[^enhanced-mptcp]: 	B. Hesmans and O. Bonaventure, "An Enhanced Socket API for Multipath TCP," In Proc. of the 2016 Applied Networking Research Workshop (ANRR'16), pp. 1--6, Berlin, Germany, Jul. 16. 2016.
