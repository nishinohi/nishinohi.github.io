+++
title = "MATAGEEK"
tags = ['matageek']
date = 2021-06-25

# For description meta tag
description = "Recipe of the legendary Krabby Patty."

# Comment next line and the default banner wil be used.
banner = 'img/matageek.png'

+++

![](img/matageek.png)

# MATAGEEKって何？
---
一言でいうとBLEメッシュネットワークを利用した狩猟用わなの検知及び通知システムです。

# わな猟って大変ですよね
---
狩猟といえば猟銃のイメージがありますが、わな猟も重要な方法の一つです。しかし仕掛けたわなは原則毎日見回りを実施する必要があります。[^1]（非狩猟鳥獣がかからないように、手負いの動物をつくらないように等）また獲物がかかった場合もできる限り早急に対処できた方が理想的です。
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/96263081-cac6-4885-2b5d-d6f0faa54969.png)
ということでイメージとしては以下のようなものが出来上がれば色々便利そうです。
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/8d2f2fb1-1e60-9222-44d6-5aff25b38ba4.png)
わなが作動したときに通知を送信するようなものであれば[SORACOM LTE-M Button](https://soracom.jp/store/5208/)などの市販のIoTモジュールで割と事足ります。ですがわな猟は捕獲率を上げるために動物の通りそうな箇所にわなを複数個仕掛けるのが一般的で、それぞれのわなにIoTモジュールを配置するのは費用的に個人で可能なレベルとは少し言いにくいです。（ピンきりですがLTE回線等が使用できるモジュールは安くても1万円はするので）

# メッシュネットワーク
---
上記の問題を解決するために、今回はBluetoothを用いたメッシュネットワークを利用します。市販品でも親機（LTE回線が使用できるもの）と子機（親機と短距離通信ができるもの）でネットワークを作るIoTモジュールもありますが、狩猟用に利用できそうなものではスター型のネットワークのものしか調べた限りなさそうでした。（あったら教えてほしいです）
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/2f2d1010-2abe-52d4-e1ff-b20e35fe1b86.png)
ここでBluetoothに詳しい人であれば「あ〜`Bluetooth5.0`の`Bluetooth Mesh`ね」と思われたでしょう。しかしBluetoothにもう少し詳しい人であれば表題に違和感を覚えたはずです。`Bluetooth 5.0`で追加された`Bluetooth Mesh`は`Bluetooth Low Energy（BLE）`に基づいてはいますが**低消費電力ソリューションではありません**。
簡単に説明すると`Bluetooth Mesh`は接続を維持するために仕様上無線をオフにすることができません。そしてBluetooth機器における電力消費の主たるものは無線です。一部のノード（メッシュ内の一つのBluetoothデバイスのこと）を低消費電力モード（Low Powerモード）で稼働させることは仕様的にも可能ですが、Low Powerモードと通信するノードはLow Powerモードにすることができません。つまり、**全てのノードを低消費電力で稼働させることは仕様上不可能**です。
となると表題の低消費電力狩猟用モジュールとは一体どういうことなのとなります。実は今回のメッシュネットワークは**Bluetooth 4のBLE機能のみ**でメッシュネットワークを形成することで低消費電力を実現しています。

# FruityMesh
---
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/19df096e-8859-64b8-defa-828207ac3a32.png)
[FruityMesh](https://github.com/mwaylabs/fruitymesh)という`Bluetooth 4`の`BLE`のみでメッシュネットワーク作成しているPJがあります。このPJについて色々説明したいのですがそれだけで記事3つくらいの分量になりそうなので今回は詳細な説明は省きます。このPJでは実際に消費電力を計測したところ電力消費の高いスキャニング（デバイスを探す機能）でも**1mA以下**でした。またメッシュ内のノードがある個数まで達したときスキャニングを停止することが可能であれば、ノードの個数にもよりますがおおよそ**150-250µA**で稼働させることができるようです。今回はこれに色々手を加えつつ、スマホからもノードの情報を取得したり操作できるようにします。偉大なる先人に感謝🙏

# 構成
---

## 狩猟用わなモジュール構成
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/50e962de-9d79-3075-4b01-ccf83bd2ff61.png)

メッシュ内に最低でも一つゲートウェイ機能（LTE回線が利用できる）があるノードを含め、任意の数のノードでメッシュネットワークを形成します。ノードを設置するフローは以下のようになります。

* スマートフォンなどからノードに接続しノードを任意のネットワークに登録（エンロール）する
* メッシュネットワーク内のノードを設置モードにしてわなを仕掛けた場所に設置する
* メッシュネットワーク内のノードを探知モードにする

### ノードを任意のネットワークに登録（エンロール）する
ノードは特定のノードとのみメッシュネットワークを作成する必要があります。（そうでないと赤の他人が近くに設置した場合相互に接続してしまいます）そのためノードには以下の２つの情報を付与することで特定のネットワークに所属させることができます。（これを登録（エンロール）と呼んでいます）これは`Bluetooth Mesh`での`Provisioning`に対応していると考えてもらっていいと思います。（参考：[Provisioning a Bluetooth Mesh Network](https://www.bluetooth.com/blog/provisioning-a-bluetooth-mesh-network-part-1/)）
* ネットワークID
* ネットワークキー

ネットワークIDは自身が所属すべきネットワークの識別ID、ネットワークキーは16byteで構成される対象のネットワークに接続するための共通鍵です。これにより例えば自分と他人がネットワークIDを同じ`1`としてエンロールしてもネットワークキーが異なれば接続されません。この機能はFruityMeshに実装されている標準機能で実現できますが、現状の実装だとノードを個別に有線（UART等）でエンロールする必要があるため以下のような機能を追加して処理が一度で済むようにしています。

* ノードは初期状態ではデフォルトのネットワークIDとネットワークキーで起動し、同じく初期状態のノード同士でメッシュネットワークを作成する
* 初期状態のノードにスマートフォンで接続し、ネットワークIDとネットワークキーをメッシュネットワーク内のノードに送信する

ただしこの機能は初期状態のノードは同じく初期状態のノードと無差別にメッシュネットワークを形成するため、登録作業は自身の所持しているノード以外のノードが存在しない環境で実施する必要があります。

### 設置モードと探知モード
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/fffb7bb2-e6a5-edfe-c20d-eb38d22af4a4.png)
モードの切り替え機能は狩猟用モジュールとして追加した機能の一つです。これらのモードの違いは主にBluetoothのスキャニングの頻度であり、つまるところ消費電力の違いです。設置モードはスキャニングの頻度を高め電力を多めに消費するかわりに他のノードが探しやすくなります。逆に探知モードのノードは現在の接続を維持しつつ他のノードを探さなくなるかわりに消費電力を大幅に抑えることが可能です。
猟場にわなを設置する場合は設置モードに設定し、わな設置後期待通りのメッシュネットワークが形成されていれば探知モードに切り替えて長時間稼働させるという使用方法を想定しています。

## インフラ構成
メッシュ内の何れかのノードが作動した場合ユーザーに通知を送信します。図ではブラウザで情報を見れるような記述がありますが今回そこは説明しませんというかまだできてません😂

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/46123292-871e-7002-a38d-8c093b3c0d78.png)

後述するSORACOM BEAMによる暗号化を利用するための証明書や秘密鍵の発行はAWS IoTを利用しています。これでSIM毎に個別の暗号化ができるのでもしこの仕組みを利用したい方がいればそれぞれでセキュリティを担保できます。

# 暗号化
---
簡単では無いですが避けては通れないのが暗号化です。今回は2種類の通信でそれぞれに暗号化が必要です。

* わなの作動情報をAWS IoTへ送信する際の暗号化
* メッシュネットワーク内の通信の暗号化

メッシュネットワーク内の通信はローカルネットワークなので、スニッフィングするにはわなが設置されている場所まで赴く必要があります。ですので正直そこまで気にしなくてもという気もしますがFruityMeshにすでに暗号化の機能が実装されているのでありがたく利用します。

## わなの作動情報の暗号化

主にIoTなどで利用されるMQTTプロトコルでわなの作動状況を送信しますが、そのままでは何も暗号化されていないのでMQTTSを使用したいところです。しかしMQTTSを使用するには一般的なIoTデバイスの乏しいリソース（今回は64Mhz、256KB SRAM程度のものを使用します）では少々辛い部分があります。そこで活躍してくれるのが[SORACOM BEAM](https://soracom.jp/services/beam/)です。SORACOM BEAMはIoTデバイスにかかる暗号化等の高負荷処理や接続先の設定をクラウドにオフロードできるサービスです。
デバイス側は下図のようにキャリアの閉域網からBeamのエンドポイントへ暗号化されていないパケットを送信するだけです。その後Beam側が受信したパケットを指定された暗号化方式で暗号化して指定された送信先へパケットを転送してくれます。
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/db602a56-c17b-2387-4ac7-0f00ae952274.png)

## メッシュネットワーク内での通信の暗号化

`Bluetooth 4`では[ペアリングとボンディング](https://www.bluetooth.com/blog/bluetooth-pairing-part-1-pairing-feature-exchange/#:~:text=Bonding%20is%20the%20exchange%20of,that%20allows%20bonding%20to%20occur.)による暗号化が仕様として策定されていますが、FruityMeshでは独自の暗号化を利用しています。`GAP`（Bluetoothでの通信の基本的な規約のようなもので暗号化もここで規定されています）の**非暗号化通信にアプリケーション層で独自の暗号化**を実施しています。（この仕様は異なるデバイスでの相互運用を目的としているようです）暗号化の方式は簡単に言うと共通鍵暗号方式です。ノード間でコネクションを確立した際、一定のタイムアウト内で乱数を共通鍵で暗号化したパケットを交換することでお互いを認証しています。

# 使用する機材
---

## メッシュネットワーク用デバイス
ここまできてやっと実際どんな機材を使うのかという話です。今回は比較的長い距離（実測で見通し距離30m程）でも通信できる[Nordic nRF52840](https://www.nordicsemi.com/Products/Low-power-short-range-wireless/nRF52840/GetStarted?lang=ja-JP)を使用します。このSoCを使用していて国内で使用できる（技適的な意味で）ものはそんなに多くなく、購入の容易さから大体以下の２つかなと思います。プログラムはJ-LINKで書き込むためSWDのコネクタがついているAdafruitの方が便利かなと思います。SparkFunの方にもついてはいるんですが物理的にちょっと使いにくい作りになっています。

* [SparkFun Pro nRF52840 Mini](https://www.sparkfun.com/products/15025)
* [Adafruit Feather nRF52840 Express](https://www.adafruit.com/product/4062)
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/adbbe057-a68f-213d-6836-94bcd5ff0405.png)

Nordic製品の開発は[SEGGER Embedded Studio](https://www.segger.com/products/development-tools/embedded-studio/)を利用することが多いようですが、 VSCodeでも開発可能で私は後者で開発しています。簡単な開発の手引のような記事を書いたことがあるので一応紹介しておきます。

* [VSCode + ARM GNU ToolsでNordicの開発環境を構築する](https://qiita.com/nishinohi/items/545521df7b06e66149c9)

FruityMeshでも環境の構築方法が紹介されているのでFruityMeshを自身でビルドする際は[ここ](https://www.bluerange.io/docs/fruitymesh/Quick-Start.html)を参照してください。CMakeを利用しているのでNordicの開発環境の構築方法とは少し異なります。

## ゲートウェイ
ゲートウェイに関しては好みのものを使用する感じです。BluetoothデバイスからゲートウェイへUART等でわなの作動状況等を送信し、最終的にクラウドへ送信してもらいます。SORACOM BEAMを使用する場合SORACOM SIMを利用できるものが良いと思います。ちなみにSORACOMでの各種サービスの動作確認済みのデバイスは[ここ](https://soracom.jp/support_partners/certified_device/)で公開されています。しかしながら実際ゲートウェイに求めている機能はキャリア網を使用してパケットを送信するだけなので市販のゲートウェイでは機能過多な感じは否めません。以前私は自分でキャリア網を使用するためだけの小さなモジュールを作ってみたので参考までに紹介しておきます。

* [WiFi・GPS機能付き3Gモジュールを作る](https://qiita.com/nishinohi/items/752f94ec1fe6b11e6e8d)

# メッシュネットワークとスマートフォンの接続
---
FruityMeshは有線（UART等）でターミナルのようにコマンドを受け付ける機能が備わっており、それを用いて各種設定や登録処理が可能ですがどうせならスマートフォンからそのような操作を実行したいところです。
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/2ec10a07-89e3-d2fd-afeb-535368093ad2.png)

FruityMeshにはスマートフォンとの接続を考慮した機能が実装されています。しかしFruityMeshのメッシュネットワークに接続できるスマートフォンアプリ自体は特に無いので自前で開発しました。現状Androidのみ対応で整理しきれていないのとセキュリティキーの保存の方法が完全にアウトなのでリンクは貼りませんが気になる方は連携しているGithubのアカウントで公開はしているのでどうぞ。

アプリの操作画面は以下のようになります。

## ノードのスキャニング
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/c4170459-f24e-a306-a29f-21ae613f65b3.png)
アドバタイズパケット（Bluetoothデバイスを発見してもらうためのパケット）にFruityMeshのService IDが含まれているもののみを表示します。また、[ノードのエンロール](###-ノードを任意のネットワークに登録（エンロール）する)で説明した初期状態のノードはアイコンが灰色で表示されます。ちなみに`MataGeek`というのはこのPJの名称です。（マタギ＋ギーク）

## ノードのエンロール
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/2dcc4a33-11de-a26c-6556-8f6dd305fe6c.png)
現状はACTIVATEのボタンを押すと端末（スマートフォン）に保存済みのネットワークIDとネットワークキーを送信します。エンロールされたノードは設定された内容で再起動します。

## モードの変更
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/34d0af56-e54d-7d00-20d7-c5430520a21a.png)
START DETECTを押すとモードを探知モードに変更できます。逆に探知モードの状態だと設置モードに変更できます。ノードの基本的な情報もここで知ることができます。

* ClusterSize
  * メッシュネットワーク内のノードの数
* Mode
  * 現在のモード（SETUPは設置モードを表します）
* Device Name
  * デバイス名（任意に変更可能）
* Trap State
  * わなの作動状況
* Battery
  * バッテリー残量

# バッテリー問題
---

以上の構成でおそらく目的の機能は達成できたかなと考えています。しかしここまで説明しておいてなんですが私は電気回路等は完全に独学でまともな知識が無いので電源をどうすべきか悩んでいます。というのも以下の内容の正しい答えがわかっていません。

* 不燃の小型で高用量バッテリーってある？
* 低消費電力用の電源回路ってどんなの？

まずバッテリーはできればリチウムイオンバッテリーを使用したいですが、最悪ノードを山中に紛失する可能性がある以上可燃性のものは避けたいです。
自分で試す電気回路の電源はいつも三端子レギュレータを使用していました。これが低消費電力プロダクトに向いていないことは知っているのですが、どのようなものが向いているのか正直よくわかっていません。もしどなたかご存知でしたら教えて下さい。

[^1]: 第Ⅱ章 捕獲に関する基礎知識 - 農林水産省
