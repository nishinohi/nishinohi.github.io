+++
title = "コンパクトな3G通信モジュールを作る"
tags = ['matageek', 'hw']
date = 2018-05-16

# For description meta tag
description = "Recipe of the legendary Krabby Patty."

# Comment next line and the default banner wil be used.
banner = 'https://cdn-learn.adafruit.com/assets/assets/000/027/325/medium800/adafruit_products_fabprint.png'

+++

# 何を作ったのか
---
GPS機能付きの3G通信モジュールです。HWの設計に関する知識を持ち合わせていないので0からHWを設計したわけではなく、OSHで提供されている多機能な3Gモジュールを改修してコンパクトなモジュールを再設計・製造したという話です。

# なぜ作ったのか
---
これを作った当時（2016年くらい）、携帯電波網を利用できる比較的安いArduino向けのモジュールがありませんでした。当時はマイコンによる3G通信で安価なホビー向けの環境は`Raspberry Pi ZERO + 3Gドングル + SORACOM SIM`の選択肢が一般的でした。そんな中「USBドングルでかい、Raspberry Pi ZERO意外とでかい、電源リソース苦しい、Arduino使いたい」という私の我儘を叶えられるモジュールはないかと奔走しましたが、そんな願ったり叶ったりなモノは見つからなかったため自作することにしました。

# OSH（オープンソースハードウェア）
## Adafruitとの出会い
安い3Gモジュールが無いかとネットの海を彷徨っていると、OSH（OpenSourceHardware）で有名なAdafruit Industriesの[Adafruit FONA 3G Cellular](https://learn.adafruit.com/adafruit-fona-3g-cellular-gps-breakout/)（以下Fona）という比較的安価（$79.95）なモジュールに出会いました。Arduinoで利用するためのライブラリも提供されています。いたれりつくせりです。しかし技適という壁が立ちはだかり、購入しても日本では使えません。
![](https://cdn-shop.adafruit.com/640x480/2687-01.jpg)

## モジュール部分を技適対応すればよいのでは？
Fonaには3G通信用のモジュールに`SIM5320`が使用されていますが、特定のアンテナとセットで技適が取得されている`SIM5320J`という日本向けのモジュールがありました。モジュールをこれに置き換えて自分で作成してしまえば日本で使用することが可能です。そしてAdafruitの製品は全てOSH。全ての回路図が[Eagle](https://www.autodesk.co.jp/products/eagle/overview)のデータ形式で公開されています。

## 改修内容
AdafruitのモジュールはSIM5320の様々な機能を利用するために多くの部品が実装されていますが、通信するだけであれば部品点数を減らせそうだったため以下のような回路に修正することにしました。

- 電話機能用のスピーカー回路削除
- イヤホンジャック削除
- バッテリーは電池駆動を想定したのでリチウム電池用の充放電管理回路を削除

# 回路図に立ち向かう

## 自分の電気回路の知識レベル
電子工作の知識は全てインターネットとYouTuberから得ましたがそれでも何とかなったので、以降の話も特に電気回路の知識がなくても理解できると思います。

## OSH
Adafruitに感謝しながら以下のリポジトリをクローンしましょう。ちなみに[fritzing用のライブラリ](https://github.com/adafruit/Fritzing-Library)も公開しているようです（fritzing形式のFONAの回路図はありません）。

- [FONA回路図（Eagle）](https://github.com/adafruit/Adafruit-FONA-SIMCOM-3G-Breakout-PCB)
- [FONAライブラリ](https://github.com/adafruit/Adafruit_FONA)

## 回路図をみる
[Eagle](https://www.autodesk.co.jp/products/eagle/free-download)をインストールしていない場合、まずインストールしましょう。思った以上に複雑でないことが分かると思います。それぞれの機能毎に回路図を分けて記述してあるので読みやすいですね。配線図は一見複雑なように見えますが、実際の配線は回路図からある程度Eagleの機能で自動生成できます。

![](https://cdn-learn.adafruit.com/assets/assets/000/027/324/original/adafruit_products_schem.png)
![](https://cdn-learn.adafruit.com/assets/assets/000/027/325/medium800/adafruit_products_fabprint.png)

## 不要な回路を削る
前編で触れましたが、3G通信以外の機能が不要であれば以下の回路は不要なので今回は削除します。

- 電話機能用のスピーカー
- イヤホンジャック
- バッテリーは電池駆動を想定したのでリチウム電池用の充放電管理回路

また、`FONA`自体は`Arduino`での利用を想定しているため`SIM5320`からの3.3V信号をレベルシフター（回路図の「LEVEL SHIFTING」の箇所の三角形の部品）で5V信号に変換しています。しかし、私は`ESP8266`での利用を想定しており、3.3V信号がそのまま利用できるのでレベルシフターも不要です。よって、回路図の以下のものは不要になります。

- 3.5MM HEADPHONE/MIC
- LEVEL SHIFTING
- LIPO CHARGER
- AUDIO FILTERING

それらを削除した後の回路図、配線図は以下のようになります。グッとシンプルかつ簡単になりましたね。アンテナのコネクタを小さくしたかったので、3G通信アンテナ用のコネクタをSMAからUFLに変更しましたがこれはどっちでもいいです（変換コネクタもあるので）。

![](https://qiita-image-store.s3.amazonaws.com/0/167138/2665f1e4-c606-0bfc-8fd9-628e0ebe85bf.png)
![](https://qiita-image-store.s3.amazonaws.com/0/167138/14906106-a395-36c1-ab82-4246a866a06d.png)

改修後の回路図の詳細は下記のリンクを参照して下さい。

* [Adafruit-FONA-SIMCOM-3G-Breakout-PCB](https://github.com/nishinohi/Adafruit-FONA-SIMCOM-3G-Breakout-PCB)

## 発注する
基盤の設計が完了したら後は発注するだけです。発注先は[Seeed](https://www.seeedstudio.com/)です。[Fusion PCB](https://www.seeedstudio.com/fusion_pcb.html)というサービスは、基盤の製造だけでなく部品の実装までそこそこな価格で対応してもらえます。
今回の基盤は表面実装1台、基盤のみ5枚で`$100`ほどでした。FONAより`$20`程高くなっていますが単品の発注でここまで低価格に抑えられたのはすごいです。

発注までの細かい流れは以下の記事が大変参考になりました。また、Seeedの[twitter](https://twitter.com/SeeedFusion?lang=ja)は日本語対応OKで気さくに質問に答えてもらえます。

* [Fusion PCBAでハードウェアを「コンパイル」する](https://qiita.com/mayfair/items/0206bd437c4302be5500#fnref1)

## 実物を待つ
どうもSIM5320Jの入手性が良くなかったらしく、納品まで1ヶ月ほどかかりました。実物を手にしたときは中々に感動しました。謎の配線が飛び出していますが、設計ミスでレギュレーターの配線がおかしくなっていた箇所を無理やり修理した痕跡です。

![](https://qiita-image-store.s3.amazonaws.com/0/167138/8ee1a381-8110-9020-147b-83811be9ffe6.jpeg)

# 使ってみる

## ライブラリの修正
Adafruitから提供されているライブラリは`SIM5320`の新しいファームウェアに対応しておらずSIM5320のATコマンドに関するドキュメントとにらめっこしながら修正しました。ちなみにAdafruitのHPにライブラリは現在大きい改修を実施していると記載されていますが、9ヶ月程放置されています。。。

## 完成した姿
実際に動かせる状態になったモジュールがこちらになります。
こんな見てくれですが、しっかりSIMを認識して、AWS IoTにMQTTで色々Publishすることができました。

![](https://qiita-image-store.s3.amazonaws.com/0/167138/058dd852-7efb-5e7f-cd9c-ad9973689649.jpeg)

# 次のステップ
今回はあくまでモジュールとして作成しましたが、マイコンを組み込んでシンプルなGatewayとして再設計してみたいと思います。