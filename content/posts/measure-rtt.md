+++
title = "Measuring RTT of two places"
date = 2019-04-26T14:57:45+09:00
draft = true
tags = ["network"]
categories = ["覚書"]
+++

## 2点間の RTT を大まかに計測する
大学間の Round Trip Time（RTT）参考値として知りたい場合がある．
しかし，大学内ネットワークは icmp をドロップするようにしている
ことがままあるため，指定した機器との間では RTT を計測できないことがある．

そこで，宛先の AS（e.g., 大学ネットワーク）の出入り口となるルータを
推定し，そこを端点のひとつとして RTT を計測する．

## AS の出入り口を特定する
例えば
大学ネットワークの出入り口（インターネットとの境界にあるルータ）を