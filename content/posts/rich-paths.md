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
（マルチパスだけでも長い...）


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

<font size="2">[インターネット・プロトコル・スイート - Wikipedia](https://ja.wikipedia.org/wiki/インターネット・プロトコル・スイート) より（2018/06/07 時点で確認）</font>

階層化の目的や思想ついてはまた別のエントリで書くとます．
ここではタネンバウム先生の本から階層化の目的について引用させていただきます．

> 時間とともにネットワークは大きくなり，新しい設計が出現して既存のネットワークと接続する必要性が生じる。 我々は，変化を支援するために用いられる重要な構造手法，すなわち問題全体を分割し，実装の詳細を隠すプロトコル階層化（protocol layering）を先に述べた。他にも多くの戦略がある。

<font size="2">[アンドリュー・S・タネンバウム; デイビッド・S・ウエザロール. コンピュータネットワーク 第5版](https://www.amazon.co.jp/コンピュータネットワーク-第5版-アンドリュー・S・タネンバウム-ebook/dp/B076HJDZHQ/ref=sr_1_1?ie=UTF8&qid=1528368745&sr=8-1&keywords=コンピュータネットワーク) (Kindle の位置No.1176-1179). 日経BP社. Kindle 版. </font>


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
この3つを紹介します．

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

WebRTC の DataChannel には ユーザ空間で SCTP (over UDP) が採用されています．
このような記述を見つけましたし，将来的には QUIC に置き換わるのでしょう．

> SCTP is used in WebRTC for the implementation and delivery of the Data Channel. Google is experimenting with the QUIC protocol as a future replacement to SCTP.

<font size="2">[SCTP - WebRTC Glossary](https://webrtcglossary.com/sctp/) より (2018/06/07 時点で確認)</font>

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

スケジューラは，Retransmission TimeOut (RTO) に直面したパスは潜在的にダウンした（potentially failed）
とみなして別のパスを使うようになっているようです．
- [mptcp: sched: Improve active/backup subflow selection](https://github.com/multipath-tcp/mptcp/pull/70)


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
MPQUIC の説明をするために QUIC の説明をします．
注目度の高さからか，大変勉強になる記事が多いです，ありがとうございます．
以下の記事を参考に，かい摘んで説明します．

- [GoogleのQUICプロトコル：TCPからUDPへWebを移行する | POSTD](https://postd.cc/googles-quic-protocol-moving-web-tcp-udp/)

QUIC は元々 Google 社が考案した connection-oriented なプロトコルで，HTTP のメッセージを高速に安全に転送することを目的としています．
2014 年以降，Chrome で試験的に使用されています．また，IETF でも標準化が進められており，
前者を gQUIC 後者を iQUIC と呼ぶこともあります．お互い要素を取り込み合ったりしているのですが，
細かい部分では仕様が異なるようです．
Google Chrome からこの URL（ [chrome://net-internals](chrome://net-internals)）を叩くと QUIC や HTTP/2 のセッション情報がモニタリングできます．

私はあまり標準化を追っていないので標準化に関する詳しい話はできません．
標準化に関しては，以下の記事が大変参考になりました，ありがとうございます．

- [QUICの現状確認をしたい（2018/1）](https://qiita.com/flano_yuki/items/251a350b4f8a31de47f5)
- [QUICの現状確認をしたい 2018 /2 (MTU, Migration, Packet Number Encryptionなど) - ASnoKaze blog](https://asnokaze.hatenablog.com/entry/2018/02/06/004539)

以下に QUIC の主な特徴を示します．

- **UDP の上位で動作**：通信路確立時間の短縮
- **TCP のような輻輳制御アルゴリズムを提供**：公平性の確保
- **セッション管理**：L3 ハンドオーバ時の遅延削減
- **ペイロードだけではなく制御情報もほとんど暗号化可能**：情報保護
- **前方誤り訂正（Forward Error Correction; FEC）付与**：再送抑制
- **ユーザ空間実装**：開発，デプロイの高速化

このように非常に高機能です．
論文に関しては，2017年，ネットワーク系のトップカンファレンスである ACM 
SIGCOMM で発表された論文[^sigcomm-quic] がおそらく初めてだと思われます． 

この論文，著者の順番がアルファベット順になっていることに何か意図があるんでしょうかね．

論文中ではインターネットトラフィックの 7% が QUIC によるものと推定する[^sigcomm-quic]とあります．
下記ツイートのように，QUIC のトラフィックシェアも上がってきているようで本当にすごいです．
こんな複雑なプロトコルがインターネット規模でスケールして動作していることに驚愕です．
{{< tweet 996378414754947073 >}}
{{< tweet 996384026540752896 >}}

<mark>ところでみなさん，DCCP って覚えていますか？</mark>

##### QUIC の位置付け
私の研究分野がネットワークアーキテクチャであるので，この視点で見てしまいがちです．
図6 にプロトコルスタックにおける QUIC の位置付けを示します．
{{< figure src="/img/quic-stack.png" title="図6. Protocol Stack (QUIC)" class="center"  >}}
<font size="2"> https://datatracker.ietf.org/meeting/98/materials/slides-98-edu-sessf-quic-tutorial/ より引用</font>

一般に QUIC はトランスポート層プロトコルと言われています．はじめは UDP sublayer みたいな位置付けかと
思っていましたが，図6 のように UDP の上位で動作します．そして一部アプリケーション機能を提供します[^sigcomm-quic]．

ここで私の良くない病気が出ます．

アプリケーション（HTTP？）機能を一部提供するのにトランスポート層プロトコルと言われてしまうと，
例えば OSI 参照モデルでは第何層に位置するのか，私はよくわからなくなってしまうのです．
こんなことを気にしてなんになるんだ...？（でも気にする）
厳密に第何層かなんて実際に開発したり使用したりする上ではあまり気にする必要ないんですけどね．

従来のトランスポート層プロトコルが担ってきたサービスをアプリケーションに提供しているという
意味ではトランスポート層プロトコルだと思いますが，一部アプリケーション機能も持っているため
L7 とも捉えることができると思います．個人的には L4 の機能を持った L7 がしっくり来ます．

これに関して，MPQUIC の論文[^mpquic] には以下のように書かれています．

> QUIC is a recent proposal initiated by Google and embraced by many others that <mark>collapses the functions of the classical HTTP/2, TLS and TCP protocols into a single application layer protocol</mark> that runs over UDP.

<font size="2">参考文献[^mpquic] より抜粋</font>

QUIC の論文[^sigcomm-quic] では "Transport Layer Protocol" と言われています（どこに位置づくの）．
あと，しばしば SPDY(L7) とか HTTP/2(L7) 
とかと同列で書かれたりしているのを見るとやっぱり L7 では？という気分になります．
元論文が "Transport Layer Protocol" だと言う以上，これからは HTTP/2 on QUIC とか，SPDY on TCP とか，
HTTP/2 on TCP とかいう言い方をしたほうがいいのだろうか（この書き方が厳密に正しいのかすら不明だが)．

ところで L4=UDP で L7=QUIC+HTTP という解釈は？  
セッション管理もあるし L4=UDP，L5=QUIC，L7=HTTP とか？？  
いやいや他にｍ

(病気終わり)

- - - 
#### QUIC のデータ転送
MPQUIC の説明に必要な QUIC の仕様を述べます．
QUIC パケットは，暗号化されていないヘッダ，暗号化された残りのヘッダ，及び Frame で構成されます．
暗号化されないヘッダ部分は以下の3つ．

| フィールド | 説明 |
|---|---|
|Type|Initial, Rety, Handshake, 0-RTT Protected 等の識別|
|Connection ID (CID)|コネクションの識別子|
|Packet Number (PN)|TCP でいうところのシーケンス番号|

主な Frame の種類として

| Frame 名 | 説明 |
|---|---|
|STREAM| CID 内に stream を作成，ストリームのデータの運搬|
|ACK| 送信側にどのパケットが届いたのか通知|
|CONNECTION\_CLOSE| コネクションを終了することを相手に通知|
|RST\_STREAM| stream を突然終了|

などがあります．
アプリケーションペイロードは STREAM Frame の StreamData というフィールドに格納されます．
輻輳検知などに用いる Round Trip Time 推定（RTT estimation）は，ACK Frame の ACK Delay 
フィールドを使用します．

論文を読みつつ，TCP と比較したときに最も興味深いのは，

- <mark>常に PN が増加し続ける</mark>

ことでした．これにより再送の曖昧さ（ambiguity of multiple retransmission）
を回避できます．

簡単に説明します．

1. 受信側が受信状況からあるパケットがロスをしたと判断 (単に到着が遅れているだけかもしれない)
2. 受信側が再送要求
3. 受信側が当該データを受信

3 の時点で，TCP はそのデータが「単に遅れて到着した」パケットなのか
「再送要求して送られた」パケットなのか判断ができません．シーケンス番号が共通なので．
一方 QUIC では PN が増加し続けるので，時系列が容易に把握できます．TCP
よりも正確な RTT 推定が可能です．

RTT 推定は Bufferbloat 問題の解決に役立ちます．Bufferbloat 問題は，実際には輻輳が発生していないのに
両エンドの輻輳制御アルゴリズムの性能を落とします．単純な仕組みですが，PN 
が単調増加することのメリットは大きいように思えます．論文にも MPTCP 
と比較してロスリカバリに長けていると書かれています[^mpquic]．

また，シングルパスの TCP，QUIC では輻輳制御に CUBIC を用いています．
論文中では，MPTCP，MPQUIC には OLIA[^olia] というスキームを用いています．CUBIC 
は，マルチパスプロコトルの下では unfair な挙動をすることが知られています[^unfair-cubic]．

---
#### MPQUIC における Path
MPQUIC ではひとつのコネクション ID
の中に Path という概念を追加してマルチパスを実現します．暗号化されないヘッダ部分に
Path ID を含めることで Path を識別します．Path は IP アドレスにひも付きます．
IPv4, IPv6 の dual-stack ホストの場合，それぞれで Path を持ちえます．
STREAM Frame がどの Path（経路）を通っても関係なく受信側で 
stream のデータ（アプリケーションペイロード）が復元されます．

Path は Path Manager という部分で操作されます．
Path Manager は Path の作成と削除を担います．QUIC のハンドシェイク終了後，
両ホストでまずひとつの Path を開きます．あとは必要に応じて片方のホストが Path
を開きます．Path は UDP 上で実現されるため，Path を activate 
するにはパケットをひとつ流すだけで済みます[^mpquic]．一方で MPTCP 
では Path は TCP で実現されるため，Path ごとに 3-way handshake が必要になります．


MPQUIC では以下の新たな Frame を用いて Path を操作します．

| Frame 名 | 説明 |
|---|---|
|ADD\_ADDRESS Frame| ホストのすべてのアドレスを交換|
|PATH Frame| ホスト global な視点で Active な Path の性能を確認|

ADD\_ADDRESS Frame は SCTP のアドレスリスト交換や MPTCP の ADD\_ADDR 
シンボルに相当します．これにより各ホストで Path のエンドポイントを共有できます．
MPTCP の ADD\_ADDR と異なり，Frame 
は暗号化されるためセキュリティミドルボックスの影響を受けません．

PATH Frame はホスト global な視点で Path の性能の統計情報を確認できます．

##### 耐障害性
先述のように MPQUIC では PATH Frame を用いて Active な Path の性能を確認できます．
これは RTT 推定や slow な Path，急激に性能が劣化した Path 
の検知に使用できます．これにより，複数インターフェイス（I/Fs) が存在するホスト
（dual-homing host） 上で SCTP ライクな failover を実現します．


##### 帯域集約
先述のように，QUIC は UDP 上で動作するので TCP マルチストリーミングで発生する HoL
Blocking が発生しません．受信したら，stream 内で順番が揃っていれば上位層にパケットを渡せます．
この点で非常にマルチパス通信との相性が良いです．
ロスが発生した場合，MPTCP はミドルボックス対策で各パスに順番に再送しなくてはならないですが，MPQUIC
では Frame を同一 Path に送る必要がない点も異なります．

個人的に MPQUICは SCTP と MPTCP にインスパイヤされていると思います．
SCTP よりも現代的なアプリケーションの要求に応える仕組みになっており，MPTCP
が下位層から TCP にみえるように MPQUIC は UDP に見えるのでセキュリティミドルボックスの突破に貢献します．
MPQUIC の論文[^mpquic] はショートペーパーなので，より詳細なものがフルペーパーで読めるのを楽しみにしています．


### おわりに
長くなりすぎました．

ですがひととおり読んでいただくと，マルチパス通信のモデルが理解できるのではないかと思います．
やっぱりマルチパスは D-plane が難しいですね．


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
[^olia]: R. Khalili, N. Gast, M. Popovic, U. Upadhyay, and J. L. Boudec, "MPTCP is not pareto-optimal: performance issues and a possible solution." In Proc. of the 8th International Conference on Emerging Networking EXperiments and Technologies (CoNEXT '12), pp. 1--12, Nice, France, Dec. 10--13, 2012.
[^unfair-cubic]: D. Wischik, C. Raiciu, A. Greenhalgh, and M. Handley. 2011. "Design, Implementation and Evaluation of Congestion Control for Multipath TCP," In NSDI '11.
