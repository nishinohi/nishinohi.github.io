+++
title = "コンパクトなGatewayを作る"
tags = ['matageek', 'hw']
date = 2021-01-25

# For description meta tag
description = "Recipe of the legendary Krabby Patty."

# Comment next line and the default banner wil be used.
banner = 'https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/0b931b39-7010-718a-dd5f-1441b42e4556.png'

+++

# 何を作ったのか
---
以前に作成した3G通信モジュールを参考に、マイコンとLTEモジュールを組合わせたコンパクトなGatewayを作成しました。

# OSH
---

## 参考にする製品

まずはOSHから回路図をもらってきます。今回は`SORACOM`認定デバイスでもあり`Seeed`から提供されている[Wio LTEの回路図](https://wiki.seeedstudio.com/Wio_LTE_Cat.1/#resource)を参考にします。Wio LTEで実装されているLTE用の`EC21`の仕様書の参考回路図などから自分で回路を組んでもいいのですが、最終的にPCBAをSeeedに発注する場合Seeedが提供している部品リストを利用したほうが安価で納品も速いです。なお、後述しますが最終的に今回は技適対応済みの`EC25-J`を実装することになりました。

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/be013c2a-44ea-5685-80b6-ce512f362d3e.png)

## 回路図を見る

前回と同様に通信以外の不要な回路を削除します。`Wio LTE`には`STM32F4`というマイコンが搭載されていますが、今回はWiFi機能もつけたかったためマイコン部分を[ESP32](https://www.espressif.com/en/products/socs/esp32)に置き換えます。また、Wio LTEは3層実装でしたが部品点数を減らせば2層に収まりそうだったので2層で実装にします。

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/f465b3cb-c4f8-cee5-9b40-156178dd0e87.png)

# 改修作業
---
## 改修内容

以下が今回の改修を行った回路図と配線図です。通信機能のみが必要なので上図の回路図よりかなり部品点数が減りました。マイコンもESP32に交換済みです。WiFi用のアンテナが干渉されないように基盤の外にでるように配置しましたが、電波関係の知識は全く無いのでどのような配置がアンテナにとってよい配置なのかはわかっていません。

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/6c3f39db-0c74-3454-c8de-9f69e457b944.png)

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/39b15ca3-394e-8e31-a0e0-4e27ce5c8f31.png)

## 完成品

とりあえず届いた完成品を見てみます。いかにも無理やりつけた配線が飛び出していますが、これは私がLTEモジュールの起動用のPINを少しいじる必要があったので付け足しました。

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/bb3ded8d-9348-0f1b-ef78-6bdc3db6c787.png)

## トラブル

どうもSeeedが提供していたSIMカードホルダーのEagle用のライブラリに不備があったようでそのまま配線するとSIMを正しく読み取れませんでした。なので一度SIMカードホルダーを半田を溶かして無理やり引き剥がし配線をやり直しました。見た目が残念なことになりましたがこれで正しく動作したので一安心です。ちなみにSeeed側からは謝罪を兼ねてSeeed内で使用できるクーポン券が送られてきました。

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/0b931b39-7010-718a-dd5f-1441b42e4556.png)

# EC25-Jは特別な入手経路
---

実は今回LTE用のモジュールとして実装している`EC25-J`ですが、本来技適マークが印字されているものは**個人では注文できません**。Seeed主催の勉強会で登壇した際に知り合ったSeeedのエンジニア兼マーケティング担当の方に相談したところ、`EC25-J`なら実装できるように手配すると言われ今回このモジュールが完成しました。本当にありがとうございました、この場を借りてお礼申し上げます。