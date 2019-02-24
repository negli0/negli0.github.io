+++
title = "Power Balance in the WEB"
date = 2019-02-24T18:21:44+09:00
draft = false
tags = ["web", "network", "https"]
categories = ["雑記"]
+++

### はじめに
ご存知の通り現在の WEB および WEB 周辺技術には HTTPS が多く使用されます．
某国のでっかいファイヤーウォールの件や
SNI を利用した某国のトラフィック遮断の件を見たり，
WEB の動向をのんびり追ったりしていて思うことがちらほらあるわけです．
私の研究は WEB とはあまり関係ないのですが，実用を考えたときの
インターネット上のデファクトスタンダードが WEB に存在すると
感じます．WEB の世界の「あたりまえ」を意識することは大切だと思いながら，
最近感じたことを書きます．

### 現在の WEB になるまでの大まかな流れ
私の話で登場するのは，ざっくりと

- **サービス事業者**
    - コンテンツ事業者とも言えます
    - 大きいところだと Google, Akamai, Facebook など
	    - いわゆるハイパージャイアント
- **ネットワーク**
    - モバイルキャリア
	- ブロードバンドネットワーク


の2者です．今回はこの2者に焦点を当てます．

まずは あきみちさん
（[Twitter: @geekpage](https://twitter.com/geekpage)）の 2009 年（！）
のこの記事を読んでください．

- [インターネットの形を変えて行くGoogle,Facebook,Akamai...: Geekなページ](http://www.geekpage.jp/blog/?id=2009/10/20/1)

ここから私の言いたいことを要約すると，
**ハイパージャイアント（コンテンツ事業者）の台頭で ISP の収入が減ったという変化**
です．特に注目したいのは記事内の次の文です．

> コンテンツを管理するだけではなく、ネットワークも自律的に自分で運用管理することによって、巨大な存在がより巨大になって行く様子を今まさに皆が目にしている感じなのかも知れません。 

1. 商用 ISP のおかげでインターネットが普及
2. インターネットが普及したおかげでコンテンツ事業者が力をつけ始める
3. 力をつけたコンテンツ事業者が独自ネットワークを持つ
4. コンテンツ事業者と peer を張ったほうがネットワークオペレーションのコストが下がる
5. トラフィックを集めやすくなったコンテンツ事業者がさらに力をつける

実際はもっと複雑なんでしょうが，概ねこんな感じのことが起こっているという
ことでしょうか．例えば，一昔前は ISP からメールアドレスを貰っていたと
今は WEB ベースのメール（Gmailなど）が猛威を奮っているわけです．

### 次の方向性
先の記事が書かれたのが 2009 年で私がこのエントリを書いているのが 
2019 年です．研究の傍ら観測可能な範囲で，ここ数年でどのような変化が
起こっているかと言うと，

**コンテンツ事業者側**

- サービス利用者のために超低遅延を目指す
    - CDN 増強，QUIC の導入
	- クラウドのリージョン増加
- 情報セキュリティの観点から TLS ベースの通信を強く推奨
    - HTTP → HTTPS → HTTP over QUIC (HTTP/3) の流れ
	- Everything over HTTPS（さらにそういう流れが進む？）
	    - メール，動画配信などは既に over HTTPS

**ネットワーク側**

- SDN/NFV や Service Chaining など，汎用マシン/汎用技術に基づく新しいトラフィックの操作に注力
	- OpenFlow[^openflow] → P4[^p4] (Data Plane Programmability)
    - IPv6 Segment Routing (SRv6)[^srv6]
	- Service Function Chaining (SFC)[^sfc]
- オペレーションの自動化
	- 運用コスト削減

こんな感じです．他にも進んでいく方向性が現れている部分があると思います．
私はコンテンツ事業者でもネットワークオペレータでも無いので，
実際と違うことがあれば（あるとおもう...）ごめんなさい．
ここからだけではネットワークオペレータ側がどうやって先の変化の
盛り返しを図っているのか私にはちょっとわかりません．
例えば，今後の ISP 業務はどう変わっていくのでしょうか．

### パワーバランス
ここが本題になります．先の通り，ネットワークオペレータ側は運用コストや
設備コストを削減するために**「低コストで賢いトラフィック操作」**
を頑張っている印象です．
一方でコンテンツ事業者（エンド）側は**「低遅延や高セキュリティ」**
を頑張っている
印象です．話題の QUIC（Quick UDP Internet Connection）[^quic-ietf] 
というプロトコルですが，
登場した背景および取っている手法は次の2点で概ね同意がいただけるかと
思います．

- **低遅延 → over UDP 化**
    - ネットワークオペレータ側のミドルボックスの影響を回避
    - TCP ベースにおいて課題だった Head of Line Blocking を回避
- **高セキュリティ → TLS 必須**
    - QUIC ヘッダの一部を除くほぼ全ての UDP ペイロードを暗号化

###### 余談ですが，over UDP は要素技術ではなく，あくまでも結果として選ばれた手段 だと思いますが， IP 直上で動作するように設計上プロトコル番号を持ってほしかったというのが私の本音です．そのうえで現在のミドルボックス の都合上，現実的な解として over UDP 採用した，と言える． <s>そうすれば QUIC はトランスポートだって言って全員黙るでしょうに（ぉ）</s>

現在の主な QUIC の用途としては HTTPS の下のトランスポートですが，
IETF QUIC では用途を HTTPS に限っていないようです．
つまりポート番号は 443 以外でも設計上 OK みたいです．

では，ネットワークオペレータが何に基づいてパケット操作をしているかと
いえば， それは 
**5 tuple （src IP, dst IP, src Port, dst Port, Protocol No.）**
でしょう．

| ポート | サービス|
|:-:|:-:|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |

狭義のポート番号はノード内の通信端点の識別子ですが，
ポート80 なら HTTP，443 なら HTTPS 
というふうに， いわゆる Well Known ポートはサービスと対応しており，広義
ではサービスの識別子とも言えます．
ご存知の通り，ブロードバンドもモバイルも，ポート 80，443 のリクエストが
大半を占めています．IIJ から出ている Internet Infrastructure 
Review[^iir40] を読むと面白いと思います．

WEB 上サービスが多様化して全部 HTTPS 上に集約されていくのは，時代の流れも
あって止められないでしょう．が，**周辺技術までも over HTTPS 化するのは
それでいいんか？と思います．** アプリケーションペイロードを暗号化する
のは賛成です．
例えば DNS over HTTPS[^doh] ですが，機密性を考慮するなら over TLS（over QUIC?）
で良いと思うのです．
{{< tweet 1097277192168300544 >}}
私はコンテンツ事業者でもネットワークオペレータでも
ないのですが，
<mark>Twitter で垂れ流したとおり，
私はこの辺のバランスがとても大事だと思っていますが，そう思う人は少ないのでしょうか...？</mark> 

QUIC ではほとんどの制御パケットすら観測不可能である一方，
Spin Bit について議論されるなど，オペレーションとの落とし所を探るような
議論も見えます．UDP ポート番号は見えるし．
ところがなんでもかんでも over HTTPS が進むと 5 tuple 
の意味がなくなるのですが...
ポート番号はもはや機能していないというならば，それは果たして良い
（健全な）状態なの？と思ってしまいます．
<mark>実際に NoC 業務 や SoC 
業務をやっている方はどう思っているのだろう．</mark>


###### またまた余談ですが，そもそも名前解決のトラストモデルが TLS に基づくなら DHCP で降ってくるレゾルバをどうやって信頼するのとなってしまうわけで．とするならブラウザにレゾルバのアドレス（8.8.8.8 とか）を決め打ちするんですかね...


### インターネットの発達と End-to-End 原理の今後
改めて語るべくもないと思いますが，インターネットが現在ほどの規模に
発展できたのは IP というプロトコルがステートレスで複雑ではなかった
（シンプルだった）からというのは大きいポイントだと思われます．
IP で古くから言われている End-to-End 原理に則り，ネットワーク側は
いわゆるダム（dumb）ネットワークであり，機能は通信のエンド（両端）
で実装されるものでした．ところが，身近なところで言えば NAT/NAPT は
この原理を破るものであり，End-to-End 原理はもはや機能しているのか
もわかりません．学術な立場でさえも，元々の意味での End-to-End 原理
に重きを置くものかどうか，私にはよくわかりません．
もちろんネットワーク中立性という観点からは，通信は End-to−End 原理が
基本で，その中ではネットワーク側は dumb で解釈されるべきだと思います．

つまるところ，私は End-to-End 原理の解釈を今の時代に合うように
もう一度考え直したいのです．

個人的な意見を重ねますが，インターネットの発展の基本である
シンプルさは大事なのだけれど，シンプルさをなるべく維持しつつも
ネットワーク側も賢くなって（not-dumb）いくと良いなと思います．
SFC 然り，SRv6 然り，ネットワーク側が賢くなることは，
技術の発展でしばしば見られる「抽象度があがる」ことだと思います．
L2 の上に L3，L4 と抽象度の高い機能が
積み上がったように．そして SFC も SRv6 も，コスト削減に効くと思いますが，
新たな価値を創造する方向にも進んだらなお素晴らしいなと思います．
そういうのがネットワーク側の新しい業務（価値）になっていくんじゃない
のかなあ．
そのためにも，というわけではないのですが，End-to-End 原理とは
なんだったのか，改めてちょっと考えたいのです．

ともあれコンテンツ事業者もネットワークも両方ガポガポ儲かってほしい．

#### その他
早いとこ MEC（Mobile Edge Computing[^mec-mobile];
Multi-access Edge Computing[^mec-access]）
やんないと，この分野もクラウド事業者がリージョン増やしたりして
ネットワークエッジに計算機いっぱい置いちゃいそう（もう置いてる？）．
あとそろそろ IoT もトップダウンで技術レベルに落として議論したいですよね．
雲とセンサとスマホがでかい矢印でつながったポンチ絵描いてるだけだと
なにも進まないし．インターネット ≠ WEB でしょうし，WEB 以外の
インターネットの使い方を探したいです
（探している人おったら友達になりたい）．
新しいアーキテクチャといえば NDN（Named Data Networking）[^ndn]が
パッと浮かぶんですが，今どういう状況なんでしょうかね？
商用稼働までに時間がかかると思いますが，CDN 業務奪っちゃわないかと
思うんですが（そうではない？）．

### さいごに
WEB だけでも社会インフラになってきているわけで，それらを堅牢化していく
のはそういうものかなと思います（しかたないね）．とはいえネットワーク側と
エンド側がバランスよく，より賢く（高機能に）なっていく世界ががとても
すばらしいなとも思っています．

[^quic-ietf]: [IETF Datatracker | QUIC (quic)](https://datatracker.ietf.org/wg/quic/documents/)
[^doh]: [IETF Datatracker | RFC 8484 - DNS Queries over HTTPS (DoH)](https://datatracker.ietf.org/doc/rfc8484/)
[^openflow]: Nick McKeown, Tom Anderson, Hari Balakrishnan, Guru Parulkar, Larry Peterson, Jennifer Rexford, Scott Shenker, and Jonathan Turner. 2008. OpenFlow: enabling innovation in campus networks. SIGCOMM Comput. Commun. Rev. 38, 2 (March 2008), 69-74. 
[^p4]: Pat Bosshart, Dan Daly, Glen Gibb, Martin Izzard, Nick McKeown, Jennifer Rexford, Cole Schlesinger, Dan Talayco, Amin Vahdat, George Varghese, and David Walker. 2014. P4: programming protocol-independent packet processors. SIGCOMM Comput. Commun. Rev. 44, 3 (July 2014), 87-95. 
[^srv6]: [IETF Datatracker | draft-filsfils-spring-srv6-network-programming-07 - SRv6 Network Programming](https://datatracker.ietf.org/doc/draft-filsfils-spring-srv6-network-programming/)
[^sfc]: [IETF Datatracker | Service Function Chaining (sfc)](https://datatracker.ietf.org/wg/sfc/about/)
[^mec-mobile]: Yun Chao Hu, Milan Patel, Dario Sabella, Nurit Sprecher, and Valerie Young. Mobile Edge Computing A key technology towards 5G. ETSI White Paper 11, September 2015. Also available at [here](http://www.etsi.org/images/files/ETSIWhitePapers/etsi_wp11_mec_a_key_technology_towards_5g.pdf)
[^mec-access]: Alex Reznik, Rohit Arora, Mark Cannon, Luca Cominardi, Walter Featherstone, Rui Frazao, Fabio Giust, Sami Kekki, Alice Li, Dario Sabella, Charles Turyagyenda, and Zhou Zheng. Develop- ing Software for Multi-Access Edge Computing. ETSI White Paper 20, September 2017. Also available at [here](https://www.etsi.org/images/files/ETSIWhitePapers/etsi_wp20_MEC_SoftwareDevelopment_FINAL.pdf)
[^ndn]: [Named Data Networking (NDN) - A Future Internet Architecture](https://named-data.net)
[^iir40]: [Internet Infrastructure Review Vol.40 - 定期観測レポート](https://www.iij.ad.jp/dev/report/iir/pdf/iir_vol40_report.pdf)
